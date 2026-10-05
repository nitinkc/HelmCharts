# Flux on GKE hands-on labs

These commands have real side effects: Flux bootstrap writes to GitHub and installs cluster-wide controllers. Verify the cluster context first. Use the [Flux on GKE solution](../solutions/flux-gke.md) only after attempting the tasks.

## Lab 3.1 — Bootstrap Flux with GitHub

Create a GitHub token with the permissions required to administer the chosen repository. Export it without placing it in shell history where possible:

```bash
read -s GITHUB_TOKEN
export GITHUB_TOKEN
printf '\nToken loaded into this shell.\n'
```

Run preflight and bootstrap:

```bash
kubectl config current-context
flux check --pre
flux bootstrap github \
  --token-auth \
  --owner="$GITHUB_USER" \
  --repository="$GITHUB_REPOSITORY" \
  --branch=main \
  --path="$FLUX_PATH" \
  --personal

flux check
flux get all -A
```

Clone the newly bootstrapped repository into a separate working directory and enter it:

```bash
export FLUX_REPO_DIR="${HOME}/${GITHUB_REPOSITORY}"
git clone "https://github.com/${GITHUB_USER}/${GITHUB_REPOSITORY}.git" "$FLUX_REPO_DIR"
cd "$FLUX_REPO_DIR"
```

If the directory already exists, update that verified clone instead of deleting it.

## Lab 3.2 — Grant private Artifact Registry access

Check Workload Identity Federation for GKE:

```bash
gcloud container clusters describe "$CLUSTER_NAME" \
  --location "$CLUSTER_LOCATION" --project "$PROJECT_ID" \
  --format='value(workloadIdentityConfig.workloadPool)'
```

Expected: `${PROJECT_ID}.svc.id.goog`. If empty, enable it deliberately:

```bash
gcloud container clusters update "$CLUSTER_NAME" \
  --location "$CLUSTER_LOCATION" \
  --workload-pool="${PROJECT_ID}.svc.id.goog" \
  --project "$PROJECT_ID"
```

Create a dedicated Google service account if needed and grant read-only artifact access:

```bash
gcloud iam service-accounts describe "$GSA_EMAIL" --project "$PROJECT_ID" >/dev/null 2>&1 || \
gcloud iam service-accounts create "$GCP_SERVICE_ACCOUNT" \
  --display-name="Flux source controller" --project "$PROJECT_ID"

gcloud artifacts repositories add-iam-policy-binding "$GAR_REPOSITORY" \
  --location="$GAR_LOCATION" \
  --member="serviceAccount:${GSA_EMAIL}" \
  --role="roles/artifactregistry.reader" \
  --project="$PROJECT_ID"

gcloud iam service-accounts add-iam-policy-binding "$GSA_EMAIL" \
  --role="roles/iam.workloadIdentityUser" \
  --member="serviceAccount:${PROJECT_ID}.svc.id.goog[flux-system/source-controller]" \
  --project "$PROJECT_ID"
```

Persist this patch in `${FLUX_PATH}/flux-system/kustomization.yaml`:

```yaml
patches:
  - target:
      kind: ServiceAccount
      name: source-controller
    patch: |-
      - op: add
        path: /metadata/annotations/iam.gke.io~1gcp-service-account
        value: flux-source@PROJECT_ID.iam.gserviceaccount.com
```

Replace `PROJECT_ID`, commit, and push. Then verify:

```bash
flux reconcile kustomization flux-system --with-source
kubectl get serviceaccount source-controller -n flux-system -o yaml
kubectl rollout restart deployment/source-controller -n flux-system
kubectl rollout status deployment/source-controller -n flux-system
```

## Lab 3.3 — Declare the OCI source and Helm release

Create `apps/my-first-chart/namespace.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: helm-flux-lab
```

Create `apps/my-first-chart/oci-repository.yaml`:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: OCIRepository
metadata:
  name: my-first-chart
  namespace: helm-flux-lab
spec:
  interval: 5m
  provider: gcp
  url: oci://GAR_LOCATION-docker.pkg.dev/PROJECT_ID/helm-charts/my-first-chart
  ref:
    tag: 0.1.0
```

Create `apps/my-first-chart/helm-release.yaml`:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: web
  namespace: helm-flux-lab
spec:
  interval: 5m
  chartRef:
    kind: OCIRepository
    name: my-first-chart
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
  values:
    replicaCount: 2
    redis:
      enabled: true
      architecture: standalone
      auth:
        enabled: false
```

Create `apps/my-first-chart/kustomization.yaml` listing all three resources. Then create `${FLUX_PATH}/apps.yaml`:

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: apps
  namespace: flux-system
spec:
  interval: 5m
  path: ./apps
  prune: true
  sourceRef:
    kind: GitRepository
    name: flux-system
  wait: true
  timeout: 5m
```

Replace every placeholder with your exported values, commit, and push.

## Lab 3.4 — Observe automatic reconciliation

```bash
flux reconcile source git flux-system
flux get kustomizations -A
flux get sources oci -A
flux get helmreleases -A
kubectl get all -n "$APP_NAMESPACE"
```

Change `replicaCount` in Git from `2` to `3`, commit, and push. Do **not** run Helm. Observe:

```bash
flux events --for HelmRelease/${RELEASE_NAME} -n "$APP_NAMESPACE"
kubectl rollout status deployment -n "$APP_NAMESPACE"
kubectl get deployment -n "$APP_NAMESPACE" \
  -o custom-columns=NAME:.metadata.name,DESIRED:.spec.replicas,READY:.status.readyReplicas
```

Record the Git commit, Flux-reported revision, and resulting replica count.

## Completion checklist

- [ ] Bootstrap manifests exist in GitHub and controllers are Ready.
- [ ] Workload Identity uses a dedicated GSA and no JSON key.
- [ ] Private `OCIRepository` is Ready.
- [ ] `HelmRelease` is Ready and uses `chartRef`.
- [ ] A Git-only values change reaches GKE.
