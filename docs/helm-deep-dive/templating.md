# Helm templating deep dive

Interviews often move past commands and ask you to reason about rendering behavior.

## Built-in objects and scope

Know the purpose of `.Values`, `.Release`, `.Chart`, `.Capabilities`, `.Files`, `.Template`, and the root object `$`. Inside `with` and `range`, dot (`.`) changes scope; `$` still points to the root.

```gotemplate
{{- $root := . -}}
{{- range $name, $value := .Values.extraLabels }}
{{ $name }}: {{ $value | quote }}
app.kubernetes.io/instance: {{ $root.Release.Name }}
{{- end }}
```

## Control structures and whitespace

- `if`, `with`, and `range` control rendering.
- `{{-` trims whitespace on the left; `-}}` trims on the right.
- `nindent` inserts a newline and indents; `indent` only indents.
- `toYaml | nindent N` is the normal pattern for nested YAML.

A common interview debugging exercise is malformed YAML caused by incorrect indentation or aggressive whitespace trimming.

## Named templates

Define reusable fragments in `_helpers.tpl`, then call them with `include` so the result can enter a pipeline:

```gotemplate
{{- define "my-first-chart.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-first-chart.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}
```

Prefer names prefixed by chart name because template names are global across parent and subcharts.

## Functions worth knowing

- `required` fails rendering when a mandatory value is empty.
- `default` supplies computed fallback values; static defaults belong in `values.yaml`.
- `quote`, `toYaml`, `nindent`, `include`, `dict`, `list`, `kindIs`, and `semverCompare` appear frequently.
- `tpl` evaluates a value as a template. It is powerful but expands the input surface and must not be used casually with untrusted values.
- `lookup` queries a live cluster. This makes rendering cluster-dependent and behaves differently under plain `helm template`; use sparingly.

## Debugging rendering

```bash
helm lint ./my-first-chart
helm template interview ./my-first-chart --debug
helm template interview ./my-first-chart --show-only templates/deployment.yaml
helm install interview ./my-first-chart --dry-run=server --debug
```

`--dry-run=server` is needed for realistic `lookup` behavior. Explain the difference between rendering failure, Kubernetes schema rejection, and runtime workload failure.
