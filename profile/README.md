# Flavoriy

**A production-style DevOps platform demonstrating end-to-end software delivery — CI/CD, GitOps, progressive delivery, observability, and Infrastructure as Code on AWS.**

Flavoriy powers the deployment of **TikTo**, a task and calendar planning application built with Next.js, TypeScript, and five backend microservices. The project is a hands-on exploration of how modern engineering teams ship software safely and reliably at scale.

---

## 🏗️ Architecture at a Glance

```
Developer → Pull Request → CI (test, scan, build) → Container Registry
                                     ↓
                          GitOps Repo (image tag update)
                                     ↓
                              Argo CD (sync)
                          ↓                    ↓
                  K3s (Development)     Amazon EKS (Production)
                                              ↓
                                     Argo Rollouts (Canary)
                                              ↓
                                  OpenSearch (error-rate analysis)
```

Infrastructure is provisioned independently via Terraform, and Kubernetes state is version-controlled and reconciled continuously by Argo CD — Git is always the single source of truth.

---

## 📦 Repositories

| Repository | Purpose | Key Components |
|---|---|---|
| [**tikto**](https://github.com/flavoriy/tikto) | Application & CI | Next.js frontend, 5 microservices, tests, SonarCloud, Docker builds, Trivy scanning, GHCR publishing |
| [**gitops-manifest**](https://github.com/flavoriy/gitops-manifest) | Kubernetes desired state | Kustomize overlays, Argo CD Applications, Argo Rollouts, image promotion |
| [**Infrastructure-as-Code**](https://github.com/flavoriy/Infrastructure-as-Code) | AWS infrastructure | Terraform modules for VPC, EKS, EC2, OpenSearch, Secrets Manager |

---

## 🚀 What This Project Demonstrates

- **Continuous Integration** — Automated linting, type checking, unit testing, and SonarCloud quality gates on every pull request, with path-based change detection so only affected services are built.
- **Container Security** — Every image is scanned with Trivy; builds are blocked on unresolved High/Critical vulnerabilities before promotion.
- **GitOps Delivery** — Application pipelines never deploy directly to Kubernetes. All changes flow through Git, with Argo CD continuously reconciling cluster state.
- **Progressive Delivery** — Production releases use Argo Rollouts Canary deployments (10% → 50% → 80% → 100%), with automated smoke tests, health checks, and error-rate analysis via Amazon OpenSearch — auto-rolling back if thresholds are exceeded.
- **Infrastructure as Code** — A fully modular, Multi-AZ AWS environment provisioned with Terraform, decoupled from application delivery.
- **Environment Isolation** — Lightweight K3s for fast development iteration; Amazon EKS with Spot node groups for production.

---

## 🛠️ Tech Stack

| Category | Technologies |
|---|---|
| Application | Next.js, TypeScript, Node.js |
| CI | GitHub Actions, Vitest, ESLint, SonarCloud |
| Containers & Security | Docker, GitHub Container Registry, Trivy |
| Kubernetes | Amazon EKS, K3s, Kustomize |
| GitOps & Delivery | Argo CD, Argo Rollouts |
| Networking | Istio, AWS ALB, Tailscale |
| Infrastructure as Code | Terraform |
| AWS | VPC, EKS, EC2, OpenSearch, Secrets Manager |
| Observability | Amazon OpenSearch, health checks, smoke tests |

---

## 📐 Core Principles

- Git is the single source of truth for cluster state
- Application pipelines never deploy directly to Kubernetes
- Production only ever reuses artifacts already validated in development
- Security and quality gates run *before* promotion, not after
- Infrastructure and application delivery are independently managed
- Every production release supports gradual rollout and automatic rollback

---

📖 Full implementation details, commands, and troubleshooting guides live in each repository above.
