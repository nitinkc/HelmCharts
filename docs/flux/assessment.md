# Flux integration knowledge check and Gate 3

## Retrieval quiz

1. Which Helm values source has the highest precedence?
2. What does `helm test` run?
3. Which Flux controller fetches Git and OCI artifacts?
4. Why is an OCI chart represented by `OCIRepository`, not an HTTP `HelmRepository`?

## Flux integration quiz

1. What does bootstrap write to Git and install in the cluster?
2. Why must the Workload Identity annotation be persisted in Git?
3. Which IAM role permits private chart pulls without permitting pushes?
4. How is the Kubernetes service account mapped to the Google service account?
5. What is the relationship among `GitRepository`, Flux `Kustomization`, `OCIRepository`, and `HelmRelease`?
6. Which condition would you inspect if the chart tag does not exist?
7. What three pieces of evidence prove a Git commit caused the cluster change?
8. Why is a Ready Deployment insufficient proof that every Flux source is healthy?

## Gate 3

Make a new Git commit that changes only Helm values. Without invoking `helm install` or `helm upgrade`:

1. show the Git revision reported by Flux;
2. show the Ready `OCIRepository` and its chart revision;
3. show the Ready `HelmRelease`;
4. show the updated Kubernetes workload;
5. explain the Git → source → release → Kubernetes chain;
6. explain why the cluster does not require a long-lived Google JSON key.

**Pass:** automatic reconciliation succeeds and you can explain every credential boundary.

Proceed to [GitOps operations](../operations/concepts.md).
