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

## Bootstrap Guide

```bash
# Install ArgoCD CRDs from upstream 
kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.4.4/manifests/crds/application-crd.yaml

kubectl create -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.4.4/manifests/crds/applicationset-crd.yaml

kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.4.4/manifests/crds/appproject-crd.yaml

# Create argocd namespace
kubectl create namespace argocd

# Apply the bootstrap YAML from GitHub, Use --server-side to avoid the 256KB annotation limit
curl -sL https://raw.githubusercontent.com/utkarsh-homelab/homelab-gitops/main/bootstrap/argocd-install.yaml \
  | kubectl apply --server-side -n argocd -f -

# Wait for ArgoCD to be ready
kubectl wait --for=condition=ready pod -n argocd -l app.kubernetes.io/name=argocd-server --timeout=300s
```

For Detailed Instructions refer to this [Guide](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_04-argocd-bootstrap-and-gitops-setup-manual.md)

## Companion Repo

Helm charts are maintained in [homelab-infra-charts](https://github.com/utkarsh-homelab/homelab-infra-charts).
