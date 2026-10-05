# Flux on GKE solutions

## Bootstrap verification

A healthy bootstrap has Ready Git source and Kustomization resources in `flux-system`:

```bash
flux check
flux get sources git -A
flux get kustomizations -A
kubectl get deployments -n flux-system
```

## Workload Identity

The durable mapping contains three parts:

1. `roles/artifactregistry.reader` granted to the Google service account;
2. `roles/iam.workloadIdentityUser` allowing `flux-system/source-controller` to impersonate it;
3. `iam.gke.io/gcp-service-account` on the Kubernetes service account, persisted through the bootstrap Kustomization patch.

After pushing the patch, restart the controller once so new Pods use the mapping:

```bash
flux reconcile kustomization flux-system --with-source
kubectl rollout restart deployment/source-controller -n flux-system
kubectl rollout status deployment/source-controller -n flux-system
kubectl get serviceaccount source-controller -n flux-system \
  -o jsonpath='{.metadata.annotations.iam\.gke\.io/gcp-service-account}{"\n"}'
```

## App Kustomization

`apps/my-first-chart/kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - oci-repository.yaml
  - helm-release.yaml
```

Before commit, replace placeholders safely and inspect the diff:

```bash
sed -i.bak \
  -e "s/GAR_LOCATION/${GAR_LOCATION}/g" \
  -e "s/PROJECT_ID/${PROJECT_ID}/g" \
  -e "s/tag: 0.1.0/tag: ${CHART_VERSION}/" \
  apps/my-first-chart/oci-repository.yaml
rm apps/my-first-chart/oci-repository.yaml.bak
git diff --check
git diff
```

Delete the backup only if you just created it and verified the target path.

## Expected ready chain

```bash
flux reconcile source git flux-system
flux reconcile kustomization apps
flux get sources oci -A
flux get helmreleases -A
kubectl get all -n "$APP_NAMESPACE"
```

If the Git Kustomization is Ready but OCI is not, inspect the OCI condition. `401/403` points to identity/IAM; `not found` points to repository path or tag. If OCI is Ready but HelmRelease is not, move downstream to Helm conditions, history, hooks, and workload events.
