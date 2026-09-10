# Topic-based learning roadmap

| Order | Topic | Hands-on outcome | Readiness gate |
|---|---|---|---|
| 1 | GitOps fundamentals | Trace reconciliation and inspect cluster prerequisites | Explain pull vs push and all core controllers |
| 2 | Advanced Helm and OCI | Add a dependency, hook, test, package, and private OCI push | Install and test a versioned chart from Artifact Registry |
| 3 | Flux on GKE | Bootstrap GitHub, configure Workload Identity, and deploy an `OCIRepository` through a `HelmRelease` | A Git commit changes GKE without `helm upgrade` |
| 4 | GitOps operations | Exercise drift, failure, suspend/resume, and recovery | Diagnose and recover live scenarios without hints |
| 5 | Helm deep dive | Debug templates, add schema, test atomic upgrades, and build a render matrix | Explain rendering scope, lifecycle, quality, and security trade-offs |
| 6 | Flux deep dive | Exercise dependencies, `valuesFrom`, drift rules, pruning, and constrained RBAC | Design and troubleshoot a multi-layer reconciliation system |
| 7 | Interview preparation | Complete scenario drills and the capstone | Diagnose an unseen failure and defend the design under questioning |

## Suggested cadence for each topic

- **Concept session (60–90 min):** concepts and notes
- **Lab session A (2 hrs):** first lab block
- **Lab session B (2 hrs):** second lab block
- **Troubleshooting session (60–90 min):** failure analysis and challenge
- **Assessment session (45 min):** closed-notes knowledge check and gate

Keep a lab journal with the command, expected result, actual result, diagnosis, and fix. Screenshots alone do not demonstrate understanding; capture the condition or event that proves your conclusion.

## Scope boundary

This path intentionally excludes image automation, multi-tenant Flux, environment overlays, and progressive delivery. Finish the GitOps operations gate before adding those topics.

## Progress tracker

- [ ] Environment and GKE context verified
- [ ] Gate 1: GitOps mental model
- [ ] Gate 2: advanced Helm and private OCI
- [ ] Gate 3: automatic Git-to-GKE reconciliation
- [ ] Gate 4: drift/failure diagnosis and synthesis
- [ ] Helm templating and release-lifecycle deep dive complete
- [ ] Flux dependencies, pruning, values, RBAC, and secrets deep dive complete
- [ ] Interview capstone and unseen scenario drill complete
- [ ] Optional resources cleaned up
