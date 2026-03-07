# DevOps-Arsenal

Dual CI/CD pipeline architecture comparing Jenkins and GitHub Actions with GitOps deployment to Kubernetes.

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Jenkins](https://img.shields.io/badge/jenkins-%232C5263.svg?style=for-the-badge&logo=jenkins&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white)
![ArgoCD](https://img.shields.io/badge/argocd-EF7B4D.svg?style=for-the-badge&logo=argo&logoColor=white)

## 🎯 Project Overview

A production-grade DevOps project demonstrating two complete CI/CD pipelines (Jenkins and GitHub Actions) deploying the same application to Kubernetes using GitOps principles. This comparative implementation showcases different automation approaches, security scanning, and automated deployment strategies.

### Key Features
- **Dual CI/CD Pipelines** - Jenkins and GitHub Actions running in parallel
- **GitOps Deployment** - Automated Kubernetes deployments via ArgoCD
- **Security Integration** - Trivy vulnerability scanning in both pipelines
- **Automated Testing** - Health checks and functional tests
- **Manifest Updates** - Automatic deployment YAML updates via Git
- **Separation of Concerns** - CI and CD in separate repositories
- **Production-Ready** - Multi-stage Docker builds, health checks, resource limits

## 🏗️ Architecture

```
                        Developer Push to GitHub
                                   ↓
                    ┌──────────────┴──────────────┐
                    ↓                             ↓
            JENKINS PIPELINE              GITHUB ACTIONS PIPELINE
                    ↓                             ↓
         ┌──────────────────┐           ┌──────────────────┐
         │ Build jenkins/   │           │ Build github/    │
         │ - Docker Build   │           │ - Docker Build   │
         │ - Run Tests      │           │ - Run Tests      │
         │ - Trivy Scan     │           │ - Trivy Scan     │
         │ - Push to Hub    │           │ - Push to Hub    │
         └──────────────────┘           └──────────────────┘
                    ↓                             ↓
         Image: jenkins-5              Image: github-12
                    ↓                             ↓
                    └──────────────┬──────────────┘
                                   ↓
                        DevOps-Arsenal-2 Repo
                         (Kubernetes Manifests)
                                   ↓
                    ┌──────────────┴──────────────┐
                    ↓                             ↓
         deployment.yaml (root)        k8s-githubactions/
         service.yaml                  deployment.yaml
         ingress.yaml                  service.yaml
                                      ingress.yaml
                    ↓                             ↓
                    └──────────────┬──────────────┘
                                   ↓
                            ArgoCD Watches
                                   ↓
                          Detects Git Change
                                   ↓
                      Syncs to Kubernetes Cluster
                                   ↓
                    ┌──────────────┴──────────────┐
                    ↓                             ↓
              /jenkins path                 /github path
           (Jenkins-built app)         (GitHub Actions-built app)
```

## 📂 Repository Structure

### DevOps-Arsenal (This Repo - CI)
```
DevOps-Arsenal/
├── .github/
│   └── workflows/
│       └── ci.yaml              # GitHub Actions pipeline
│
├── jenkins/
│   ├── Dockerfile               # Production-ready multi-stage build
│   ├── index.html               # Jenkins-branded application
│   └── nginx.conf               # Optimized NGINX configuration
│
├── github/
│   ├── Dockerfile               # Production-ready multi-stage build
│   ├── index.html               # GitHub Actions-branded application
│   └── nginx.conf               # Optimized NGINX configuration
│
├── Jenkinsfile                  # Jenkins pipeline definition
├── .dockerignore
├── .gitignore
└── README.md
```

### DevOps-Arsenal-2 (CD Repository)
```
DevOps-Arsenal-2/
├── deployment.yaml              # Jenkins app deployment (root level)
├── service.yaml                 # Jenkins ClusterIP service
├── ingress.yaml                 # Jenkins ingress (/jenkins path)
│
└── k8s-githubactions/
    ├── deployment.yaml          # GitHub Actions app deployment
    ├── service.yaml             # GitHub Actions ClusterIP service
    └── ingress.yaml             # GitHub Actions ingress (/github path)
```

## 🚀 Live Demo

### Application Endpoints
- **Jenkins Pipeline App**: http://INGRESS_IP/jenkins
- **GitHub Actions App**: http://INGRESS_IP/github

### ArgoCD Dashboard
- **URL**: http://ARGOCD_IP
- **Applications**: jenkins-app, github-app

## 🛠️ Technologies & Tools

| Technology | Purpose | Implementation |
|------------|---------|----------------|
| **Jenkins** | CI/CD Automation | Groovy pipeline with webhook triggers |
| **GitHub Actions** | CI/CD Automation | YAML workflows with path-based triggers |
| **Docker** | Containerization | Multi-stage builds with NGINX Alpine |
| **Trivy** | Security Scanning | Vulnerability detection in both pipelines |
| **ArgoCD** | GitOps CD | Automated K8s deployments from Git |
| **Kubernetes (AKS)** | Container Orchestration | Azure Kubernetes Service |
| **NGINX** | Web Server | Production-optimized configuration |
| **Git** | Version Control | Dual-repo GitOps pattern |

## 📊 Pipeline Comparison

| Feature | Jenkins Pipeline | GitHub Actions Pipeline |
|---------|------------------|-------------------------|
| **Trigger** | Webhook on push | GitHub Events (push to github/) |
| **Language** | Groovy | YAML |
| **Build Context** | jenkins/ folder | github/ folder |
| **Image Tag** | jenkins-{BUILD_NUMBER} | github-{RUN_NUMBER} |
| **Testing** | Bash scripts | Bash scripts |
| **Security Scan** | Trivy CLI | Trivy GitHub Action |
| **Manifest Update** | Git clone + sed + push | Git clone + sed + push |
| **Manifest Path** | deployment.yaml (root) | k8s-githubactions/deployment.yaml |
| **Artifacts** | trivy-report.json | trivy-report.json (uploaded) |
| **Deployment Target** | /jenkins path | /github path |

## 🔄 CI/CD Workflow

### Jenkins Pipeline Flow
```
1. Webhook Trigger (any push to main)
   ↓
2. Checkout code from Git
   ↓
3. Build Docker image from jenkins/
   ↓
4. Run tests (health check + functionality)
   ↓
5. Security scan with Trivy
   ↓
6. Push to Docker Hub (jenkins-N, jenkins-latest)
   ↓
7. Clone DevOps-Arsenal-2 repository
   ↓
8. Update deployment.yaml (root level) with new tag
   ↓
9. Commit and push to Arsenal-2
   ↓
10. ArgoCD detects change and deploys
```

### GitHub Actions Pipeline Flow
```
1. Push to main (changes in github/ folder)
   ↓
2. Checkout code
   ↓
3. Generate image tag (github-N)
   ↓
4. Build Docker image from github/
   ↓
5. Run tests (health check + functionality)
   ↓
6. Security scan with Trivy Action
   ↓
7. Push to Docker Hub (github-N, github-latest)
   ↓
8. Clone DevOps-Arsenal-2 repository
   ↓
9. Update k8s-githubactions/deployment.yaml
   ↓
10. Commit and push to Arsenal-2
   ↓
11. ArgoCD detects change and deploys
   ↓
12. Upload security report as artifact
```


## 🚦 Quick Start

### Prerequisites
- Azure account with AKS cluster
- Docker Hub account
- GitHub account
- Jenkins server (can be deployed on K8s)
- kubectl configured
- ArgoCD installed on cluster

### Setup Instructions

#### 1. Clone Repositories
```bash
# Clone CI repository
git clone https://github.com/YOUR_USERNAME/DevOps-Arsenal.git
cd DevOps-Arsenal

# Clone CD repository (separate directory)
git clone https://github.com/YOUR_USERNAME/DevOps-Arsenal-2.git
```

#### 2. Configure Jenkins
```bash
# Install required Jenkins plugins:
# - Docker Pipeline
# - Git
# - Credentials Binding

# Add credentials in Jenkins:
# - Docker Hub credentials (ID: dockerhub-credentials)
# - GitHub token (ID: git)

# Create new Pipeline job pointing to Jenkinsfile
# Configure webhook in GitHub repo settings
```

#### 3. Configure GitHub Actions
```bash
# Add secrets to DevOps-Arsenal repository:
# Settings → Secrets and Variables → Actions

# Required secrets:
# - DOCKER_USERNAME: Your Docker Hub username
# - DOCKER_PASSWORD: Your Docker Hub password
# - GIT_TOKEN: GitHub Personal Access Token with repo scope
```

#### 4. Update Configuration
```bash
# Update Docker image names in:
# - Jenkinsfile (DOCKER_IMAGE variable)
# - .github/workflows/ci.yaml (DOCKER_IMAGE variable)
# - DevOps-Arsenal-2/deployment.yaml (root level)
# - DevOps-Arsenal-2/k8s-githubactions/deployment.yaml

# Update repository URLs:
# - Jenkinsfile (GIT_REPO variable → DevOps-Arsenal-2)
# - .github/workflows/ci.yaml (MANIFEST_REPO variable → DevOps-Arsenal-2)
```

#### 5. Deploy to Kubernetes
```bash
# Create ArgoCD applications for both deployments

# Jenkins App via ArgoCD UI:
# - Name: jenkins-app
# - Repo: https://github.com/YOUR_USERNAME/DevOps-Arsenal-2
# - Path: . (root directory)
# - Namespace: default
# - Sync: Automatic

# GitHub Actions App via ArgoCD UI:
# - Name: github-app
# - Repo: https://github.com/YOUR_USERNAME/DevOps-Arsenal-2
# - Path: k8s-githubactions
# - Namespace: default
# - Sync: Automatic
```

#### 6. Test Pipelines
```bash
# Test Jenkins pipeline
cd DevOps-Arsenal
echo "<!-- test -->" >> jenkins/index.html
git add jenkins/
git commit -m "Test Jenkins pipeline"
git push origin main

# Test GitHub Actions pipeline
echo "<!-- test -->" >> github/index.html
git add github/
git commit -m "Test GitHub Actions pipeline"
git push origin main

# Watch pipelines run:
# - Jenkins: http://your-jenkins-url
# - GitHub Actions: https://github.com/YOUR_USERNAME/DevOps-Arsenal/actions
```

## 📈 Docker Images

### Build Locally
```bash
# Build Jenkins image
cd jenkins/
docker build -t YOUR_USERNAME/jenkins-image:jenkins-1 .
docker push YOUR_USERNAME/jenkins-image:jenkins-1

# Build GitHub Actions image
cd ../github/
docker build -t YOUR_USERNAME/github-image:github-1 .
docker push YOUR_USERNAME/github-image:github-1
```

### Image Features
- **Base Image**: nginx:alpine (lightweight, ~25MB)
- **Security**: Runs as non-root user (nginx)
- **Health Checks**: Built-in /health endpoint
- **Optimization**: Gzip compression enabled
- **Headers**: Security headers (X-Frame-Options, X-Content-Type-Options, X-XSS-Protection)

## 🔒 Security Features

### Pipeline Security
- ✅ Trivy vulnerability scanning (HIGH/CRITICAL severity)
- ✅ Security scan reports generated and stored
- ✅ Non-blocking scans to maintain deployment velocity
- ✅ Credentials managed via Jenkins/GitHub secrets

### Container Security
- ✅ Non-root user execution (UID 101)
- ✅ Read-only root filesystem where possible
- ✅ No privilege escalation
- ✅ Minimal attack surface (Alpine base)

### Kubernetes Security
- ✅ Security contexts defined
- ✅ Resource limits enforced
- ✅ Network policies ready
- ✅ RBAC configured via ArgoCD

