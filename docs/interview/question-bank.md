# Interview question bank

Answer aloud in 60–120 seconds. Use a concrete example and state trade-offs.

## Helm fundamentals

1. What happens from `helm upgrade` until Pods become Ready?
2. Explain chart version versus `appVersion` versus image tag.
3. Explain values precedence, including multiple `-f` files and `--set`.
4. How do `with` and `range` change dot, and when do you use `$`?
5. Why use `include` instead of `template` in many helper patterns?
6. What do `required`, `tpl`, and `lookup` do, and what risks do they introduce?
7. What is stored for a Helm release and where?
8. What is the difference among `--wait`, `--atomic`, and `--cleanup-on-fail`?
9. Why are CRDs difficult to upgrade through Helm?
10. What does `helm test` prove and not prove?
11. How would you force a Pod rollout after a ConfigMap change?
12. How would you test a chart across a matrix of optional features?

## Flux and GitOps

13. Trace a Git commit through Flux to a running Pod.
14. Distinguish `GitRepository`, `OCIRepository`, Flux `Kustomization`, and `HelmRelease`.
15. Distinguish a Flux Kustomization from `kustomization.yaml`.
16. What happens when Git is unavailable after an application is running?
17. How do `interval`, watched-resource events, webhook Receivers, and manual reconcile interact?
18. What does `prune` do, and when is it dangerous?
19. How do `wait`, `healthChecks`, and `dependsOn` differ?
20. How does `valuesFrom` precedence work?
21. Describe HelmRelease install and upgrade remediation choices.
22. How does Helm drift detection work, and when would you ignore a field?
23. Why is a manual Helm rollback usually not durable under Flux?
24. How would you structure infrastructure and applications across environments?

## Security and platform

25. Why is Workload Identity better than a service-account JSON key?
26. Which identity pulls a private OCI chart in this design?
27. How would you restrict what a Flux Kustomization or HelmRelease can create?
28. Why is a base64 Kubernetes Secret unsafe to commit?
29. Compare SOPS decryption with an external secret operator.
30. How do you protect the software supply chain from Git to chart to image?
31. What repository and branch controls matter for GitOps?
32. What is the blast radius of source-controller compromise?

## Troubleshooting and design

33. An `OCIRepository` is Ready but `HelmRelease` is not. Where do you look?
34. Git shows the new commit but the cluster does not. Walk through diagnosis.
35. A release is Ready but Pods are not. Which layer owns the failure?
36. Flux repeatedly reverts an HPA's replica changes. How do you reason about it?
37. A chart works locally but fails under Flux. List likely differences.
38. How do you promote a chart from staging to production without mutable tags?
39. When would CI push delivery be preferable to Flux?
40. What would you monitor and alert on in a production Flux installation?

## Evaluation rubric

A strong answer includes the control flow, owning component, observable evidence, security boundary, and at least one trade-off. A weak answer only defines a command or says to reinstall/reconcile without locating the failure.
