# 🏗️ SkillSwap Infrastructure

This repository manages the **deployment and infrastructure** of the SkillSwap platform using **Kubernetes**, **ArgoCD**, and **GitOps** principles.
## ⚙️ Deployment Flow

1. **App repo push → CI builds → pushes to GHCR**
2. **CI creates PR → updates image tag in this repo**
3. **ArgoCD auto-syncs → deploys new version**
