# GitOps fundamentals solutions

## Lab 1.1

In push delivery, the CI runner initiates a connection to the Kubernetes API and therefore needs a deploy credential or federated identity. In pull delivery, `source-controller` initiates outbound access to Git and OCI while in-cluster controllers already have Kubernetes RBAC.

A complete pull trace is:

```text
Git commit → source-controller fetches Git → kustomize-controller applies Flux objects
Artifact Registry → source-controller fetches OCI chart → helm-controller executes release
helm-controller → Kubernetes API → Deployment controller → ReplicaSet → Pods
```

## Lab 1.2

A Deployment declares workload intent. ReplicaSets and Pods are progressively more concrete runtime realization. Deleting a Pod triggers the ReplicaSet controller because its observed count differs from its declared count. That is declarative Kubernetes reconciliation, but not GitOps by itself because Git is not involved.

## Lab 1.3

`flux check --pre` should confirm a supported Kubernetes version and required capabilities. Flux bootstrap needs permission to create CRDs and cluster-scoped RBAC. A failed `can-i` result requires the cluster administrator; do not work around it by granting yourself unreviewed privileges.

## Lab 1.4

| Scenario | Owner | First evidence |
|---|---|---|
| Git cannot be cloned | source-controller | `flux get sources git -A` |
| OCI tag missing | source-controller | `flux get sources oci -A` |
| Helm hook fails | helm-controller | `kubectl describe helmrelease` |
| No Ready Pods | Kubernetes workload controllers | Deployment conditions/events and Pod logs |
| Alert delivery fails | notification-controller | Alert/Provider conditions and controller logs |
