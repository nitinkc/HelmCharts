# Advanced Helm and OCI packaging

**Time:** 6–8 hours · **Working chart:** `my-first-chart`

## Values precedence

From lowest to highest precedence:

1. chart `values.yaml`;
2. parent values targeting a dependency;
3. each `-f file.yaml`, from left to right;
4. `--set` and `--set-string` values.

Use `helm get values RELEASE -n NAMESPACE --all` to inspect computed release values. A dependency receives values beneath its dependency name or alias.

## Dependencies

Dependencies are declared in `Chart.yaml` and resolved into `charts/` with:

```bash
helm dependency update ./my-first-chart
helm dependency list ./my-first-chart
```

`Chart.lock` records the resolved set. Commit `Chart.yaml` and `Chart.lock`; decide consistently whether packaged dependency archives belong in source control.

## Hooks

A hook is a Kubernetes resource annotated with an event such as `pre-install`, `post-install`, `pre-upgrade`, or `test`. Hook weights order hooks numerically from lowest to highest. Delete policies prevent completed Jobs from accumulating.

Hooks can block a release and should be small, observable, idempotent, and safe to retry.

## Helm tests

`helm test` runs resources annotated with `helm.sh/hook: test`. A test proves only what it checks. A successful HTTP request to a Service validates discovery, routing, and an application response; it does not prove persistence, availability under load, or external ingress.

## OCI charts

Artifact Registry stores Helm charts in Docker-format repositories as OCI artifacts. The chart name and `version` in `Chart.yaml` become the OCI artifact name and tag.

```text
LOCATION-docker.pkg.dev/PROJECT/REPOSITORY/CHART:VERSION
```

Package versions are immutable learning artifacts: increment `version` before publishing a changed chart instead of overwriting the same tag.
