# GitOps drift, failure, and operations

**Time:** 6–8 hours · **Outcome:** diagnose before changing anything

## Drift versus failed desired state

- **Drift:** desired state is valid, but live state differs. Reconciliation restores the declared state.
- **Failed desired state:** Git declares something the controllers cannot realize, such as a missing OCI tag or a failing hook. Reconciliation retries or remediates, but cannot invent a valid artifact or value.

Do not treat every non-Ready status as drift. Read the condition's `reason`, `message`, observed generation, and related events.

## Diagnostic ladder

Inspect from upstream to downstream:

1. Git source revision
2. Flux `Kustomization`
3. OCI source revision/authentication
4. `HelmRelease` conditions and Helm history
5. generated Kubernetes resources
6. workload events and container logs

Changing resources before locating the failing layer destroys evidence.

## Suspend and reconcile

Suspension pauses a controller's reconciliation of one resource; it does not make manual edits the new desired state. Explicit reconcile requests an immediate check instead of waiting for the interval.

```bash
flux suspend helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
flux resume helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
flux reconcile helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE" --with-source
```

## Rollback mental model

In GitOps, the durable rollback is normally a Git revert or a new commit restoring known-good desired state. A manual Helm rollback can be overwritten by Flux because Git still asks for the failed/new state.
