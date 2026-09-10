# Helm + Flux GitOps Learning Path

A topic-based, hands-on curriculum for advanced Helm and Flux GitOps on an existing GKE cluster. The course uses the included `my-first-chart`, GitHub, private Google Artifact Registry, and GKE Workload Identity.

## Curriculum

The MkDocs site includes:

- GitOps pull versus push and Flux controller architecture
- Helm dependencies, hooks, tests, values precedence, and OCI packaging
- Flux bootstrap for GitHub
- Private Artifact Registry access without service-account keys
- `OCIRepository` and `HelmRelease` reconciliation
- Drift, failure, remediation, suspend/resume, quizzes, and readiness gates
- Separate solutions, troubleshooting, command reference, and cleanup
- Helm templating, schema, atomic lifecycle, security, and render-matrix deep dives
- Flux dependencies, `valuesFrom`, pruning, drift rules, RBAC, and secret-design labs
- Forty interview questions, ten scenario drills, and a portfolio capstone

The original source plan is retained in [Helm_Flux_GitOps_Plan.md](Helm_Flux_GitOps_Plan.md).

## Run the documentation locally

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
mkdocs serve
```

Open <http://127.0.0.1:8000>.

## Validate the site

```bash
mkdocs build --strict
```

## Existing charts

- `my-first-chart/` is the progressive lab chart.
- `todo-app/` remains available as an additional example.
- Existing packaged `.tgz` files and `index.yaml` are legacy chart-repository artifacts; the curriculum teaches OCI publication to Artifact Registry.

Start with the [course home page](docs/index.md) and define your own GCP project, GKE cluster, location, Artifact Registry, and GitHub variables before running any lab command.
