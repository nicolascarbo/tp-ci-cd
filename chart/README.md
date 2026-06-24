# Helm Chart

Deploys the Go app to Kubernetes.

## Structure

```
chart/
├── Chart.yaml           # Chart metadata
├── values.yaml          # Default configuration
└── templates/
    ├── _helpers.tpl     # Template helpers
    ├── deployment.yaml  # Deployment with liveness/readiness probes on /health
    └── service.yaml     # ClusterIP service on port 80 → container port 8080
```

## Key values

| Key | Default | Description |
|---|---|---|
| `image.repository` | `ghcr.io/nicolascarbo/tp-ci-cd` | Container image |
| `image.tag` | `"v1.0.3"` | Image tag — updated automatically by CI |
| `replicaCount` | `1` | Number of pod replicas |
| `resources.limits.cpu` | `100m` | CPU limit |
| `resources.limits.memory` | `128Mi` | Memory limit |
| `service.port` | `80` | Service port |
| `service.targetPort` | `8080` | Container port |

## Validate

```bash
mise run helm-lint      # helm lint ./chart
mise run helm-validate  # helm template | kubeconform against K8s 1.33.0
```
