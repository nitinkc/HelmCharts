# GitOps fundamentals knowledge check and Gate 1

Take this closed-notes. Give reasoning, not one-word definitions.

## Quiz

1. Why can a pull model reduce the distribution of cluster credentials?
2. What breaks if `source-controller` can reach GitHub but not Artifact Registry?
3. Why is deleting a Pod and watching a ReplicaSet recreate it not, by itself, GitOps?
4. Which controller applies a directory of plain Kubernetes YAML from Git?
5. Which controller turns a `HelmRelease` into a Helm action?
6. Where would you look first if a Git revision is current but a Helm install is failed?
7. What evidence proves reconciliation happened rather than a human running `helm upgrade`?
8. Name two security concerns that pull-based delivery does not solve automatically.

**Scoring:** 1 point each. Repair any topic below 6/8 before attempting the gate.

## Gate 1

Without notes, in five minutes:

1. Draw GitHub, Artifact Registry, the four Flux controllers, and the Kubernetes API.
2. Trace one values change end to end.
3. Explain the difference between pull and push delivery.
4. Explain where credentials live in each model.
5. Assign Git fetch, manifest apply, Helm action, and alert delivery to their owners.

### Pass rubric

- **Pass:** correct direction of control, all controller responsibilities, and a clear distinction between desired/observed state.
- **Retry:** treats Flux as a CI runner, confuses `kustomize-controller` with `helm-controller`, or cannot identify the evidence of reconciliation.

Continue to [Advanced Helm and OCI](../helm/concepts.md) only after passing.
