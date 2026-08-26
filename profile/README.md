# Flavoriy – End-to-End DevOps Platform

Flavoriy is a hands-on DevOps project focused on building a complete software delivery platform with CI/CD, Kubernetes, GitOps, progressive delivery, observability, and Infrastructure as Code on AWS.

The platform deploys TikTo, a task and calendar planning application built with Next.js, TypeScript, and five backend microservices.

## Application Overview

TikTo is organized as a monorepo containing:

- One Next.js web frontend operating as a Backend for Frontend (BFF)
- Five internal backend microservices
- Shared libraries and development tooling
- Automated testing, security scanning, containerization, and delivery workflows

The platform separates application code, Kubernetes desired state, and cloud infrastructure into three repositories.

## Repository Structure

Repository                                                                     Responsibility              Main components
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
tikto (https://github.com/flavoriy/tikto)                                      Application and CI          Next.js frontend, five microservices, tests, SonarCloud, Docker builds, Trivy scanning, and GHCR publishing
─────────────────────────────────────────────────────────────────────────────  ──────────────────────────  ─────────────────────────────────────────────────────────────────────────────────────────────────────────────
gitops-manifest (https://github.com/flavoriy/gitops-manifest)                  Kubernetes desired state    Kustomize overlays, Argo CD Applications, Argo Rollouts, environment configuration, and image promotion
─────────────────────────────────────────────────────────────────────────────  ──────────────────────────  ─────────────────────────────────────────────────────────────────────────────────────────────────────────────
Infrastructure-as-Code (https://github.com/flavoriy/Infrastructure-as-Code)    AWS infrastructure          Terraform modules for VPC, EKS, EC2, OpenSearch, Secrets Manager, and supporting infrastructure

## System Architecture

flowchart LR
  Developer[Developer] -->|Pull request or merge| AppRepo[tikto repository]

  AppRepo --> CI[GitHub Actions]
  CI --> Quality[Tests and SonarCloud]
  Quality --> Build[Docker build]
  Build --> Security[Trivy scan]
  Security --> Registry[GitHub Container Registry]

  CI -->|Update image tag| GitOps[gitops-manifest repository]
  GitOps --> ArgoCD[Argo CD]

  ArgoCD --> Dev[K3s Development]
  ArgoCD --> Rollouts[Argo Rollouts]
  Rollouts --> Prod[Amazon EKS Production]

  Terraform[Terraform IaC] --> AWS[AWS Infrastructure]
  AWS --> Dev
  AWS --> Prod
  AWS --> OpenSearch[Amazon OpenSearch]

  OpenSearch -->|Error-rate analysis| Rollouts

## End-to-End Delivery Flow

### 1. Continuous Integration

When a developer opens or updates a pull request, GitHub Actions runs:

- ESLint validation
- TypeScript type checking
- Vitest unit tests and code coverage
- SonarCloud static analysis and Quality Gate validation

Path-based change detection ensures that workflows test and build only the affected services. Independent service jobs run in parallel, while dependency caching and shared base images reduce unnecessary build time.

### 2. Container Build and Security

After changes are merged or a release tag is created, the delivery workflow:

1. Builds a Docker image for each affected service
2. Tags the image with an immutable version
3. Scans the image with Trivy
4. Blocks promotion when unresolved High or Critical vulnerabilities are detected
5. Publishes validated images to GitHub Container Registry

Validated development images are promoted to production through image retagging instead of rebuilding, ensuring that production uses the same artifact tested in development.

### 3. GitOps Promotion

After an image is published, the application pipeline updates the corresponding image tag in patch-image.yaml within the gitops-manifest repository.

This commit acts as the handoff between the CI pipeline and the Kubernetes desired state. Application pipelines do not deploy directly to Kubernetes clusters.

### 4. Continuous Deployment

Argo CD monitors the gitops-manifest repository and synchronizes the declared configuration with the target Kubernetes cluster.

- Development uses standard Kubernetes Deployments on K3s
- Production uses Argo Rollouts on Amazon EKS
- Kustomize manages environment-specific configuration
- Istio and AWS ALB route production traffic

### 5. Canary Delivery and Automated Rollback

Production releases use a Canary strategy that gradually shifts traffic through the following stages:

10% → 50% → 80% → 100%

Each stage performs automated validation using:

- Application smoke tests
- Runtime health checks
- Error-rate analysis from Amazon OpenSearch

If the configured error threshold is exceeded, Argo Rollouts automatically stops the promotion and rolls the application back to the previous stable version.

### 6. Infrastructure as Code

The AWS environment is provisioned independently from application delivery using reusable Terraform modules.

Provisioned resources include:

- Multi-AZ Amazon VPC
- Amazon EKS production cluster with Spot node groups
- EC2 instance running the private K3s development environment
- EC2 instance hosting Argo CD
- Amazon OpenSearch
- AWS Secrets Manager
- Application Load Balancer integration
- Remote Terraform state management

This separation allows infrastructure, application code, and Kubernetes configuration to evolve through independent workflows.

## Development and Production Environments

Area                   Development                                 Production
━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Compute                Single-node K3s on EC2                      Amazon EKS with Spot node groups
─────────────────────  ──────────────────────────────────────────  ──────────────────────────────────────────────────────
Network access         Private access through Tailscale VPN        AWS ALB and Istio Ingress Gateway
─────────────────────  ──────────────────────────────────────────  ──────────────────────────────────────────────────────
Deployment strategy    Standard Kubernetes Deployment              Argo Rollouts Canary deployment
─────────────────────  ──────────────────────────────────────────  ──────────────────────────────────────────────────────
Configuration          Development Kustomize overlay               Production Kustomize overlay
─────────────────────  ──────────────────────────────────────────  ──────────────────────────────────────────────────────
Validation             Manifest and integration checks             Smoke tests and error-rate analysis
─────────────────────  ──────────────────────────────────────────  ──────────────────────────────────────────────────────
Primary purpose        Fast iteration and deployment validation    Controlled production traffic and automated rollback

## Technology Stack

Area                          Technologies
━━━━━━━━━━━━━━━━━━━━━━━━━━━━  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Application                   Next.js, TypeScript, Node.js
────────────────────────────  ─────────────────────────────────────────────────
Continuous Integration        GitHub Actions, Vitest, ESLint, SonarCloud
────────────────────────────  ─────────────────────────────────────────────────
Containers and Security       Docker, GitHub Container Registry, Trivy
────────────────────────────  ─────────────────────────────────────────────────
Kubernetes                    Amazon EKS, K3s, Kustomize
────────────────────────────  ─────────────────────────────────────────────────
GitOps and Delivery           Argo CD, Argo Rollouts
────────────────────────────  ─────────────────────────────────────────────────
Networking                    Istio, AWS Application Load Balancer, Tailscale
────────────────────────────  ─────────────────────────────────────────────────
Infrastructure as Code        Terraform
────────────────────────────  ─────────────────────────────────────────────────
AWS                           VPC, EKS, EC2, OpenSearch, Secrets Manager
────────────────────────────  ─────────────────────────────────────────────────
Observability and Analysis    Amazon OpenSearch, health checks, smoke tests

## Design Principles

The platform follows several core principles:

- Application pipelines never deploy directly to Kubernetes
- Git is the source of truth for the desired cluster state
- Development and production use isolated environments
- Production reuses artifacts already validated in development
- Security and quality checks run before image promotion
- Infrastructure and application delivery remain independently managed
- Production releases support gradual traffic shifting and automated rollback

## Documentation

Each repository contains implementation details, operational commands, deployment instructions, and troubleshooting guidance:

- tikto (https://github.com/flavoriy/tikto) – application development and CI workflows
- gitops-manifest (https://github.com/flavoriy/gitops-manifest) – Kubernetes, GitOps, and progressive delivery
- Infrastructure-as-Code (https://github.com/flavoriy/Infrastructure-as-Code) – AWS infrastructure and Terraform
