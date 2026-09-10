# Interview capstone

Build a portfolio-ready GitOps delivery repository for `my-first-chart` without following a solution line by line.

## Requirements

### Helm

- chart passes schema validation and linting;
- reusable labels/selectors use named templates;
- configuration changes trigger a checksum rollout;
- dependency can be enabled or disabled;
- hook and Helm test have bounded timeouts;
- resources, probes, and security context are explicitly configured;
- chart is packaged with an immutable version and pushed to private GAR.

### Flux

- GitHub bootstrap is declarative;
- source-controller uses GKE Workload Identity and repository-scoped GAR Reader;
- infrastructure and apps are separate Flux Kustomizations with dependency ordering;
- app uses `OCIRepository` and `HelmRelease`;
- non-secret values come from a ConfigMap through `valuesFrom`;
- drift detection and remediation behavior are explicit;
- reconciliation runs with the least Kubernetes RBAC practical for the lab.

### Operations

Demonstrate and recover from:

1. manual replica drift;
2. nonexistent chart tag;
3. invalid values rejected by schema;
4. failed Helm hook;
5. unauthorized GAR pull;
6. blocked dependency Kustomization.

## Evidence packet

Produce:

- architecture and reconciliation diagrams;
- repository tree and explanation;
- successful validation output;
- one table mapping each failure to controller, condition, evidence, impact, and fix;
- Git history showing pull-request-sized changes and Git-based recovery;
- a two-minute explanation of push versus pull delivery.

## Mock interview

Ask another person to introduce one failure without naming it. You have ten minutes to:

1. state your diagnostic hypothesis;
2. inspect upstream to downstream without random edits;
3. identify the owning controller;
4. propose immediate mitigation and durable correction;
5. explain the security implications.

## Scoring rubric

| Area | Strong signal | Weak signal |
|---|---|---|
| Mental model | Traces every controller and artifact | Calls Flux a pipeline runner |
| Helm | Explains scope, lifecycle, schema, and rollback | Recites install commands only |
| Flux | Uses conditions, dependencies, remediation, and drift settings | Reconciles repeatedly without diagnosis |
| Security | Uses Workload Identity, scoped IAM, RBAC, encrypted secrets | Commits credentials or grants broad admin |
| Operations | Preserves evidence and recovers through Git | Manually patches until green |
| Communication | States trade-offs and limitations | Claims GitOps solves all delivery problems |
