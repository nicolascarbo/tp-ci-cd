# TP CI/CD

A practical project implementing a full CI/CD chain: a Go web app built, tested, scanned, published to GHCR, and deployed to Minikube via ArgoCD (GitOps).

```
push to master
      │
      ▼
┌──────────────────────────────────────────┐
│              GitHub Actions CI           │
│  lint+test ──┐                           │
│  security  ──┼──► publish (master only)  │
│  helm-val  ──┘       │                   │
└──────────────────────┼────────────────────┘
                       │ image + values.yaml commit
                       ▼
               ghcr.io/nicolascarbo/tp-ci-cd:vX.Y.Z
                       │
                       ▼
                   ArgoCD (polls master every ~3 min)
                       │ helm upgrade
                       ▼
                   Minikube cluster
```

## Repository Layout

| Directory | Description |
|---|---|
| [`app/`](app/README.md) | Go web application source, Dockerfile, tests |
| [`chart/`](chart/README.md) | Helm chart for Kubernetes deployment |
| [`argocd/`](argocd/README.md) | ArgoCD Application manifest and bootstrap guide |
| [`.github/workflows/`](.github/workflows/ci.yml) | GitHub Actions CI/CD pipeline |
| `mise/tasks/` | mise task definitions (ci.toml / local.toml) |

## Prerequisites

Install [mise](https://mise.jdx.dev/getting-started.html), then run:

```bash
mise install   # installs go, golangci-lint, helm, trivy, kubeconform
```

## Quick Start

### CI tasks (same commands the pipeline runs)

```bash
mise run lint            # golangci-lint
mise run test            # go test ./...
mise run security-scan   # trivy filesystem scan
mise run helm-lint       # helm lint
mise run helm-validate   # helm template | kubeconform
```

### Local cluster tasks

```bash
mise run argocd-install    # install ArgoCD on Minikube
mise run argocd-bootstrap  # apply the ArgoCD Application manifest
mise run argocd-status     # check sync and health status
mise run argocd-ui         # open port-forward to the ArgoCD UI (https://localhost:8080)
mise run argocd-password   # print the admin password
mise run app-forward       # port-forward the app to http://localhost:8081
```

## CI/CD Pipeline Deep Dive

The pipeline is defined in [`.github/workflows/ci.yml`](.github/workflows/ci.yml).

**Trigger:** every push or pull request targeting `master`.

### Job graph

```
lint ──────┐
security ──┼──► publish  (master push only)
helm-val ──┘
```

The first three jobs run in parallel. `publish` only runs on direct pushes to `master` (not on PRs) and requires all three to pass first.

### `lint` job

Runs on `ubuntu-latest`. Installs tools via `jdx/mise-action` with caching, then:

1. `mise run lint` — golangci-lint on `./app`
2. `mise run test` — `go test ./...` in `./app`

### `security` job

Runs on `ubuntu-latest`. Installs tools via `jdx/mise-action`, then:

1. `mise run security-scan` — trivy filesystem scan of the whole repo, exits with code 1 on any HIGH or CRITICAL finding

### `helm-validate` job

Runs on `ubuntu-latest`. Installs tools via `jdx/mise-action`, then:

1. `mise run helm-lint` — `helm lint ./chart`
2. `mise run helm-validate` — renders Helm templates and pipes to `kubeconform` for schema validation against Kubernetes 1.33.0

### `publish` job

Runs on `ubuntu-latest`, gates on `lint`, `security`, and `helm-validate`, master-only.

**Step 1 — Auto-semver tag**
Reads the latest `vX.Y.Z` git tag, increments the patch number, and outputs the new tag (e.g. `v1.0.3` → `v1.0.4`). If no tag exists, starts at `v1.0.0`.

**Step 2 — Docker Buildx**
Sets up Docker Buildx for multi-platform image builds.

**Step 3 — Login to GHCR**
Authenticates with `ghcr.io` using `GITHUB_TOKEN` (no manual secret needed).

**Step 4 — Build and push**
Builds the image from `./app/Dockerfile` for both `linux/amd64` and `linux/arm64` (required for Minikube on Apple Silicon). Pushes two tags:
- `ghcr.io/nicolascarbo/tp-ci-cd:vX.Y.Z` (pinned, immutable)
- `ghcr.io/nicolascarbo/tp-ci-cd:latest`

Layer caching uses GitHub Actions cache (`type=gha`) to speed up subsequent builds.

**Step 5 — Git tag**
Creates and pushes the `vX.Y.Z` git tag to the repository.

**Step 6 — GitOps commit**
Updates the `tag:` field in `chart/values.yaml` with `sed`, commits with message `chore(gitops): update image tag to vX.Y.Z [skip ci]`, rebases on `master`, and pushes. The `[skip ci]` label prevents an infinite CI trigger loop. ArgoCD polls `master` every ~3 minutes and triggers a `helm upgrade` when it detects the new commit.

## GitOps Flow

```
Developer push to master
        │
        ▼
GitHub Actions CI (lint / test / security / helm-validate / publish)
        │
        ├── pushes image → ghcr.io/nicolascarbo/tp-ci-cd:vX.Y.Z
        │
        └── commits chart/values.yaml (tag: "vX.Y.Z") [skip ci] to master
                │
                ▼
          ArgoCD polls master every ~3 min
                │ detects new tag in values.yaml
                ▼
          helm upgrade tp-ci-cd ./chart on Minikube
                │
                ▼
          Rolling update → pods running vX.Y.Z
```
