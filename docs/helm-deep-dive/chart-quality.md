# Chart design, quality, and security

## Values contract

Treat values as a public API. Use predictable names, shallow structures, correct types, and defaults that render a valid chart. Add `values.schema.json` to reject invalid input before Kubernetes sees it.

```json
{
  "$schema": "https://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "replicaCount": { "type": "integer", "minimum": 1, "maximum": 10 },
    "image": {
      "type": "object",
      "properties": {
        "repository": { "type": "string", "minLength": 1 },
        "tag": { "type": "string" }
      },
      "required": ["repository"]
    }
  },
  "required": ["replicaCount", "image"]
}
```

## Release lifecycle

Be able to explain:

- install creates revision 1; upgrade creates another revision;
- release state is stored as Kubernetes Secrets by default;
- rollback creates a new revision based on an old one;
- `--wait` waits for selected resources; `--wait-for-jobs` includes Jobs;
- `--atomic` implies waiting and rolls back/uninstalls after failure;
- `--cleanup-on-fail` removes newly created resources after a failed upgrade;
- CRDs in `crds/` are installed specially and are not upgraded or deleted like normal templates.

## Safe chart practices

- Set CPU/memory requests and probes intentionally.
- Provide pod/container security contexts rather than assuming cluster defaults.
- Avoid embedding secrets in values committed to Git; rendered release data can expose them.
- Pin dependency versions and review dependency provenance.
- Never template arbitrary user input through `tpl` without understanding the trust boundary.
- Prefer immutable image digests for strong promotion guarantees.

## Quality pipeline

A defensible chart pipeline layers checks:

```text
schema validation → helm lint → render matrix → Kubernetes schema validation
→ policy checks → ephemeral install → helm test → upgrade/rollback test
```

`helm lint` alone does not prove the manifests are accepted by your target Kubernetes version or that the application becomes healthy.
