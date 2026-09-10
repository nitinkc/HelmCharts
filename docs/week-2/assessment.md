# Advanced Helm knowledge check and Gate 2

## Quiz

1. Which wins: `values.yaml`, the first `-f`, the second `-f`, or `--set`?
2. Where must parent-chart values for a dependency normally be nested?
3. What does `Chart.lock` preserve?
4. What is the difference between hook event and hook weight?
5. Why should hook Jobs be idempotent?
6. What does a successful Service HTTP `helm test` prove, and what does it not prove?
7. Why should a changed chart receive a new chart version before an OCI push?
8. Which Artifact Registry repository format stores Helm OCI charts?

**Scoring:** explain at least 6/8 correctly before the gate.

## Retrieval check

Without notes, trace the override chain for `redis.auth.enabled` from dependency defaults through parent defaults, two values files, and `--set`.

## Gate 2

Demonstrate from a clean namespace:

1. `helm dependency list` reports a valid dependency.
2. `helm lint` and `helm template` succeed.
3. The lifecycle hook completes during install or upgrade.
4. `helm test --logs` passes.
5. The chart is listed in private Artifact Registry with a unique version.
6. `helm pull` and an OCI-based install work for that exact version.
7. You can explain every non-default value and identify which layer supplied it.

**Pass:** all checks succeed and no credential file or token is committed.

Proceed to [Flux on GKE](../week-3/concepts.md).
