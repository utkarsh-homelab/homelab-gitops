# homelab-gitops

ArgoCD App of Apps — single source of truth for the homelab Kubernetes cluster.

## Structure

```
bootstrap/
  └── argocd-install.yaml    # Raw manifests for initial ArgoCD install
apps/
  ├── root.yaml              # Root App of Apps entry point
  └── argocd.yaml            # ArgoCD self-management (sync wave -2)
```

## Sync Waves

| Wave | Apps | Purpose |
|------|------|---------|
| -2   | argocd | Self-management |

## Bootstrap

```bash
kubectl create namespace argocd
kubectl apply -f https://raw.githubusercontent.com/utkarsh-homelab/homelab-gitops/main/bootstrap/argocd-install.yaml
kubectl apply -f https://raw.githubusercontent.com/utkarsh-homelab/homelab-gitops/main/apps/root.yaml
```

## Companion Repo

Helm charts are maintained in [homelab-infra-charts](https://github.com/utkarsh-homelab/homelab-infra-charts).
