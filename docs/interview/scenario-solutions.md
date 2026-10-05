# Scenario drill model solutions

Attempt each [scenario drill](scenario-drills.md) first. These are model reasoning paths, not the only acceptable answers.

## 1. Green source, red release

- **Ruled out:** Git and chart acquisition are healthy at the reported revisions.
- **Inspect:** HelmRelease conditions and observed generation, `helm history`, hook/test Jobs, namespace events, generated workload status, and helm-controller logs.
- **Likely causes:** invalid values that still render, failed hook/test, immutable-field change, timeout, RBAC, or unhealthy workload under wait.
- **Impact:** the old release may remain or remediation may roll back/uninstall depending on policy.
- **Fix:** correct the chart/values in Git or publish a corrected immutable chart version; do not bypass Flux with a lasting manual upgrade.
- **Prevention:** schema, render matrix, ephemeral upgrade/test, and explicit remediation.

## 2. Drift or another controller?

The HPA and Helm drift correction both write replicas. Confirm HPA events and Helm drift logs before changing policy. If HPA ownership is intended, omit fixed replica ownership where the chart supports it or ignore only `/spec/replicas` for the target Deployment. Do not disable all drift detection; that would hide unrelated edits.

## 3. Local Helm works, Flux gets 403

The laptop uses local gcloud/Docker credentials, while Flux uses source-controller's workload identity. Verify the OCI URL, GKE workload pool, Kubernetes service-account annotation, Workload Identity User binding, and repository IAM. Grant the mapped Google service account only `roles/artifactregistry.reader` on the required GAR repository, restart source-controller after identity changes, and confirm the OCIRepository condition.

## 4. Deleted from Git, still in cluster

Check whether the correct Flux Kustomization owns the ConfigMap, whether `prune` is enabled, whether reconciliation is suspended/failed, and whether prune-disable annotations or deletion policy apply. Also check whether another release or Kustomization manages an object with the same identity. Correct ownership and policy in Git; do not delete first and erase the evidence.

## 5. Dependency deadlock

Draw edges from dependent to prerequisite. A cycle means neither prerequisite can become Ready, so retries cannot progress. Split bootstrap into acyclic layers such as Flux bootstrap → CRDs/controllers → policies/secrets infrastructure → applications. A dependency graph must be a directed acyclic graph.

## 6. Secret exposed in history

Replacing plaintext with ciphertext does not revoke the original value from commits, forks, caches, clones, logs, or systems where it was used. Revoke/rotate the credential first, assess access and impact, then follow the organization's history-remediation process. Add secret scanning, protected review, SOPS/external-secret workflows, and least-privilege short-lived credentials. Never reproduce the exposed value during diagnosis.

## 7. Rollback keeps disappearing

The manual rollback changes Helm's live revision, but Git still requests the bad chart/value. Flux correctly returns to declared state. During an incident, suspension may preserve mitigation briefly, but the durable correction is a Git revert or new commit pinning a known-good immutable chart and image; then reconcile and resume.

## 8. Promotion design

Publish immutable chart versions and image digests once. Keep environment-specific desired references in reviewable Git paths or overlays. Promote by pull request from the tested reference into staging and production, with policy and health gates. Branch-per-environment can work but complicates cross-environment diffs; mutable tags weaken auditability. Image automation should open or make controlled Git changes rather than mutate clusters directly.

## 9. CRD upgrade

Helm installs CRDs from `crds/` but intentionally does not manage their upgrade/delete lifecycle like ordinary templates. Review conversion and backward compatibility, update the CRD before custom resources that need the new schema, back up data, and plan migrations. Managing CRDs in a separate infrastructure Kustomization gives explicit ordering and ownership; application releases can depend on its readiness.

## 10. Slow reconciliation

Identify which loop is delayed: Git source, Kustomization, OCI source, or HelmRelease. Webhook Receivers can trigger source reconciliation quickly, and artifact events propagate downstream; intervals retain eventual consistency if events are lost. Explicit reconcile is useful for diagnosis. Very short intervals increase API/registry traffic, rate limiting, log noise, and controller load, so tune from delivery objectives and observed metrics.

## Scenario grading rubric

Score each dimension from 0–2:

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| Ownership | Wrong component | Partially correct | Correct controller/layer |
| Evidence | Random commands | Some relevant status | Conditions, revisions, events, logs in order |
| Impact | Not addressed | Guessed | Distinguishes rollout and existing-app impact |
| Recovery | Manual-only | Immediate mitigation | Mitigation plus durable Git/IAM/design fix |
| Prevention | None | Generic advice | Specific validation, policy, monitoring, or architecture |
| Communication | Unstructured | Understandable | Concise hypothesis-to-trade-off narrative |

**10–12:** strong answer · **7–9:** acceptable but refine · **0–6:** repeat the corresponding lab.
