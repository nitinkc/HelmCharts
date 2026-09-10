# GitOps operations hands-on labs

Run these only against the learning release. Capture conditions and events before recovery. The [GitOps operations solution](../solutions/gitops-operations.md) contains expected evidence.

## Lab 4.1 — Create and observe drift

Record the Git-declared replica count, then modify the live Deployment:

```bash
kubectl get deployment -n "$APP_NAMESPACE" -o wide
kubectl scale deployment -n "$APP_NAMESPACE" \
  --all --replicas=7
kubectl get deployment -n "$APP_NAMESPACE" --watch
```

Wait for reconciliation or request it:

```bash
flux reconcile helmrelease "$RELEASE_NAME" \
  -n "$APP_NAMESPACE" --with-source
kubectl get deployment -n "$APP_NAMESPACE" \
  -o custom-columns=NAME:.metadata.name,DESIRED:.spec.replicas,READY:.status.readyReplicas
flux events --for HelmRelease/${RELEASE_NAME} -n "$APP_NAMESPACE"
```

Explain why the Deployment controller initially creates seven replicas and why Flux later restores the Git-declared count.

## Lab 4.2 — Diagnose a missing chart version

In Git, change the `OCIRepository` tag to a version that does not exist, commit, and push. Observe without immediately reverting:

```bash
flux reconcile source git flux-system
flux get sources oci -A
kubectl describe ocirepository "$CHART_NAME" -n "$APP_NAMESPACE"
flux events --for OCIRepository/${CHART_NAME} -n "$APP_NAMESPACE"
flux get helmreleases -A
```

Answer:

1. Which resource becomes non-Ready first?
2. Does the existing release disappear?
3. Why can `helm-controller` not repair the source failure?

Recover with `git revert <bad-commit>`, push, reconcile, and prove readiness.

## Lab 4.3 — Diagnose a Helm action failure

Publish a new chart version whose post-upgrade hook intentionally calls a nonexistent Service, then point the `OCIRepository` to that version through Git.

Inspect:

```bash
flux get helmreleases -A
kubectl describe helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
helm history "$RELEASE_NAME" -n "$APP_NAMESPACE"
kubectl get jobs,pods -n "$APP_NAMESPACE"
kubectl logs -n "$APP_NAMESPACE" job/<HOOK_JOB_NAME>
flux logs --kind=HelmRelease --name="$RELEASE_NAME" \
  --namespace="$APP_NAMESPACE" --level=error
```

Compare this failure surface with Lab 4.2. Recover by reverting the Git commit to the last known-good tag.

## Lab 4.4 — Suspend, edit, and resume

```bash
flux suspend helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
kubectl scale deployment -n "$APP_NAMESPACE" --all --replicas=5
sleep 30
kubectl get deployment -n "$APP_NAMESPACE"
flux resume helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE"
flux reconcile helmrelease "$RELEASE_NAME" -n "$APP_NAMESPACE" --with-source
kubectl get deployment -n "$APP_NAMESPACE"
```

Explain why suspension is an operational pause, not a new source of truth.

## Lab 4.5 — Synthesis artifact

Write a one-page comparison of Flux and GitLab CI/CD covering:

- control direction and network initiation;
- location and lifetime of cluster credentials;
- audit trail and approval flow;
- drift detection;
- artifact promotion and rollback;
- failure evidence and operator workflow;
- one scenario where each approach is preferable.

## Completion checklist

- [ ] Drift is corrected to Git state.
- [ ] Missing-source and failed-release conditions are distinguished.
- [ ] Recovery is performed through Git.
- [ ] Suspend/resume behavior is explained.
- [ ] The delivery-model comparison uses evidence, not slogans.
