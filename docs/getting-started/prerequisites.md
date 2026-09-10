# Prerequisites

## Required access

You need:

- a non-production GKE cluster and permission to retrieve credentials;
- cluster-admin permissions for Flux bootstrap;
- permission to enable APIs, create an Artifact Registry repository, and manage IAM in the selected GCP project;
- a GitHub personal account and a repository you can administer;
- a GitHub token suitable for Flux bootstrap, kept only in an environment variable.

## Required tools

```bash
for command in gcloud kubectl helm flux git; do
  command -v "$command" >/dev/null || echo "Missing: $command"
done

gcloud version
kubectl version --client
helm version
flux version --client
git --version
```

Use Helm 3.8 or newer for native OCI support. Install current stable releases through the official instructions for your operating system.

## GCP prerequisites

```bash
gcloud auth list
gcloud config get-value project
gcloud services enable \
  container.googleapis.com \
  artifactregistry.googleapis.com \
  iamcredentials.googleapis.com \
  --project "$PROJECT_ID"
```

!!! note
    API enablement and IAM changes can take several minutes to propagate. Diagnose the actual permission before repeatedly changing bindings.

## Flux preflight

Run this only after connecting to the intended cluster:

```bash
flux check --pre
```

Do not proceed to bootstrap until every mandatory check passes.
