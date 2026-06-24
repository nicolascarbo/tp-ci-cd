# ArgoCD Bootstrap

These manifests are applied **once** to a Minikube cluster to wire up the GitOps loop.

## Prerequisites

- ArgoCD installed in the `argocd` namespace:
  ```bash
  kubectl create namespace argocd
  kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  kubectl wait --for=condition=available --timeout=120s deployment/argocd-server -n argocd
  ```
- `kubectl` pointed at the Minikube cluster (`kubectl config use-context minikube`)
- GHCR package for `ghcr.io/nicolascarbo/tp-ci-cd` set to **public** (or a pull secret configured in the cluster)

## Apply

```bash
kubectl apply -f argocd/application.yaml
```

> `argocd/namespace.yaml` is optional — the `CreateNamespace=true` sync option handles namespace creation automatically.

## Verify

```bash
# Check ArgoCD detected the app (may take ~60s for first sync)
kubectl get application tp-ci-cd -n argocd

# Access the ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Default password:
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

## GitOps Flow (Approach 1 — targetRevision Git)

1. Developer pushes to `master` → CI triggers
2. CI: lint / test / security / Helm validate (parallel)
3. CI: build & push `ghcr.io/nicolascarbo/tp-ci-cd:vX.Y.Z`
4. CI: commit `chart/values.yaml` with `tag: "vX.Y.Z"` and `[skip ci]` → push to `master`
5. ArgoCD polls `master` (default: every 3 min) → detects new commit
6. ArgoCD: `helm upgrade` on Minikube with new image tag → rolling update
