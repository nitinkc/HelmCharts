# Environment variables

Define these once per terminal. Replace the example values; do not commit tokens or credentials.

```bash
export PROJECT_ID="your-gcp-project-id"
export CLUSTER_NAME="your-existing-cluster-name"
export CLUSTER_LOCATION="us-central1"
export GAR_LOCATION="us-central1"
export GAR_REPOSITORY="helm-charts"
export GITHUB_USER="your-github-user"
export GITHUB_REPOSITORY="flux-gke-lab"
export FLUX_PATH="clusters/${CLUSTER_NAME}"
export APP_NAMESPACE="helm-flux-lab"
export RELEASE_NAME="web"
export CHART_NAME="my-first-chart"
export CHART_VERSION="0.1.0"
export GCP_SERVICE_ACCOUNT="flux-source"
```

`CLUSTER_LOCATION` accepts either a region such as `us-central1` or a zone such as `us-central1-a`.

## Validate values

```bash
printf '%-22s %s\n' \
  PROJECT_ID "$PROJECT_ID" \
  CLUSTER_NAME "$CLUSTER_NAME" \
  CLUSTER_LOCATION "$CLUSTER_LOCATION" \
  GAR_LOCATION "$GAR_LOCATION" \
  GAR_REPOSITORY "$GAR_REPOSITORY" \
  GITHUB_REPOSITORY "$GITHUB_REPOSITORY"

gcloud projects describe "$PROJECT_ID" --format='value(projectId)'
```

## Derived values

```bash
export GAR_HOST="${GAR_LOCATION}-docker.pkg.dev"
export OCI_REPOSITORY="${GAR_HOST}/${PROJECT_ID}/${GAR_REPOSITORY}/${CHART_NAME}"
export GSA_EMAIL="${GCP_SERVICE_ACCOUNT}@${PROJECT_ID}.iam.gserviceaccount.com"
```

If you open a new shell, reload all variables before continuing. An empty variable can make a command target the wrong resource or produce a misleading error.
