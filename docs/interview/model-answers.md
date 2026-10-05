# Interview question model answers

Use these to grade structure and correctness, not as scripts. Strong interview answers adapt to the stated environment.

## Helm fundamentals

??? success "1. From `helm upgrade` to Ready Pods"
    Helm merges values, validates the schema, renders templates, compares the release, and submits Kubernetes objects. Helm stores a new release revision. Kubernetes admission and workload controllers then create ReplicaSets and Pods; scheduling, image pulls, probes, and readiness determine availability. With `--wait`, Helm observes supported resources until ready or timeout.

??? success "2. Chart version, appVersion, and image tag"
    Chart `version` identifies the package and should change when chart content changes. `appVersion` is informational application metadata. The image tag/digest selects the actual container. They may align, but Helm does not enforce that relationship.

??? success "3. Values precedence"
    Dependency defaults are overridden by parent values, user `-f` files apply left to right, and `--set`/`--set-string` normally win. Inspect computed values with `helm get values --all`; avoid excessive `--set` for structured data.

??? success "4. `with`, `range`, and `$`"
    Both constructs rebind dot to the current object or item. `$` remains the root context, so `$.Release.Name` still works inside nested scopes. Variables can preserve other objects before scope changes.

??? success "5. `include` versus `template`"
    `include` returns rendered text and can be piped through functions such as `nindent`. The `template` action inserts output directly and is less composable. Prefix helper names because named templates are globally visible across charts.

??? success "6. `required`, `tpl`, and `lookup`"
    `required` fails rendering for a missing mandatory value. `tpl` evaluates a string as a template, which is flexible but expands the trust surface. `lookup` queries a live cluster and makes rendering environment-dependent; server-side dry run is needed to test it realistically.

??? success "7. Helm release storage"
    Helm stores release revisions in the target cluster, as Secrets by default, containing rendered release information and values. Access to those Secrets can expose sensitive rendered configuration, so RBAC and secret design matter.

??? success "8. `--wait`, `--atomic`, and `--cleanup-on-fail`"
    `--wait` waits for supported resources; Jobs require the relevant wait behavior. `--atomic` waits and rolls back a failed upgrade or removes a failed install. `--cleanup-on-fail` removes resources newly created by a failed upgrade but does not provide the same complete rollback semantics.

??? success "9. CRD lifecycle"
    CRDs in `crds/` are installed before templates, but Helm does not safely upgrade or delete them automatically because CRD changes can affect every custom resource and other releases. Plan compatibility, migrations, and often manage CRDs separately.

??? success "10. `helm test` boundaries"
    It runs resources carrying the test hook annotation. It proves only the assertions encoded—for example, Service reachability—not load capacity, durability, external ingress, or every dependency.

??? success "11. ConfigMap-triggered rollout"
    Hash the rendered ConfigMap into a Pod-template annotation. A config change changes the Deployment's Pod template, creating a rollout. This is useful where the application or consumption method does not reload configuration dynamically.

??? success "12. Chart test matrix"
    Lint and render combinations of optional dependencies, autoscaling, ingress, security settings, and boundary values. Add Kubernetes schema/policy validation and an ephemeral install, test, upgrade, and rollback path. Fail the matrix on the first invalid combination.

## Flux and GitOps

??? success "13. Git commit to Pod"
    Source-controller fetches the Git revision. Kustomize-controller applies the desired Flux and Kubernetes objects. Source-controller may fetch an OCI chart; helm-controller reconciles the HelmRelease. Kubernetes workload controllers then create and ready Pods. Each layer exposes revision and conditions.

??? success "14. Four Flux resources"
    `GitRepository` and `OCIRepository` acquire external content. Flux `Kustomization` builds/applies manifests from a source. `HelmRelease` declares Helm lifecycle and values using a chart source. Sources produce artifacts; reconcilers consume them.

??? success "15. Flux Kustomization versus `kustomization.yaml`"
    The Flux CR is a controller instruction with source, path, interval, prune, health, and dependency policy. `kustomization.yaml` is a Kustomize build file listing resources and transformations. Flux can point at a directory containing that file.

??? success "16. Git outage after deployment"
    Existing workloads normally continue because Kubernetes state is already applied. The Git source becomes stale or non-Ready and new changes stop. Impact depends on artifact retention and downstream actions; alert on source readiness and age rather than assuming the app immediately fails.

??? success "17. Reconciliation triggers"
    Intervals provide eventual consistency. Watched ConfigMap/Secret changes and source revision events can trigger dependent reconciliation. Receivers reduce Git-event latency. Manual reconcile is an operational request, not the normal delivery mechanism.

??? success "18. Pruning"
    Prune garbage-collects objects previously managed by a Kustomization but removed from desired state. It is dangerous when ownership or path boundaries are unclear, when shared resources are included, or when deletion policy was not designed.

??? success "19. `wait`, `healthChecks`, and `dependsOn`"
    `wait` assesses all reconciled resources. `healthChecks` selects particular resources when not broadly waiting. `dependsOn` gates one Kustomization on another Kustomization's Ready condition. They solve health observation and ordering, not arbitrary scripting.

??? success "20. `valuesFrom` precedence"
    Referenced ConfigMaps/Secrets merge in listed order, later entries override earlier ones, and inline values normally override afterward. `targetPath` has special replacement behavior. Watched labels can trigger immediate reconciliation after value-source changes.

??? success "21. HelmRelease remediation"
    Install and upgrade can retry failed actions; upgrade remediation may roll back, while failed installs may uninstall before retry depending on policy. Tests can participate in success. Choose based on whether preserving the last good release or rebuilding cleanly is safer.

??? success "22. Drift detection"
    Helm-controller compares live resources with the desired release manifest stored by Helm. Enabled mode corrects differences. Ignore fields only when another approved controller owns them, such as HPA-managed replicas; broad ignores can conceal unauthorized changes.

??? success "23. Manual rollback durability"
    The rollback changes cluster release state, but Git still declares the newer/bad state. Flux eventually reconciles back. Suspend only if needed for controlled mitigation, then revert or correct Git and resume.

??? success "24. Repository structure"
    Separate cluster entrypoints, infrastructure/CRDs, shared app bases, and environment-specific versions/overlays. Keep the dependency graph acyclic. Promote immutable chart versions and image digests through reviewed changes rather than live edits or mutable tags.

## Security and platform

??? success "25. Workload Identity"
    It maps a Kubernetes service account to Google IAM and supplies short-lived credentials. It avoids downloadable JSON keys, supports revocation and audit, and permits narrow role scope. Kubernetes RBAC and Google IAM are still separate controls.

??? success "26. Private OCI pull identity"
    Source-controller performs the pull. Its Kubernetes service account is mapped through GKE Workload Identity to the dedicated Google service account, which receives Artifact Registry Reader on the specific repository.

??? success "27. Restrict Flux reconciliation"
    Set `serviceAccountName` and give that service account namespace-scoped Roles where possible. Separate tenants/Kustomizations, restrict cross-namespace references, lock down controller defaults, and avoid cluster-admin reconciliation for ordinary apps.

??? success "28. Base64 Secret"
    Base64 is reversible encoding, not encryption. Anyone with repository access can decode it, and deletion does not remove Git history or clones. Use SOPS/KMS or an external secret system and rotate anything exposed.

??? success "29. SOPS versus external secret operator"
    SOPS stores encrypted desired Secret manifests in Git and Flux decrypts at apply time using authorized KMS identity. An external operator stores references in Git and retrieves values from a secret manager. Compare Git visibility, rotation, outage behavior, access boundaries, and operational dependencies.

??? success "30. Supply-chain protection"
    Protect branches and reviews, minimize bot permissions, sign/verify commits and artifacts where supported, scan dependencies and images, use immutable chart versions and image digests, restrict registry writes, record provenance/SBOMs, and enforce admission policy.

??? success "31. Repository controls"
    Require protected branches, reviews/CODEOWNERS for cluster paths, status checks, signed changes where required, least-privilege automation, short-lived credentials, audit retention, and separation of duties for sensitive environments.

??? success "32. Source-controller blast radius"
    It can read configured sources and produce artifacts consumed by downstream controllers; its cloud/repository credentials define external blast radius. It does not need unrestricted Kubernetes mutation itself, but compromised artifacts can influence reconcilers. Limit identities, namespaces, egress, and accepted sources.

## Troubleshooting and design

??? success "33. Ready OCI, failed HelmRelease"
    The chart was fetched, so inspect HelmRelease conditions, observed generation, history, hooks/tests, values, timeout, generated resources, events, and helm-controller logs. Do not troubleshoot GAR authentication first.

??? success "34. Commit present, cluster unchanged"
    Confirm source-controller observed the exact Git SHA, then inspect Flux Kustomization path/build/readiness, OCI revision if relevant, HelmRelease conditions, and finally workload rollout. The earliest stale or non-Ready layer owns the next investigation.

??? success "35. Ready release, unhealthy Pods"
    Move to Kubernetes: Deployment conditions, ReplicaSet, Pod events, scheduling, image pulls, probes, resources, and application logs. Flux may have successfully applied valid manifests even though runtime behavior is unhealthy.

??? success "36. HPA and drift conflict"
    The HPA legitimately changes `/spec/replicas`, while drift correction restores Git's value. Confirm ownership, then ignore only that field or adjust release design. Keep drift protection for unrelated fields.

??? success "37. Local Helm works, Flux fails"
    Compare identity/IAM, chart source and version, namespace, Kubernetes service account/RBAC, values and `valuesFrom`, cluster capabilities, Flux remediation/tests, dependency readiness, and local uncommitted files. Reproduce the exact rendered inputs.

??? success "38. Immutable promotion"
    Publish unique chart versions and immutable image digests. Promote with a reviewed Git change to the environment's pinned reference, run validation, and preserve rollback by reverting to a known-good reference.

??? success "39. When CI push may fit"
    CI push can be simpler for teams with mature centralized pipelines, short-lived federated deploy identities, imperative orchestration, or non-Kubernetes targets. Flux is strong for continuous drift correction and cluster-local pull. Choose based on control, risk, and operations—not fashion.

??? success "40. Production monitoring"
    Alert on source/Kustomization/HelmRelease non-Ready duration, stale revisions, reconciliation failures and latency, controller errors/restarts/resources, queue/rate-limit symptoms, notification failures, and deployment health. Include Git/chart revision in diagnostics and avoid alerting on transient retries alone.
