# ci-cd-pipelines

This repo is my collection of Pipelines that I have done starting from the basic ones like a simple react-nginx app all the way to a CI pipline with Jenkins that pushes images to AWS ECR. 

---

## Projects

| Directory | Stack |
|---|---|
| `voting-app/` | Python · Node.js · .NET · Redis · PostgreSQL | 
| `react-express-mongodb/` | React · Express · MongoDB |
| `react-nginx/` | React · Nginx |
| `simple-python-app/` | Python |

---

## Featured: AWS EKS CI/CD Pipeline — Voting App

Worked extensively on this project. A distributed, multi-service voting application deployed on **AWS EKS using Fargate** as serverless compute, with a fully automated CI/CD pipeline.

### Architecture

```
                        ┌─────────────────────────────────────────────┐
                        │               AWS EKS (Fargate)              │
                        │                                              │
  User ──► ALB Ingress ─┤─► Vote UI (Python) ──► Redis               │
           Controller   │                           │                  │
                        │                        Worker (.NET) ──► PostgreSQL
                        │                                              │
                        │                    Result UI (Node.js) ◄────┘
                        └─────────────────────────────────────────────┘
```

### Services

- **Vote** — Python frontend to cast votes
- **Redis** — In-memory queue for incoming votes
- **Worker** — .NET service that consumes votes from Redis and writes to PostgreSQL
- **PostgreSQL** — Persistent vote storage
- **Result** — Node.js frontend to display live results

### CI Pipeline (Jenkins)

```
Code Push → GitHub
     │
     ▼
Jenkins Pipeline
     ├── Build Docker images per service
     ├── Push images to AWS ECR (private registry)
     └── Auto-update Helm values in GitHub (image tags)
                              │
                              ▼
                     (ArgoCD / manual Helm upgrade picks up changes)
```

### Helm

- Custom Helm charts authored for the entire application stack
- Parameterised deployments with versioned releases
- One-command rollbacks via Helm history
- Separate Helm packages repo maintained for controlled upgrade workflows

### AWS Infrastructure

| Component | Service |
|---|---|
| Kubernetes cluster | EKS |
| Compute | Fargate (serverless) |
| Container registry | ECR |
| Load balancer | ALB via AWS Load Balancer Controller |
| Storage | S3 (artifacts) |
| Monitoring | CloudWatch + SNS alerts |

---

## Prerequisites

- AWS CLI configured with appropriate IAM permissions
- `kubectl` pointed at your EKS cluster
- `helm` v3+
- Jenkins instance with AWS credentials and GitHub webhook configured
- Docker

---

## CI/CD Flow (How it Works)

1. Developer pushes code to GitHub
2. Jenkins webhook triggers the pipeline
3. Jenkins builds Docker image and pushes to **AWS ECR**
4. Jenkins updates the `image.tag` value in the Helm chart values file and commits back to GitHub
5. Helm upgrade is run (manually or via ArgoCD sync) — new pods roll out on EKS Fargate

---

## Rollback

```bash
# View release history
helm history voting-app -n voting

# Rollback to previous revision
helm rollback voting-app <revision> -n voting
```

---

## Other Pipelines

### `react-express-mongodb/`
Full-stack JavaScript application (React + Express + MongoDB) with a Jenkins pipeline that builds and pushes Docker images for both frontend and backend services.

### `react-nginx/`
React SPA served through an Nginx container. The Jenkins pipeline handles multi-stage Docker builds — Node.js build stage followed by a minimal Nginx production image.

### `simple-python-app/`
A minimal Python Flask application demonstrating the basics of a Jenkins CI pipeline — linting, testing, Docker build, and registry push.

---

## Tech Stack

![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=flat&logo=jenkins&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=flat&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=flat&logo=helm&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat&logo=docker&logoColor=white)
![Go](https://img.shields.io/badge/Go-00ADD8?style=flat&logo=go&logoColor=white)

---
