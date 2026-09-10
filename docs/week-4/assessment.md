# GitOps operations knowledge check and Gate 4

## Operations quiz

1. How does drift differ from an invalid desired state?
2. Why should diagnosis proceed from sources toward workloads?
3. What remains running when an `OCIRepository` tag becomes unavailable?
4. Which evidence distinguishes an OCI authentication failure from a Helm hook failure?
5. Why can a manual `helm rollback` be temporary under Flux?
6. What does suspending a `HelmRelease` change, and what does it not change?
7. When is explicit reconciliation preferable to waiting for the interval?
8. Compare credential exposure in GitLab CI push and Flux pull delivery.

## Final synthesis quiz

1. Trace a values commit to a Pod update through every Flux resource.
2. Explain Helm values precedence with a dependency example.
3. Explain why Workload Identity is preferred to a JSON key.
4. Assign five likely failures to their owning controllers.
5. Describe a safe rollback for a bad chart version.
6. Explain what each readiness gate proved that the previous gate did not.

## Gate 4 live demonstration

Ask a peer to choose, without telling you in advance:

- replica drift;
- a nonexistent OCI tag;
- a failed Helm hook;
- suspended reconciliation.

You must:

1. identify the scenario using status, conditions, events, and logs;
2. name the owning controller;
3. recover through Git when desired state is wrong;
4. prove all Flux resources and the workload are Ready;
5. defend your Flux-versus-CI comparison under follow-up questions.

**Pass:** correct diagnosis before remediation, durable recovery, and a complete explanation of control and credentials.

After passing, use the [cleanup guide](../reference/cleanup.md) or continue with image automation, environment overlays, or progressive delivery as a separate track.
