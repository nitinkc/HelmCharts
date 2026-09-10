# GitOps fundamentals labs

These labs are read-only. Write answers in your lab journal before checking the [GitOps fundamentals solution](../solutions/week-1.md).

## Lab 1.1 — Map push and pull delivery

**Goal:** reason about control flow and credentials.

Draw two diagrams for the same change, `replicaCount: 1` to `replicaCount: 3`:

1. GitLab CI runs `helm upgrade` against GKE.
2. Flux running in GKE observes a Git commit.

For each arrow, annotate:

- who initiates the network connection;
- where Kubernetes credentials exist;
- where failure is reported;
- what notices a later manual edit.

**Validation:** your Flux diagram must include Git, `source-controller`, `kustomize-controller`, `helm-controller`, Artifact Registry, and the Kubernetes API.

## Lab 1.2 — Inspect the cluster as observed state

```bash
kubectl config current-context
kubectl get namespaces
kubectl get deployments,replicasets,pods,services -A
kubectl get events -A --sort-by='.lastTimestamp' | tail -n 20
```

Answer:

1. Which resources declare intent and which represent runtime realization?
2. If a Pod is deleted, which controller recreates it?
3. Is that behavior GitOps? Explain why or why not.

## Lab 1.3 — Flux preflight without installation

```bash
flux check --pre
kubectl api-resources | grep -E 'deployment|service|customresourcedefinition'
kubectl auth can-i create customresourcedefinitions.apiextensions.k8s.io
kubectl auth can-i create clusterroles.rbac.authorization.k8s.io
```

Record each preflight result. Do not bootstrap Flux in this topic.

## Lab 1.4 — Failure ownership game

Name the first controller and command you would inspect for each scenario:

| Scenario | Controller | First evidence |
|---|---|---|
| Git cannot be cloned | ? | ? |
| OCI chart tag is missing | ? | ? |
| Helm install hook fails | ? | ? |
| Deployment creates no Ready Pods | ? | ? |
| Alert cannot reach its receiver | ? | ? |

## Completion checklist

- [ ] I drew both delivery models from memory.
- [ ] I can separate Kubernetes reconciliation from GitOps reconciliation.
- [ ] `flux check --pre` has no unexplained failure.
- [ ] I assigned every failure to the correct owner.
