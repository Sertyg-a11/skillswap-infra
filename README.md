# SkillSwap Infrastructure

GitOps infrastructure repository for SkillSwap application using ArgoCD.

## Architecture Overview

```
GitHub Actions CI/CD
├── skillswap-backend (Build → Test → Scan → Push → PR)
├── skillswap-frontend (Build → Test → Scan → Push → PR)
└── skillswap-infra (ArgoCD auto-sync)

Kubernetes Cluster (skillswap namespace)
├── Frontend (Nginx :80)
├── API Gateway (Spring :8081)
├── User Service (Spring :8082)
├── Message Service (Spring :8083)
├── PostgreSQL (:5432)
├── Redis (:6379)
├── RabbitMQ (:5672)
└── Monitoring (Prometheus + Grafana)
```

## Deployment Flow

1. **Push to main** → CI builds and tests code
2. **Security scan** → Trivy + OWASP dependency check
3. **Build & push images** → GHCR with SHA tags
4. **Auto PR to infra** → Updates image tags in values files
5. **Merge PR** → ArgoCD auto-syncs to cluster

## Repository Structure

```
skillswap-infra/
├── apps/skillswap/           # Helm charts
│   ├── api-gateway/
│   ├── user-service/
│   ├── message-service/
│   └── frontend/
├── bootstrap/                # ArgoCD apps
│   ├── argocd-apps.yaml     # Root application
│   └── apps/                # Child applications
├── environments/            # Environment values
│   ├── dev/
│   ├── stg/
│   └── prod/
└── local/                   # Local development
```

## Quick Start

### Prerequisites
- Kubernetes cluster (k3d, minikube, or cloud)
- kubectl configured
- ArgoCD installed

### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### Deploy SkillSwap

```bash
kubectl apply -f bootstrap/argocd-apps.yaml
```

### Access ArgoCD UI

```bash
kubectl port-forward svc/argocd-server -n argocd 8443:443
# Open https://localhost:8443 (admin / <password>)
```

## Services

| Service | Port | Description |
|---------|------|-------------|
| Frontend | 80 | React SPA |
| API Gateway | 8081 | Routes & auth |
| User Service | 8082 | User management |
| Message Service | 8083 | Real-time messaging |
| PostgreSQL | 5432 | Database |
| Redis | 6379 | Caching |
| RabbitMQ | 5672 | Message broker |

## CI/CD Pipeline Features

- Unit & integration tests with Testcontainers
- OWASP dependency vulnerability check
- Trivy container image scanning
- SARIF reports uploaded to GitHub Security
- Automated PR creation for GitOps
- Multi-stage Docker builds
- Image caching for faster builds

## Monitoring

Access Grafana at: `http://grafana.skillswap.localtest.me`

Metrics available:
- JVM metrics (heap, GC, threads)
- HTTP request metrics
- Custom business metrics

## Local Development

```bash
cd local
docker compose -f docker-compose.dev.yml up -d
```

## Troubleshooting

```bash
# Check ArgoCD status
kubectl get applications -n argocd

# View logs
kubectl logs -f deployment/skillswap-api-gateway -n skillswap

# Force sync
argocd app sync skillswap-api-gateway --force
```
