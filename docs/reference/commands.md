# Command cheat sheet

## Context and Kubernetes

```bash
kubectl config current-context
kubectl get all -n "$APP_NAMESPACE"
kubectl get events -n "$APP_NAMESPACE" --sort-by='.lastTimestamp'
kubectl describe <kind> <name> -n "$APP_NAMESPACE"
kubectl logs -n "$APP_NAMESPACE" <pod-or-job>
```

## Helm

```bash
helm lint ./my-first-chart
helm dependency update ./my-first-chart
helm dependency list ./my-first-chart
helm template "$RELEASE_NAME" ./my-first-chart -n "$APP_NAMESPACE"
helm upgrade --install "$RELEASE_NAME" ./my-first-chart -n "$APP_NAMESPACE" --wait
helm status "$RELEASE_NAME" -n "$APP_NAMESPACE"
helm history "$RELEASE_NAME" -n "$APP_NAMESPACE"
helm get values "$RELEASE_NAME" -n "$APP_NAMESPACE" --all
helm test "$RELEASE_NAME" -n "$APP_NAMESPACE" --logs
helm rollback "$RELEASE_NAME" <REVISION> -n "$APP_NAMESPACE" --wait
```

## OCI and Artifact Registry

```bash
gcloud auth configure-docker "$GAR_HOST"
helm package ./my-first-chart --destination /tmp
helm push "/tmp/${CHART_NAME}-${CHART_VERSION}.tgz" \
  "oci://${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}"
helm pull "oci://${OCI_REPOSITORY}" --version "$CHART_VERSION"
gcloud artifacts docker images list \
  "${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}" --include-tags
```

## Flux

```bash
flux check --pre
flux check
flux get all -A
flux get sources git -A
flux get sources oci -A
flux get kustomizations -A
flux get helmreleases -A
flux reconcile source git flux-system
flux reconcile kustomization apps --with-source
flux reconcile helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE" --with-source
flux events --for HelmRelease/${RELEASE_NAME} -n "$APP_NAMESPACE"
flux logs --kind=HelmRelease --name="$RELEASE_NAME" -n "$APP_NAMESPACE"
flux suspend helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
flux resume helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
```

## Git evidence

```bash
git status --short
git diff --check
git diff
git log -5 --oneline
git rev-parse HEAD
```
