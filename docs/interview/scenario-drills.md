# Scenario-based interview drills

For each scenario, spend two minutes clarifying assumptions, five minutes diagnosing, and two minutes presenting a durable fix.

## Scenario 1 — Green source, red release

`GitRepository`, Flux `Kustomization`, and `OCIRepository` are Ready. `HelmRelease` reports `UpgradeFailed` after a chart update.

Explain what this rules out, which conditions/history/events you inspect, how hooks and tests affect success, and how remediation settings change the outcome.

## Scenario 2 — Drift or another controller?

A Deployment's replicas keep changing between two and five. Git declares two and an HPA is enabled.

Explain the competing controllers, how drift detection sees the field, and when an ignore rule is appropriate. Do not simply disable reconciliation.

## Scenario 3 — Works with Helm, fails with Flux

`helm upgrade` from your laptop succeeds, but Flux gets `403` pulling the same private OCI chart.

Identify the different identities and credential paths. Propose verification commands and a keyless fix with repository-scoped IAM.

## Scenario 4 — Deleted from Git, still in cluster

A ConfigMap removed from the repository remains live.

Investigate `prune`, ownership labels/annotations, Kustomization scope, suspension, and whether another reconciler owns it.

## Scenario 5 — Dependency deadlock

Infrastructure and app Kustomizations both show dependency-not-ready. Each indirectly depends on the other.

Draw the graph, explain why retries cannot solve it, and redesign bootstrapping into an acyclic order.

## Scenario 6 — Secret exposed in history

A secret was committed as base64 YAML and later replaced with SOPS ciphertext.

Explain why the incident is not resolved, what must be rotated, how Git history and clones affect exposure, and how to prevent recurrence. Do not print or recover the secret.

## Scenario 7 — Rollback keeps disappearing

An operator runs `helm rollback`; the application recovers briefly, then Flux returns it to the bad version.

Explain desired state, Helm storage, and the durable Git-based recovery. Discuss when suspension may be justified during mitigation.

## Scenario 8 — Promotion design

Design promotion across development, staging, and production for an OCI chart and immutable image. Compare pull requests that update tags/digests, environment overlays, branches, and automatic image updates.

## Scenario 9 — CRD upgrade

A dependency introduces a new CRD version, but a normal Helm upgrade does not update the CRD as expected.

Explain Helm's CRD lifecycle constraints, ordering, backward compatibility, migration, and why CRDs often belong in a separately managed infrastructure layer.

## Scenario 10 — Slow reconciliation

A team expects instant deployment, but configured intervals are ten minutes.

Explain source and downstream intervals, event-driven reconciliation, webhook Receivers, explicit reconcile, rate limits, and why reducing every interval to seconds is not automatically a good design.

## Answer structure

Use this structure consistently:

1. **Clarify:** what changed, when, and in which environment?
2. **Hypothesis:** earliest likely failing layer.
3. **Evidence:** conditions, revisions, events, history, and logs.
4. **Impact:** new rollout blocked, existing workload affected, or both?
5. **Mitigation:** safest immediate action.
6. **Durable fix:** Git/IAM/chart/configuration change.
7. **Prevention:** validation, policy, alert, or design improvement.
