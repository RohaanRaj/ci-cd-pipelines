# ci-cd-pipelines

A collection of CI/CD pipeline implementations across different application stacks, demonstrating end-to-end DevOps workflows using Jenkins, Docker, Helm, and AWS.

---

## Projects

| Directory | Stack | Description |
|---|---|---|
| `voting-app/` | Python · Node.js · .NET · Redis · PostgreSQL | Multi-service voting app deployed on AWS EKS with Fargate |
| `react-express-mongodb/` | React · Express · MongoDB | Full-stack JS app with containerised CI pipeline |
| `react-nginx/` | React · Nginx | Static frontend served via Nginx with Docker build pipeline |
| `simple-python-app/` | Python | Minimal Python app demonstrating a basic Jenkins CI setup |

---

## Featured: AWS EKS CI/CD Pipeline — Voting App

The flagship project in this repo. A distributed, multi-service voting application deployed on **AWS EKS using Fargate** as serverless compute, with a fully automated CI/CD pipeline.

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

## Getting Started

### 1. Clone the repo

```bash
git clone https://github.com/RohaanRaj/ci-cd-pipelines.git
cd ci-cd-pipelines/voting-app
```

### 2. Deploy with Helm

```bash
# Install the application stack
helm install voting-app ./helm/voting-app \
  --namespace voting \
  --create-namespace \
  --set image.tag=<your-ecr-image-tag>
```

### 3. Apply ALB Ingress

```bash
kubectl apply -f k8s/ingress.yaml -n voting
```

### 4. Access the app

Get the ALB DNS from:
```bash
kubectl get ingress -n voting
```

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

## Author

**M Rohan Raj** — [github.com/RohaanRaj](https://github.com/RohaanRaj) · [linkedin.com/in/rohanraaj](https://linkedin.com/in/rohanraaj)

