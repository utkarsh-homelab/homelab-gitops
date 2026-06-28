# homelab-gitops

ArgoCD App of Apps — single source of truth for the homelab Kubernetes cluster.

## Structure

```
bootstrap/
  ├── argocd-install.yaml    # Raw manifests for initial ArgoCD install
  |
apps/
  ├── root.yaml              # Root App of Apps entry point
  ├── argocd.yaml            # ArgoCD self-management (sync wave -2)
  ├── metallb.yaml           # MetalLB Layer2 LoadBalancer (sync wave -1)
  ├── traefik.yaml           # Ingress controller (sync wave 0)
  ├── cert-manager.yaml      # TLS certs via Let's Encrypt DNS01 (sync wave 0)
  ├── csi-driver-nfs.yaml    # NFS-backed dynamic PVCs (sync wave 1)
  └── kube-prometheus-stack.yaml  # Monitoring stack (sync wave 1)
```
---

## Sync Waves

| Wave | Apps | Purpose |
|------|------|---------|
| -2   | argocd | Self-management |
| -1   | metallb | LoadBalancer IPs |
|  0   | traefik, cert-manager | Ingress + TLS |
|  1   | csi-driver-nfs, kube-prometheus-stack | Storage + monitoring |

---

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

# Apply root application (app-of-apps)
kubectl apply -f https://raw.githubusercontent.com/utkarsh-homelab/homelab-gitops/main/apps/root.yaml
```
---

## Detailed Guides

For Detailed Instructions on setting up ArgoCD refer to:
- [Guide - Manual](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_04-argocd-bootstrap-and-gitops-setup-manual.md)
- [Guide - Automated](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_05-automating-argocd-bootstrap.md)

### Detailed Guides for other Apps

- [Guide - CSI-Driver-NFS](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_06-csi-driver-nfs.md)
- [Guide - MetalLB](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_07-metallb.md)
- [Guide - Traefik + Cert-Manager](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_08-traefik-cert-manager.md)
- [Guide - Kube-Prometheus-Stack](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_09-kube-prometheus-stack.md)
---

## Companion Repo

Helm charts are maintained in [homelab-infra-charts](https://github.com/utkarsh-homelab/homelab-infra-charts).
