# pi-cluster

Kubernetes HomeLab running on Raspberry Pi, fully managed with GitOps.

A production-style cluster with declarative infrastructure, secrets management, observability, and zero-trust networking — all on constrained, real hardware.

---

## Overview

This repository contains the complete configuration for a personal Kubernetes cluster. 

Everything is defined as code and continuously reconciled by Flux CD. No manual `kubectl apply` after the initial bootstrap.

The design prioritizes the same practices used in real production environments:

- Declarative GitOps workflow
- Encrypted secrets in Git
- First-class observability
- Restricted security contexts
- External exposure without opening ports on the home network

Running the stack on Raspberry Pi hardware adds an extra constraint that forces careful resource awareness and solid fundamentals.

---

## Architecture

GitHub (this repo) 
↓ (Flux GitRepository + Kustomizations) Flux CD on the cluster 
├── apps/ → Linkding (bookmark manager) + Cloudflare Tunnel 
├── monitoring/ → kube-prometheus-stack (Prometheus, Grafana, Alertmanager) 
└── clusters/ → Flux system + environment-specific overlays (staging)


**Key design choices**
- **GitOps only** — no manual `kubectl apply` after bootstrap
- **Kustomize overlays** for base vs. environment (staging)
- **SOPS + age** for secret encryption (`.sops.yaml` + encrypted YAML)
- **Traefik** as IngressClass
- **Cloudflare Tunnel** (`cloudflared`) for zero-trust external access
- **HelmRelease** (via Flux) for the monitoring stack
- Persistent storage via PVC for application data

Public hostnames currently include:
- `lds.oskcloud.net` / `ldpi.oskcloud.net` → Linkding
- `grs.oskcloud.net` → Grafana

---

## Tech stack

| Category            | Tools / Components                          |
| ------------------- | ------------------------------------------- |
| Orchestration       | Kubernetes (k3s on Raspberry Pi)            |
| GitOps              | Flux CD v2 (Kustomization + HelmRelease)    |
| Manifest management | Kustomize                                   |
| Secrets             | SOPS + age                                  |
| Ingress             | Traefik                                     |
| External access     | Cloudflare Tunnel (cloudflared)             |
| Observability       | kube-prometheus-stack (Prometheus, Grafana) |
| Application         | Linkding                                    |
| Storage             | PersistentVolumeClaims                      |
| Source control      | GitHub                                      |

---

## Repository structure

apps/ 
├── base/linkding/ # Deployment, Service, PVC, Namespace 
└── staging/linkding/ # Ingress, Cloudflare Tunnel, SOPS secrets

clusters/ 
└── staging/ 
├── flux-system/ # Flux bootstrap & sync 
├── apps.yaml 
├── monitoring.yaml 
└── .sops.yaml

monitoring/ 
├── controllers/ # HelmRepository + HelmRelease for kube-prometheus-stack 
└── configs/ # Grafana TLS secret, kustomizations


All configuration is declarative. Changes land in Git → Flux reconciles automatically (typically every 1–10 minutes depending on the Kustomization).

---

## More on tech:

This repository will show:

- **GitOps & declarative infrastructure** (Flux CD, Kustomize, HelmRelease)
- **Secrets management** in Git (SOPS + age encryption)
- **Kubernetes operators & controllers** (Flux, Prometheus Operator)
- **Ingress & zero-trust networking** (Traefik + Cloudflare Tunnel)
- **Observability** (metrics, dashboards, TLS for Grafana)
- **Security hygiene** (non-root containers, restricted securityContext, encrypted secrets)
- **Environment separation** (base + staging overlays)
- **Resource-constrained environments** (Raspberry Pi hardware)
- **End-to-end ownership** — from cluster bootstrap to application + monitoring

These are the same patterns used in production Platform / SRE / DevOps roles.

---

## About the author

**Osman Sarper Küçük**  
DevOps / Platform Engineer focused on reliable, scalable infrastructure.

Previously Web3 Data Specialist (onchain analytics, data pipelines, BigQuery, TypeScript). Now applying that systems thinking to Kubernetes, automation, CI/CD, and observability.

- Personal site: [osk.cool](https://osk.cool)
- LinkedIn: [osmansarperkucuk](https://www.linkedin.com/in/osmansarperkucuk)
- GitHub: [Osman-SK](https://github.com/Osman-SK)
- X: [@OSK546](https://x.com/OSK546)

---

## Notes for reviewers

- This is an active personal lab, not a polished open-source product.
- The cluster runs continuously and is used daily.
- Secrets are encrypted; the public repo contains only ciphertext.
- Commits follow a clear `feat:` / `fix:` style and show iterative development of the stack.

If you are evaluating candidates for Platform, DevOps, SRE, or Infrastructure roles and value hands-on GitOps + Kubernetes experience, this repository is a direct demonstration of those skills.

Feel free to reach out — happy to walk through any part of the design.
