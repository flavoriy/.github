# Production-Like CI/CD Platform

A DevOps lab focused on building a production-style CI/CD workflow with Jenkins, reusable pipeline code, GitOps, Kubernetes, and Terraform.

![Platform overview](./assets/platform-overview.png)

## Overview

This project demonstrates how an application can move from pull request to Kubernetes deployment through a controlled delivery process:

```text
Pull Request -> Jenkins CI -> Tests -> SonarCloud Quality Gate -> Security Scan
Merge        -> Build Image -> Push to GHCR -> Update GitOps -> ArgoCD Sync
```

The main focus is the **Jenkins Shared Library**. Application repositories keep their Jenkinsfiles small, while common CI/CD logic is maintained in one reusable library.

## Core Repositories

| Repository | Purpose |
|---|---|
| `Jenkins-share-lib` | Reusable Jenkins steps for build, test, scan, Docker, GitOps, and ArgoCD verification |
| `App 1 / TikTo` | Example application using the shared pipeline workflow |
| `gitops-manifest` | Kubernetes manifests for dev and prod-style deployments |
| `IaC` | Terraform and EC2 setup documentation for Jenkins and k3s infrastructure |

## CI/CD Scope

- Jenkins Multibranch Pipeline for pull requests and protected branches
- SonarCloud quality gate before merge
- Docker image build and push to GHCR
- Trivy vulnerability scan before release
- GitOps-based deployment with ArgoCD
- k3s environments for dev and production-style deployment

## Jenkins Shared Library

The shared library provides pipeline steps that can be reused across application repositories:

```groovy
buildApp()
testApp()
sonarScan()
dockerBuild()
trivyScan()
dockerPush()
updateGitopsManifest()
verifyArgoApp()
```

This keeps CI/CD behavior consistent and easier to maintain as more applications are added.

## Diagrams To Add

Place images under:

```text
.github/profile/assets/
```

Recommended images:

| File | Content |
|---|---|
| `platform-overview.png` | Overall CI/CD platform flow |
| `architecture.png` | Jenkins, GHCR, GitOps, ArgoCD, k3s, and AWS layout |
| `jenkins-shared-library.png` | Shared library structure and reusable steps |
| `gitops-flow.png` | Manifest update and ArgoCD sync flow |

