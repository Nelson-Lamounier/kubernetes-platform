# kubernetes-platform

Kubernetes platform services — Helm charts, ArgoCD Application manifests, and Grafana dashboards for the self-hosted K8s cluster.

## Overview

This repository owns the **platform layer** of the Kubernetes cluster:

| Component | Path | Description |
|-----------|------|-------------|
| **ArgoCD Applications** | `argocd-apps/` | 20 ArgoCD Application manifests (monitoring, traefik, cert-manager, etc.) |
| **Monitoring Stack** | `charts/monitoring/` | Custom Helm chart: Grafana, Prometheus, Loki, Tempo, Alloy, Promtail, node-exporter, kube-state-metrics |
| **Crossplane** | `charts/crossplane-*/` | Crossplane providers and XRD definitions |
| **ECR Token Refresh** | `charts/ecr-token-refresh/` | Automated ECR credential rotation |
| **Cert-Manager Config** | `charts/cert-manager-config/` | TLS certificate configuration |

## Architecture

ArgoCD reconciles this repository via the **App-of-Apps** pattern:

```
platform-root-app (bootstrap) → argocd-apps/*.yaml → charts/*
```

The `platform-root-app.yaml` (maintained in the bootstrap repository) points ArgoCD at `argocd-apps/` in this repository. Each Application manifest then references the corresponding chart directory.

## Development

### Prerequisites

- Helm 3.x
- Python 3.9+ (for `deploy.py` scripts)
- `ruff` (Python linting)

### Validation

```bash
# Validate all Helm charts
helm template monitoring charts/monitoring/chart/ \
  -f charts/monitoring/chart/values.yaml \
  -f charts/monitoring/chart/values-development.yaml

# Lint Python deploy scripts
ruff check deploy_helpers/ charts/monitoring/deploy.py

# Validate Grafana dashboard JSON
npx tsx --test tests/validate-dashboards.test.ts
```

## Deploy Helpers

The `deploy_helpers/` directory contains a vendored copy of the shared Python deployment library used by `deploy.py` scripts. This library provides:

- **BFF** (Backend-for-Frontend) pattern for SSM → K8s Secret deployment
- **K8s** client helpers for namespace and secret management
- **SSM** parameter retrieval with retry logic
- **S3** download utilities
- **Structured logging** and **step runner** framework

> **Note**: This is a vendored copy. The canonical source is in `cdk-monitoring/kubernetes-app/k8s-bootstrap/deploy_helpers/`.

## Repository Conventions

- **Language**: All user-facing text uses **English (UK)**
- **Linting**: Python files use `ruff`; Helm templates validated via `helm template`
- **CI/CD**: GitHub Actions — see `.github/workflows/`
