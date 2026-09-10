# Interview scenario manifests

These manifests support the Flux deep-dive labs. Review and replace names, namespaces, source references, and chart versions before committing them to your Flux repository.

- `values-from.yaml` demonstrates ordered ConfigMap values, inline precedence, Helm tests, rollback remediation, and drift detection.
- `kustomization-ordering.yaml` demonstrates infrastructure-before-app readiness ordering.
- `limited-reconciler.yaml` creates namespace-scoped RBAC for the least-privilege failure lab.

They are learning examples, not drop-in production configuration. Render or validate them against the Flux CRDs installed in your selected lab cluster before use.
