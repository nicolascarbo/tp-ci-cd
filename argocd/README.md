# ArgoCD Bootstrap

These manifests are applied **once** to a Minikube cluster to wire up the GitOps loop.

## Prerequisites

- Minikube running (`minikube start`)
- `kubectl` pointed at the Minikube cluster (`kubectl config use-context minikube`)
- `mise install` run at project root (installs all required tools)
- GHCR package for `ghcr.io/nicolascarbo/tp-ci-cd` set to **public** (or a pull secret configured in the cluster)

## Bootstrap

Install ArgoCD and register the application:

```bash
mise run argocd-install     # creates argocd namespace, applies manifests, waits (~2 min)
mise run argocd-bootstrap   # applies argocd/application.yaml
```

> `argocd-bootstrap` declares `argocd-install` as a dependency — running it directly triggers both in order.

## Verify

```bash
mise run argocd-status   # show sync and health status (may take ~60s for first sync)
```

## Access the UI

```bash
mise run argocd-ui       # prints credentials and opens port-forward to https://localhost:8080
```

Or retrieve the password separately:

```bash
mise run argocd-password
```

Login with username `admin` and the printed password.

> `argocd/namespace.yaml` is optional — the `CreateNamespace=true` sync option handles namespace creation automatically.

## GitOps Flow

1. Developer pushes to `master` → CI triggers
2. CI: lint / test / security / Helm validate (parallel)
3. CI: build & push `ghcr.io/nicolascarbo/tp-ci-cd:vX.Y.Z` to GHCR
4. CI: commit `chart/values.yaml` with `tag: "vX.Y.Z"` and `[skip ci]` → push to `master`
5. ArgoCD polls `master` (default: every 3 min) → detects new commit
6. ArgoCD: `helm upgrade` on Minikube with new image tag → rolling update
