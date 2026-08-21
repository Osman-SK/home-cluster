# Home-Cluster

**Multi-node · Multi-architecture (arm64 + amd64) Kubernetes Homelab**  
Fully managed with GitOps · [status.oskcloud.net](https://status.oskcloud.net)

A production-style personal cluster running on real constrained hardware  
(Raspberry Pi 5 + HP x86 node). Everything is declarative, encrypted in Git,  
and continuously reconciled by Flux CD. No manual `kubectl apply` after bootstrap.

---

## Cluster at a glance

| Hostname   | Arch  | RAM   | Role                          | Notes                          |
|------------|-------|-------|-------------------------------|--------------------------------|
| **rpi5**   | arm64 | 16 GB | Control plane + primary workloads | Raspberry Pi 5                 |
| **hp-8200**| amd64 | 12 GB | Worker / storage              | HP Compaq 8200 (NFS + media)   |

Live node metrics, CPU/memory, pod counts and service health are available at  
**[status.oskcloud.net](https://status.oskcloud.net)**.

---

## Architecture

```
GitHub (this repo)
       │
       ▼
Flux CD (GitRepository + Kustomizations)
       │
       ├── apps/          → Linkding, Audiobookshelf, Obsidian, Webtop,
       │                   Prowlarr, qBittorrent, Cluster Dashboard
       ├── media/         → dedicated Kustomization for *arr stack
       ├── monitoring/    → kube-prometheus-stack (Prometheus + Grafana)
       └── infra/         → NFS CSI driver + Renovate
```

**Key design decisions**

- **GitOps only** — Flux is the single source of truth
- **Kustomize overlays** (`base` + `staging`) for environment separation
- **SOPS + age** — all secrets encrypted in Git; Flux decrypts at reconcile time
- **Zero-trust external access** — Cloudflare Tunnels (no inbound ports on the home network)
- **Traefik** as the IngressClass
- **Heterogeneous scheduling** — `nodeSelector` / arch affinity where needed  
  (media workloads pinned to the amd64 storage node, dashboard prefers arm64, etc.)
- **NFS CSI** for shared persistent storage (HDD-backed StorageClasses)
- **Renovate** running in-cluster for automated dependency/image updates
- **Pod hardening** — non-root, `allowPrivilegeEscalation: false`, resource requests/limits on most workloads

---

## Public endpoints

| Hostname                                      | Application              |
|-----------------------------------------------|--------------------------|
| [status.oskcloud.net](https://status.oskcloud.net) | Cluster Dashboard (live metrics) |
| [lds.oskcloud.net](https://lds.oskcloud.net)       | Linkding                 |
| [grs.oskcloud.net](https://grs.oskcloud.net)       | Grafana                  |
| [audiobooks.oskcloud.net](https://audiobooks.oskcloud.net) | Audiobookshelf     |

All external traffic is terminated by Cloudflare Tunnels + Access.

---

## Tech stack

| Category              | Component                                      |
|-----------------------|------------------------------------------------|
| Orchestration         | k3s (multi-arch)                               |
| GitOps                | Flux CD v2 (Kustomization + HelmRelease)       |
| Manifests             | Kustomize                                      |
| Secrets               | SOPS + age                                     |
| Ingress               | Traefik                                        |
| External access       | Cloudflare Tunnel (`cloudflared`)              |
| Observability         | kube-prometheus-stack (Prometheus, Grafana)    |
| Storage               | NFS CSI + PVCs                                 |
| Dependency updates    | Renovate (in-cluster CronJob)                  |
| Applications          | 7+                                             |

---

## Repository structure

```
apps/
├── base/                     # Shared deployments, services, PVCs, namespaces
│   ├── audio-book-shelf/
│   ├── cluster-dashboard/
│   ├── linkding/
│   ├── media/                # namespace only
│   ├── obsidian/
│   ├── prowlarr/
│   ├── qbittorrent/
│   └── webtop/
└── staging/                  # Environment overlays (Ingress, Cloudflare Tunnels, SOPS secrets)
    ├── audio-book-shelf/
    ├── cluster-dashboard/
    ├── linkding/
    ├── media/
    ├── obsidian/
    └── webtop/

clusters/staging/
├── flux-system/              # Flux bootstrap
├── apps.yaml                 # Flux Kustomization → ./apps/staging
├── media.yaml                # Flux Kustomization → media stack
├── monitoring.yaml
├── infra.yaml
└── .sops.yaml

infra/controllers/
├── base/ + staging/          # NFS CSI HelmRelease + Renovate

monitoring/
├── controllers/              # kube-prometheus-stack HelmRepository + HelmRelease
└── configs/                  # Grafana TLS secret, etc.

dashboard/                    # Source of the status page (built into cluster-dashboard image)
```

All changes flow: **Git push → Flux reconcile (1–10 min) → SOPS decrypt → apply**.

---

## What this repository demonstrates

- Multi-architecture / multi-node Kubernetes on real hardware
- Full GitOps workflow (Flux + Kustomize + HelmRelease)
- Secrets-as-code with SOPS + age
- Zero-trust networking (Cloudflare Tunnels, no open ports)
- Shared storage with CSI
- Observability (Prometheus Operator + Grafana)
- Automated dependency management (Renovate)
- Security hygiene (non-root, restricted securityContexts, resource limits)
- End-to-end ownership: bootstrap → applications → monitoring → external access

These are the same patterns used in production Platform / SRE / DevOps roles.

---

## About the author

**Osman Sarper Küçük**  
DevOps / Platform Engineer focused on reliable, scalable infrastructure.

Previously Web3 Data Specialist (onchain analytics, data pipelines).  
Now applying systems thinking to Kubernetes, automation, CI/CD and observability.

- Personal site: [osk.cool](https://osk.cool)
- LinkedIn: [osmansarperkucuk](https://www.linkedin.com/in/osmansarperkucuk)
- GitHub: [Osman-SK](https://github.com/Osman-SK)
- X: [@OSK546](https://x.com/OSK546)

---

## Notes for reviewers

- This is an **active personal lab**, not a polished open-source product.
- The cluster runs 24/7 and is used daily.
- Secrets are encrypted; the public repository contains only ciphertext.
- Commits follow conventional `feat:` / `fix:` style and show iterative real-world development (including recent NFS CSI + media stack work).

If you are evaluating candidates for Platform, DevOps, SRE or Infrastructure roles and value hands-on multi-arch GitOps + Kubernetes experience, this repository is a direct demonstration of those skills.

Happy to walk through any part of the design.
