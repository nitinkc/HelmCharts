# Flux reconciliation deep dive

## Source artifacts and revisions

Source-controller fetches external content and publishes an artifact plus revision. Downstream controllers consume that artifact. Separate intervals mean source fetch, manifest application, and Helm reconciliation are related but distinct loops.

## Kustomization behavior

Important fields interviewers expect you to distinguish:

- `path`: directory built from the source artifact;
- `prune`: garbage-collect managed objects removed from desired state;
- `wait`: health-assess all reconciled resources;
- `healthChecks`: assess selected resources when `wait` is not used;
- `dependsOn`: block until another Flux Kustomization is Ready;
- `serviceAccountName`: reconcile with constrained Kubernetes RBAC;
- `decryption`: decrypt SOPS-encrypted Secrets before apply;
- `timeout` and `retryInterval`: bound health waits and failed retries.

Pruning is ownership-sensitive. Removing YAML from Git can delete the corresponding live object; suspension and deletion policies change lifecycle behavior.

## HelmRelease behavior

Beyond `chartRef` and inline values, know:

- `valuesFrom` merges ConfigMaps/Secrets in order, then inline values override them;
- `dependsOn` sequences releases by readiness;
- install and upgrade remediation can retry, uninstall, or roll back;
- tests can participate in release success and remediation;
- `driftDetection.mode: enabled` compares live state with Helm storage and corrects differences;
- `serviceAccountName` limits what the release can create;
- `targetNamespace` and `storageNamespace` solve different problems;
- `suspend` pauses future reconciliations but does not cancel an action already running.

## Event-driven versus interval-driven reconciliation

Flux reconciles periodically, when watched resources change, and when explicitly requested. Webhook Receivers can reduce Git polling latency, but intervals remain the eventual consistency safety net.

## Multi-environment reasoning

A common pattern separates:

```text
clusters/<cluster>/        bootstrap and cluster entrypoint
infrastructure/            controllers, CRDs, policies
apps/base/                 reusable app definitions
apps/staging|production/   environment overlays and pinned versions
```

Promote by pull request and version/digest change, not by editing the live cluster. Avoid branches as environments unless the team accepts harder cross-environment diffing and promotion.
