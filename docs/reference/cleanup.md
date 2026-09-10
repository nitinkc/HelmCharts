# Cleanup

Cleanup is optional. These labs reuse your existing GKE cluster; **do not delete the cluster**.

!!! danger "Review before deletion"
    The commands below remove resources. Confirm every variable, current context, and resource name first. Run only the sections for resources you created specifically for this course.

## Inventory

```bash
kubectl config current-context
flux get all -A
helm list -A
gcloud artifacts repositories describe "$GAR_REPOSITORY" \
  --location "$GAR_LOCATION" --project "$PROJECT_ID"
gcloud iam service-accounts describe "$GSA_EMAIL" --project "$PROJECT_ID"
```

## Remove the app through Git

Delete the app declarations from the Flux bootstrap repository, commit, and push. Let pruning remove managed resources, then verify:

```bash
flux reconcile source git flux-system
flux reconcile kustomization apps --with-source
kubectl get namespace "$APP_NAMESPACE"
```

If the `apps` Kustomization itself was removed, verify all intended app resources are gone before continuing.

## Uninstall Flux

Only if no other workloads use this Flux installation:

```bash
flux uninstall --dry-run
flux uninstall
```

The first command previews resources. Read it before running the second.

## Remove Workload Identity bindings

Preview current policies first:

```bash
gcloud iam service-accounts get-iam-policy "$GSA_EMAIL" --project "$PROJECT_ID"
gcloud artifacts repositories get-iam-policy "$GAR_REPOSITORY" \
  --location="$GAR_LOCATION" --project="$PROJECT_ID" \
  --flatten='bindings[].members' \
  --filter="bindings.members:serviceAccount:${GSA_EMAIL}"
```

Use the corresponding `gcloud ... remove-iam-policy-binding` commands only after confirming the exact member and role shown in the Flux on GKE topic. Deleting the dedicated Google service account also removes its bindings, but do that only if it is not reused elsewhere.

## Artifact Registry

List artifacts and estimate whether the repository is dedicated to this course:

```bash
gcloud artifacts docker images list \
  "${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}" \
  --include-tags --project "$PROJECT_ID"
```

Do not delete a shared repository. If it is dedicated to this course, use the Google Cloud console or `gcloud artifacts repositories delete` only after reviewing the repository name, location, and all contained artifacts.

## GitHub repository

Flux bootstrap may have created the repository. Archive it for portfolio evidence or delete it from GitHub settings only after confirming nothing else uses it. Unsetting `GITHUB_TOKEN` removes it from the current shell:

```bash
unset GITHUB_TOKEN
```
