# Advanced Helm and OCI solutions

## Dependency

Use the version returned by `helm search repo bitnami/redis --versions`:

```yaml
# Add under the existing top-level Chart.yaml fields.
dependencies:
  - name: redis
    version: "<SELECTED_VERSION>"
    repository: https://charts.bitnami.com/bitnami
    condition: redis.enabled
```

```yaml
# Add to values.yaml.
redis:
  enabled: true
  architecture: standalone
  auth:
    enabled: false
```

```bash
helm dependency update ./my-first-chart
helm dependency list ./my-first-chart
helm template "$RELEASE_NAME" ./my-first-chart --set redis.enabled=false
```

The highest-precedence setting wins: `--set` > later `-f` > earlier `-f` > parent defaults > dependency defaults.

## Hook Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: "{{ include "my-first-chart.fullname" . }}-post-check"
  labels:
    {{- include "my-first-chart.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": post-install,post-upgrade
    "helm.sh/hook-weight": "10"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: check
          image: busybox:1.36
          command: ["wget"]
          args: ["--spider", "--timeout=10", "{{ include "my-first-chart.fullname" . }}:{{ .Values.service.port }}"]
```

A nonexistent Service causes the Job and Helm action to fail or time out. Inspect the Job Pod logs and `helm status` before correcting it.

## Stronger test

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-first-chart.fullname" . }}-test-connection"
  labels:
    {{- include "my-first-chart.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
    - name: wget
      image: busybox:1.36
      command: ["wget"]
      args:
        - "--spider"
        - "--timeout=10"
        - "http://{{ include "my-first-chart.fullname" . }}:{{ .Values.service.port }}/"
```

## OCI publication

Set `version` in `Chart.yaml` to a new value and keep the shell variable aligned:

```bash
export CHART_VERSION="0.2.0"
helm dependency update ./my-first-chart
helm lint ./my-first-chart
helm package ./my-first-chart --destination /tmp
helm push "/tmp/${CHART_NAME}-${CHART_VERSION}.tgz" \
  "oci://${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}"
helm pull "oci://${OCI_REPOSITORY}" --version "$CHART_VERSION" --destination /tmp
```

If push authentication fails, rerun `gcloud auth configure-docker "$GAR_HOST"` and confirm the active gcloud identity has Artifact Registry Writer permission.
