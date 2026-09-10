# Interview self-assessment

This interactive assessment covers Helm, Flux, GitOps security, and troubleshooting. Progress and answers are stored only in your browser's local storage.

<!-- mkdocs-quiz intro -->

## Helm

<quiz>
Inside `with .Values.image`, how do you access the release name?

- [ ] `.Release.Name`
- [x] `$.Release.Name`
- [ ] `.Values.Release.Name`
- [ ] `root.Release.Name`

`with` changes dot to the selected object, while `$` retains the root context.
</quiz>

<quiz>
Which value has the highest precedence?

- [ ] Dependency defaults
- [ ] Parent `values.yaml`
- [ ] The last `-f` file
- [x] `--set`

`--set` and `--set-string` override values files, parent values, and dependency defaults.
</quiz>

<quiz>
What does `--atomic` do during a Helm upgrade?

- [ ] Skips hooks
- [x] Waits and rolls back after failure
- [ ] Deletes release history
- [ ] Forces resource replacement

An atomic upgrade waits for readiness and rolls back when the operation fails.
</quiz>

<quiz>
Why add `values.schema.json` to a chart?

- [ ] Encrypt values
- [ ] Replace `values.yaml`
- [x] Reject invalid value types and ranges early
- [ ] Generate Kubernetes CRDs

Schema validation catches invalid input before a release reaches the Kubernetes API.
</quiz>

<quiz>
Why add a ConfigMap checksum to a Pod-template annotation?

- [ ] Encrypt configuration
- [x] Trigger a rollout when rendered configuration changes
- [ ] Make ConfigMaps immutable
- [ ] Reduce Helm revisions

A changed checksum changes the Pod template, so the Deployment controller creates a rollout.
</quiz>

<quiz>
Which statement about Helm CRDs is correct?

- [ ] They upgrade automatically like Deployments
- [ ] They are always deleted on uninstall
- [x] Their lifecycle needs special upgrade planning
- [ ] They belong in `values.yaml`

Helm deliberately avoids normal automatic upgrade and deletion behavior for CRDs because their blast radius extends beyond one release.
</quiz>

## Flux

<quiz>
Which controller fetches Git and OCI artifacts?

- [ ] helm-controller
- [x] source-controller
- [ ] kustomize-controller
- [ ] notification-controller

Source-controller acquires external content and publishes artifacts for downstream reconcilers.
</quiz>

<quiz>
What does `prune: true` do on a Flux Kustomization?

- [ ] Removes old Git commits
- [x] Deletes managed cluster objects removed from desired state
- [ ] Deletes failed Pods only
- [ ] Clears Helm history

Pruning garbage-collects objects previously managed by the Kustomization but no longer declared in its source.
</quiz>

<quiz>
How are HelmRelease `valuesFrom` sources merged?

- [ ] Alphabetically
- [ ] The first source always wins
- [x] Later sources override earlier ones, then inline values override
- [ ] Inline values have the lowest precedence

Order is significant. Inline values normally have the final precedence, while `targetPath` has special replacement behavior.
</quiz>

<quiz>
What does Flux Kustomization `dependsOn` guarantee?

- [ ] The dependency merely exists in Git
- [x] The dependency Kustomization is Ready before reconciliation proceeds
- [ ] Both Kustomizations reconcile simultaneously
- [ ] The dependency can never be pruned

`dependsOn` gates on the dependency's Ready condition. It cannot resolve a circular dependency.
</quiz>

<quiz>
An `OCIRepository` is non-Ready with HTTP `403`. What should you investigate first?

- [ ] Deployment readiness probes
- [ ] Helm template whitespace
- [x] Registry identity and IAM
- [ ] Git merge conflicts

Verify the OCI URL, source-controller identity, Workload Identity mapping, and repository-scoped Artifact Registry permissions.
</quiz>

<quiz>
Why can a manual `helm rollback` disappear under Flux?

- [ ] Helm does not support rollback
- [x] Flux reconciles the Git-declared desired state again
- [ ] Kubernetes deletes Helm Secrets
- [ ] Artifact Registry removes the chart

Git remains the durable desired state. Correct or revert Git instead of relying on an out-of-band rollback.
</quiz>

<quiz>
When is ignoring `/spec/replicas` in drift detection commonly justified?

- [ ] Always
- [x] When another approved controller such as an HPA owns that field
- [ ] Whenever Git is unavailable
- [ ] Whenever Pods are CrashLooping

Ignore only intentionally shared fields. Broad drift exclusions can hide unauthorized changes.
</quiz>

## Security and operations

<quiz>
Why prefer GKE Workload Identity over a service-account JSON key?

- [ ] It grants cluster-admin automatically
- [x] It uses short-lived credentials and avoids downloadable keys
- [ ] It makes IAM unnecessary
- [ ] It stores credentials safely in Git

Workload Identity supports short-lived credentials, IAM auditability, and revocation without distributing a key file.
</quiz>

<quiz>
Is the value in a base64 Kubernetes Secret encrypted?

- [ ] Yes
- [ ] Only on GKE
- [x] No, base64 is encoding
- [ ] Only when the Git repository is private

Base64 is reversible. Use SOPS/KMS or an external secret system for protected GitOps secret workflows.
</quiz>

<quiz>
Git contains the expected commit, but the application did not change. What is the strongest first diagnostic step?

- [ ] Delete and reinstall Flux
- [x] Confirm the Git source revision, then inspect each downstream condition
- [ ] Run `helm upgrade` manually
- [ ] Restart every controller

Start with the earliest source and move downstream. This preserves evidence and identifies the owning controller.
</quiz>

<quiz>
The source and HelmRelease are Ready, but Pods are not. Which layer owns the immediate failure investigation?

- [ ] GitHub
- [ ] Artifact Registry
- [x] Kubernetes workload and runtime controllers
- [ ] source-controller

Inspect Deployment and Pod conditions, events, scheduling, image pulls, probes, resources, and application logs.
</quiz>

<quiz>
Which action is a durable recovery for a nonexistent chart tag?

- [ ] Edit the live Deployment
- [ ] Restart source-controller repeatedly
- [x] Revert or correct the tag in Git
- [ ] Delete Helm history

Repair the declared source reference and let reconciliation restore a valid state.
</quiz>

<quiz>
Which is the strongest production promotion signal?

- [ ] A mutable `latest` tag
- [x] A reviewed Git change to an immutable chart version and image digest
- [ ] A manual cluster edit
- [ ] A developer's local values file

Immutable references and reviewed desired-state changes make promotion reproducible and auditable.
</quiz>

<quiz>
Which answer style provides the strongest interview signal?

- [ ] Name many commands rapidly
- [ ] Claim GitOps is always more secure
- [x] Explain ownership, evidence, impact, durable correction, and trade-offs
- [ ] Recommend reinstalling whenever uncertain

A structured diagnosis demonstrates mental models and engineering judgment, not command memorization.
</quiz>

<!-- mkdocs-quiz results -->

## Next step

For every incorrect response, explain why each alternative was wrong. Then answer the corresponding open question in the [question bank](question-bank.md) before reading its [model answer](model-answers.md).
