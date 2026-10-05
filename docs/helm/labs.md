# Advanced Helm and OCI labs

Start in the repository root and verify the [environment](../getting-started/environment.md) and [GKE context](../getting-started/gke-connection.md). Try each task before opening the [Advanced Helm and OCI solution](../solutions/helm-oci.md).

## Lab 2.1 — Render, install, upgrade, and roll back

```bash
helm lint ./my-first-chart
helm template "$RELEASE_NAME" ./my-first-chart \
  --namespace "$APP_NAMESPACE" > /tmp/my-first-chart.yaml
helm upgrade --install "$RELEASE_NAME" ./my-first-chart \
  --namespace "$APP_NAMESPACE" --create-namespace --wait
helm status "$RELEASE_NAME" --namespace "$APP_NAMESPACE"
kubectl get all --namespace "$APP_NAMESPACE"
```

Upgrade to three replicas, inspect history, then roll back:

```bash
helm upgrade "$RELEASE_NAME" ./my-first-chart \
  --namespace "$APP_NAMESPACE" --set replicaCount=3 --wait
helm history "$RELEASE_NAME" --namespace "$APP_NAMESPACE"
helm rollback "$RELEASE_NAME" 1 --namespace "$APP_NAMESPACE" --wait
```

**Validate:** `helm get values`, Deployment replicas, and release history all tell the same story.

## Lab 2.2 — Add a Redis dependency

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm search repo bitnami/redis --versions | head
```

Choose a stable version and add this dependency to `my-first-chart/Chart.yaml`:

```yaml
 dependencies:
   - name: redis
     version: "<VERSION_FROM_SEARCH>"
     repository: https://charts.bitnami.com/bitnami
     condition: redis.enabled
```

Add parent defaults to `values.yaml`:

```yaml
redis:
  enabled: true
  architecture: standalone
  auth:
    enabled: false
```

Then run:

```bash
helm dependency update ./my-first-chart
helm dependency list ./my-first-chart
helm lint ./my-first-chart
helm template "$RELEASE_NAME" ./my-first-chart --set redis.enabled=false | grep -i redis || true
```

Create `/tmp/lab-values.yaml` with a different Redis setting. Compare default, `-f`, and `--set` output. Record the winning value at each layer.

## Lab 2.3 — Add a lifecycle hook

Create `my-first-chart/templates/post-install-check.yaml` containing a lightweight Job with:

```yaml
annotations:
  "helm.sh/hook": post-install,post-upgrade
  "helm.sh/hook-weight": "10"
  "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

Have the Job request the chart's Service. Upgrade the release and observe the Job and Helm status. Deliberately use a bad Service name once, predict the release behavior, inspect it, then fix it.

```bash
helm upgrade --install "$RELEASE_NAME" ./my-first-chart \
  --namespace "$APP_NAMESPACE" --wait --timeout 5m
kubectl get jobs,pods --namespace "$APP_NAMESPACE"
helm status "$RELEASE_NAME" --namespace "$APP_NAMESPACE"
```

## Lab 2.4 — Strengthen `helm test`

The chart already includes `templates/tests/test-connection.yaml`. Make its request fail on HTTP errors and add a finite timeout. Then run:

```bash
helm test "$RELEASE_NAME" --namespace "$APP_NAMESPACE" --logs
kubectl get pods --namespace "$APP_NAMESPACE" \
  -l 'helm.sh/chart' --show-labels
```

Break the test path, capture the failing logs, restore it, and prove the test passes.

## Lab 2.5 — Publish to private Artifact Registry

Create a private Docker-format repository if it does not exist:

```bash
gcloud artifacts repositories describe "$GAR_REPOSITORY" \
  --location "$GAR_LOCATION" --project "$PROJECT_ID" >/dev/null 2>&1 || \
gcloud artifacts repositories create "$GAR_REPOSITORY" \
  --repository-format=docker \
  --location="$GAR_LOCATION" \
  --description="Helm OCI learning charts" \
  --project="$PROJECT_ID"

gcloud auth configure-docker "$GAR_HOST"
```

Increment the chart `version`, update dependencies, package, and push:

```bash
helm dependency update ./my-first-chart
helm lint ./my-first-chart
helm package ./my-first-chart --destination /tmp
helm push "/tmp/${CHART_NAME}-${CHART_VERSION}.tgz" \
  "oci://${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}"

gcloud artifacts docker images list \
  "${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}" \
  --include-tags --project "$PROJECT_ID"
```

Pull and install the exact version:

```bash
helm pull "oci://${OCI_REPOSITORY}" --version "$CHART_VERSION" --destination /tmp
helm upgrade --install "${RELEASE_NAME}-oci" "oci://${OCI_REPOSITORY}" \
  --version "$CHART_VERSION" --namespace "$APP_NAMESPACE" --wait
helm test "${RELEASE_NAME}-oci" --namespace "$APP_NAMESPACE" --logs
```

## Completion checklist

- [ ] Dependency renders and can be disabled.
- [ ] Values precedence is demonstrated, not memorized.
- [ ] Hook success and failure are observable.
- [ ] `helm test` passes after a diagnosed failure.
- [ ] A unique chart version is pullable from private Artifact Registry.
