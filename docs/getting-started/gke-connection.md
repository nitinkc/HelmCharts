# Connect to your existing GKE cluster

The course never creates or deletes your cluster.

## Retrieve credentials

```bash
gcloud container clusters get-credentials "$CLUSTER_NAME" \
  --location "$CLUSTER_LOCATION" \
  --project "$PROJECT_ID"
```

The `--location` flag works for regional and zonal clusters.

## Verify context and identity

```bash
kubectl config current-context
kubectl cluster-info
kubectl auth can-i '*' '*' --all-namespaces
kubectl get nodes -o wide
```

The expected context normally contains the project, location, and cluster name. Stop if it names another cluster.

## Create the lab namespace

```bash
kubectl create namespace "$APP_NAMESPACE" --dry-run=client -o yaml | kubectl apply -f -
kubectl get namespace "$APP_NAMESPACE"
```

## Re-entry checklist

Run this at the beginning of every lab session:

```bash
gcloud config set project "$PROJECT_ID"
gcloud container clusters get-credentials "$CLUSTER_NAME" \
  --location "$CLUSTER_LOCATION" \
  --project "$PROJECT_ID"
kubectl config current-context
kubectl get namespace "$APP_NAMESPACE"
```

!!! danger "Context first"
    Never copy a mutating `kubectl`, `helm`, or `flux` command until the current context is visibly the intended learning cluster.
