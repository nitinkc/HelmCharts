# Flux interview labs

The reference manifests are in `examples/interview-scenarios/`.

## Lab F1 — Dependency ordering

Create `infrastructure` and `apps` Flux Kustomizations. Make `apps` depend on `infrastructure`, then deliberately break infrastructure readiness.

Observe:

```bash
flux get kustomizations -A
kubectl describe kustomization apps -n flux-system
```

Explain why `dependsOn` orders readiness, not arbitrary completion, and why circular dependencies never converge.

## Lab F2 — `valuesFrom` precedence

Apply the example ConfigMaps and HelmRelease. Predict the final `replicaCount` when two ConfigMaps and inline values conflict. Remove the inline value, reconcile, and observe the next winner.

```bash
flux reconcile helmrelease interview -n "$APP_NAMESPACE" --with-source
kubectl get helmrelease interview -n "$APP_NAMESPACE" -o yaml
helm get values interview -n "$APP_NAMESPACE" --all
```

## Lab F3 — Drift ignore rule

Enable drift detection, manually change replicas, and verify correction. Then add an ignore rule for `/spec/replicas`, repeat the edit, and explain when an HPA-compatible ignore is justified versus when it hides unauthorized drift.

## Lab F4 — Prune and lifecycle

Commit a ConfigMap through a Flux Kustomization with `prune: true`. Remove it from Git and predict the result. Repeat after annotating the object with the Flux prune-disabled policy. Explain ownership and orphaning risks.

## Lab F5 — Least-privilege reconciliation

Create a namespace-scoped Kubernetes service account and Role that can manage ConfigMaps but not Deployments. Set `serviceAccountName` on a test Kustomization, commit both resource types, and diagnose the partial authorization failure.

## Lab F6 — Secret design interview

Without storing a real secret, design a SOPS + Cloud KMS flow. Your answer must show:

- ciphertext in Git;
- who can decrypt;
- where decryption occurs;
- how key rotation and access revocation work;
- why base64 in a Kubernetes Secret is not encryption.

## Lab F7 — Source outage game day

Block or invalidate one source at a time: Git URL, OCI tag, and GAR IAM. For each, record the first non-Ready resource, condition reason, downstream impact, whether the existing app stays available, and durable recovery.
