# Helm interview labs

## Lab H1 — Scope and rendering bug

Add this to a temporary template and predict the failure before rendering:

```gotemplate
{{- with .Values.image }}
release: {{ .Release.Name }}
repository: {{ .repository }}
{{- end }}
```

Fix `.Release.Name` using `$`, render only that template, and explain why dot changed.

## Lab H2 — Generate resources with `range`

Add a `configMaps` list to values and render one ConfigMap per entry. Requirements:

- save root as `$root`;
- use release-aware names;
- serialize arbitrary `data` with `toYaml | nindent`;
- make output stable and valid when the list is empty.

Validate:

```bash
helm template interview ./my-first-chart --debug > /tmp/rendered.yaml
kubectl apply --dry-run=client -f /tmp/rendered.yaml
```

## Lab H3 — Add a values schema

Copy the schema from [chart quality](chart-quality.md), then test boundaries:

```bash
helm lint ./my-first-chart --set replicaCount=0
helm lint ./my-first-chart --set replicaCount=3
helm lint ./my-first-chart --set-string replicaCount=three
```

Explain why schema validation is earlier and cheaper than waiting for Kubernetes.

## Lab H4 — Atomic upgrade

1. Install a known-good release.
2. Introduce an invalid readiness path and increment chart version.
3. Run an upgrade with `--atomic --timeout 2m`.
4. Compare `helm history`, deployed revision, Pods, and events.
5. Repeat without `--atomic` and explain the difference.

## Lab H5 — Configuration checksum

Create a ConfigMap consumed by the Deployment and add a checksum annotation to the Pod template:

```gotemplate
checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

Change only ConfigMap data and prove that a rollout occurs. Explain why Kubernetes otherwise does not restart Pods for mounted or environment configuration changes in every consumption pattern.

## Lab H6 — Render matrix

Build a small shell loop that renders these combinations:

- Redis enabled/disabled;
- autoscaling enabled/disabled;
- ingress enabled/disabled;
- replica counts at schema boundaries.

Fail immediately if any render or lint command fails. This is a realistic chart-maintainer interview exercise.
