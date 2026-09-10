# Glossary

**Artifact** — Immutable content produced or fetched by a source controller and consumed by another controller.

**Chart** — Versioned Helm package containing templates, default values, and metadata.

**Controller** — A control loop that observes resources and acts to move observed state toward desired state.

**Desired state** — The declared configuration a controller attempts to realize.

**Drift** — A difference between declared desired state and live observed state.

**GitOps** — Operating a system through version-controlled, declarative desired state reconciled automatically by software agents.

**HelmRelease** — Flux resource declaring a Helm installation, values, timing, tests, and remediation policy.

**Hook** — Helm-managed resource executed for an install, upgrade, test, rollback, or delete lifecycle event.

**Kustomization (Flux)** — Flux resource that builds, applies, health-checks, and optionally prunes manifests from a source. It is distinct from a `kustomization.yaml` file.

**Observed state** — The state a controller currently sees in the target system.

**OCIRepository** — Flux source for an artifact stored in an OCI registry, including a Helm chart in Artifact Registry.

**Reconciliation** — Repeated comparison and correction of desired and observed state.

**Remediation** — Helm-controller behavior after failed install, upgrade, test, or rollback operations.

**Source of truth** — The authoritative desired-state location; in this course, the Git bootstrap repository plus the versioned chart artifact it references.

**Workload Identity Federation for GKE** — Mapping that lets Kubernetes workloads access Google APIs with short-lived credentials and IAM, without a downloaded service-account key.
