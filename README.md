# homelab-gitops

ArgoCD App of Apps — single source of truth for the homelab Kubernetes cluster.

## Structure

```
bootstrap/
  └── argocd-install.yaml           # Raw manifests for initial ArgoCD install

infra-apps/
  ├── infra-root.yaml               # Root App of Apps entry point for infra-apps
  ├── argocd.yaml                   # ArgoCD self-management (sync wave -2)
  ├── metallb.yaml                  # MetalLB Layer2 LoadBalancer (sync wave -1)
  ├── traefik.yaml                  # Ingress controller (sync wave 0)
  ├── cert-manager.yaml             # TLS certs via Let's Encrypt DNS01 (sync wave 0)
  ├── csi-driver-nfs.yaml           # NFS-backed dynamic PVCs (sync wave 1)
  ├── kube-prometheus-stack.yaml    # Monitoring stack (sync wave 1)
  ├── alertmanager-discord-webhook.yaml # Alertmanager → Discord (sync wave 2)
  ├── local-path-provisioner.yaml   # Local path block storage for Kafka (sync wave 1)
  ├── strimzi-kafka.yaml            # Strimzi Kafka operator (sync wave 2)
  ├── cloudnative-pg.yaml           # CloudNativePG PostgreSQL operator (sync wave 2)
  ├── tenant-rbac.yaml              # Tenant Access (sync wave -2)
  └── apps-root.yaml                # Root App of Apps entry point for self-hosted applications (sync wave 5)

apps/
  ├── homarr.yaml                   # Homarr Dashboard (sync wave 10)
  ├── calibre-web-automated.yaml    # CWA Digital Library Manager (sync wave 10)
  ├── navidrome.yaml                # Navidrome Music Server (sync wave 10)
  ├── slskd.yaml                    # slskd Soulseek Client (sync wave 10)
  ├── kafka-cluster.yaml            # Kafka KRaft single-node cluster (sync wave 10)
  └── postgres-cluster.yaml         # PostgreSQL 16 single-node cluster (sync wave 10)
```
---

## Sync Waves

| Wave | Apps | Purpose |
|------|------|---------|
| -2   | argocd | Self-management |
| -2   | tenant-rbac | Tenant access |
| -1   | metallb | LoadBalancer IPs |
|  0   | traefik, cert-manager | Ingress + TLS |
|  1   | csi-driver-nfs, kube-prometheus-stack, local-path-provisioner | Storage + monitoring |
|  2   | alertmanager-discord-webhook | Alertmanager → Discord notifications |
|  2   | strimzi-kafka | Kafka operator |
|  2   | cloudnative-pg | PostgreSQL operator |
|  5   | apps-root | App of Apps for self-hosted applications |
|  10  | homarr | Homelab Dashboard |
|  10  | calibre-web-automated | CWA Digital Library Manager |
|  10  | navidrome | Music streaming (Subsonic-compatible) |
|  10  | slskd | Soulseek client with web UI |
|  10  | kafka-cluster | Kafka KRaft single-node cluster |
|  10  | postgres-cluster | PostgreSQL 16 single-node cluster |

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

### Detailed Guides for Infra Apps

- [Guide - CSI-Driver-NFS](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_06-csi-driver-nfs.md)
- [Guide - MetalLB](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_07-metallb.md)
- [Guide - Traefik + Cert-Manager](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_08-traefik-cert-manager.md)
- [Guide - Kube-Prometheus-Stack](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_09-kube-prometheus-stack.md)
- [Guide - Strimzi Kafka](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_10-strimzi-kafka.md)
- [Guide - CloudNativePG PostgreSQL](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_11-cloudnative-pg.md)
- [Guide - Alertmanager Discord Webhook](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-02_12-alertmanager-discord-webhook.md)

### Detailed Guides for Self-Hosted Apps

- [Guide - Homarr](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-03_01-homarr-dashboard.md)
- [Guide - CWA](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-03_02-calibre-web-automated.md)
- [Guide - Navidrome + slskd](https://github.com/utkarsh-homelab/homelab-docs/blob/main/guides/guide-03_03-navidrome-slskd.md)

---

## Companion Repos

Helm charts for infra apps are maintained in [homelab-infra-charts](https://github.com/utkarsh-homelab/homelab-infra-charts).

Helm charts for self-hosted apps are maintained in [homelab-app-charts](https://github.com/utkarsh-homelab/homelab-app-charts).
