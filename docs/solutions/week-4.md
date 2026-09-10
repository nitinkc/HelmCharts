# GitOps operations solutions

## Drift

Immediately after `kubectl scale`, the Deployment controller treats seven replicas as live desired state and creates Pods. On the next Helm reconciliation, Flux compares the release manifest with live resources and restores the Git-declared count. Evidence includes the replica transition and a successful Helm reconciliation event.

## Missing OCI tag

Expected chain:

- `OCIRepository` becomes non-Ready with an artifact/tag resolution error;
- `HelmRelease` cannot consume a new source artifact;
- the last successfully installed workload normally remains;
- `source-controller`, not `helm-controller`, owns the source error.

Recover durably:

```bash
git log --oneline -5
git revert <BAD_COMMIT_SHA>
git push
flux reconcile source git flux-system
flux reconcile kustomization apps
flux get sources oci -A
flux get helmreleases -A
```

## Failed hook

A chart can fetch successfully while a Helm action fails. In that case, `OCIRepository` is Ready, while `HelmRelease` reports a failed install/upgrade or remediation condition. The failed Job's Pod logs provide application-level detail.

Use this order:

```bash
flux get sources oci -A
flux get helmreleases -A
kubectl describe helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
helm history "$RELEASE_NAME" -n "$APP_NAMESPACE"
kubectl get events -n "$APP_NAMESPACE" --sort-by='.lastTimestamp'
```

## Suspend/resume

While suspended, the manual replica edit can persist because `helm-controller` is not reconciling that release. Git remains authoritative; resume plus reconcile restores it. Suspension is useful during controlled incident work, but the operator must record and end the pause.

## Synthesis model

A strong comparison avoids saying one method always wins. Flux reduces the need for external cluster credentials and continuously handles drift. A CI push model can be simpler when deployment orchestration is already centralized in CI and drift correction is not required. Both need protected source control, artifact integrity, least privilege, approval controls, and observable failures.
