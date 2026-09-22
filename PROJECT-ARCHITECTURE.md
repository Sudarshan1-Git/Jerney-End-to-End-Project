# Jerney — End-to-End DevOps Project

## 1. Project Overview

Jerney is a full-stack web application consisting of:

* React frontend
* Node.js / Express backend
* PostgreSQL database

The application was taken from source code and converted into an end-to-end DevOps deployment using:

* GitHub
* GitHub Actions
* Docker
* SonarQube
* Trivy
* Docker Hub
* Kubernetes
* Terraform
* AWS VPC
* Amazon EKS
* Argo CD
* AWS Load Balancer Controller
* AWS Application Load Balancer

The goal of the project was to implement a realistic DevOps/DevSecOps workflow from source code commit to application deployment on AWS EKS.

---

# 2. High-Level Architecture

```text
                         DEVELOPER
                             |
                             | git push
                             v
                    +-------------------+
                    |      GitHub       |
                    | Source Repository |
                    +---------+---------+
                              |
                              v
                    +-------------------+
                    |  GitHub Actions   |
                    |       CI          |
                    +---------+---------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          Lint           SonarQube           Docker
                              |                |
                              |                v
                              |              Trivy
                              |                |
                              |                v
                              |          Docker Hub
                              |                |
                              +----------------+
                                       |
                                       v
                              Kubernetes Manifests
                                       |
                                       v
                                +-------------+
                                |   Argo CD   |
                                |   GitOps     |
                                +------+------+
                                       |
                                       v
                              +------------------+
                              |    AWS EKS       |
                              |   Kubernetes     |
                              +--------+---------+
                                       |
                              +--------+---------+
                              |                  |
                              v                  v
                         Frontend Service    Backend Service
                              |                  |
                              |                  v
                              |             PostgreSQL
                              |
                              v
                       AWS Application
                       Load Balancer
                              |
                              v
                            USER
```

---

# 3. Application Architecture

The application follows a three-tier architecture.

```text
                 USER
                  |
                  v
          React Frontend
                  |
                  | HTTP / API
                  v
          Node.js Backend
                  |
                  | PostgreSQL
                  v
             PostgreSQL
```

### Frontend

Technology:

* React
* Vite
* Nginx

The frontend is compiled into static files using Vite.

The production Docker image uses Nginx to serve the generated frontend files.

### Backend

Technology:

* Node.js
* Express
* PostgreSQL driver (`pg`)

The backend exposes REST APIs.

The backend listens on:

```text
Port 5000
```

The backend connects to PostgreSQL using environment variables.

### Database

Technology:

```text
PostgreSQL
```

Database configuration is provided through Kubernetes environment variables and secrets.

The backend automatically initializes the required tables when it starts.

---

# 4. Source Code Repository

GitHub repository:

```text
https://github.com/Sudarshan1-Git/Jerney-End-to-End-Project.git
```

The repository contains:

```text
Jerney-End-to-End-Project/
│
├── .github/
│   └── workflows/
│       └── ci.yml
│
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   └── src/
│
├── frontend/
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── package.json
│   ├── package-lock.json
│   └── src/
│
├── k8s/
│   ├── namespace.yaml
│   ├── backend-configmap.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── postgres-deployment.yaml
│   ├── postgres-service.yaml
│   └── ingress.yaml
│
├── terraform/
│   ├── provider.tf
│   ├── vpc.tf
│   ├── subnet.tf
│   ├── internet_gateway.tf
│   ├── nat.tf
│   ├── route_table.tf
│   ├── security_group.tf
│   ├── iam.tf
│   └── eks.tf
│
├── docs/
│   └── PROJECT-ARCHITECTURE.md
│
├── docker-compose.yml
├── .gitignore
└── README.md
```

---

# 5. GitHub Actions CI Pipeline

GitHub Actions is used for continuous integration.

The pipeline performs:

```text
Git Push
   |
   v
Checkout Code
   |
   v
Setup Node.js
   |
   v
Install Backend Dependencies
   |
   v
Backend Lint
   |
   v
Install Frontend Dependencies
   |
   v
Frontend Lint
   |
   v
SonarQube Scan
   |
   v
Build Backend Docker Image
   |
   v
Trivy Backend Scan
   |
   v
Build Frontend Docker Image
   |
   v
Trivy Frontend Scan
   |
   v
Docker Hub Login
   |
   v
Push Backend Image
   |
   v
Push Frontend Image
```

The workflow runs automatically when code is pushed to the `main` branch or when a pull request targets `main`.

---

# 6. SonarQube

SonarQube is used for static code analysis.

The pipeline scans:

```text
backend/src
frontend/src
```

The project is configured using:

```text
sonar-project.properties
```

SonarQube helps identify:

* Bugs
* Code smells
* Vulnerabilities
* Maintainability issues
* Code quality problems

SonarQube was deployed separately on an AWS EC2 instance.

---

# 7. Trivy Security Scanning

Trivy is integrated into GitHub Actions.

Both Docker images are scanned:

```text
Backend Image
      |
      v
    Trivy
      |
      v
Critical vulnerabilities check
```

and:

```text
Frontend Image
      |
      v
    Trivy
      |
      v
Critical vulnerabilities check
```

The pipeline is configured to fail when relevant unfixed CRITICAL vulnerabilities are detected.

---

# 8. Docker Architecture

Two production Docker images were created.

## Backend Docker Image

```text
node:22-alpine
        |
        v
Install updated npm
        |
        v
Copy package files
        |
        v
npm ci
        |
        v
Copy backend source
        |
        v
Expose port 5000
        |
        v
npm start
```

## Frontend Docker Image

The frontend uses a multi-stage Docker build.

```text
Node.js Build Stage
        |
        v
npm ci
        |
        v
npm run build
        |
        v
dist/
        |
        v
Nginx Production Stage
        |
        v
Serve static frontend
```

This keeps the final frontend image focused on serving the built application.

---

# 9. Docker Hub

After successful security scanning, GitHub Actions pushes the images to Docker Hub.

```text
GitHub Actions
      |
      +--------------------+
      |                    |
      v                    v
Backend Image         Frontend Image
      |                    |
      +---------+----------+
                |
                v
            Docker Hub
```

Kubernetes pulls the images from Docker Hub during deployment.

---

# 10. Local Docker Compose

Before deploying to Kubernetes, the application was tested locally using Docker Compose.

The local architecture was:

```text
Frontend
   |
   v
Backend
   |
   v
PostgreSQL
```

Docker Compose provided the local development environment for validating that:

* Frontend starts correctly
* Backend starts correctly
* Backend can communicate with PostgreSQL
* API endpoints work
* Database tables are initialized

---

# 11. Terraform Architecture

Terraform was used to provision AWS infrastructure.

Terraform manages the infrastructure rather than creating AWS resources manually.

The infrastructure includes:

```text
AWS
 |
 +-- VPC
 |
 +-- Public Subnets
 |
 +-- Private Subnets
 |
 +-- Internet Gateway
 |
 +-- NAT Gateway
 |
 +-- Route Tables
 |
 +-- Security Groups
 |
 +-- IAM Roles / Policies
 |
 +-- EKS Cluster
 |
 +-- EKS Node Group
```

Terraform state files and other generated Terraform files are excluded from Git.

---

# 12. AWS VPC Architecture

The project uses a custom VPC.

```text
                    AWS VPC
                 10.0.0.0/16
                       |
        +--------------+--------------+
        |                             |
        v                             v
   Public Subnets              Private Subnets
        |                             |
   +----+----+                   +----+----+
   |         |                   |         |
   v         v                   v         v
Public-1  Public-2            Private-1 Private-2
10.0.1.0  10.0.2.0            10.0.11.0 10.0.12.0
```

Two Availability Zones are used.

Public subnets are used for internet-facing AWS resources.

Private subnets are used for EKS worker nodes.

---

# 13. Internet Gateway

The Internet Gateway provides internet connectivity for resources in public subnets.

```text
Internet
   |
   v
Internet Gateway
   |
   v
Public Subnets
```

The internet-facing Application Load Balancer uses the public subnets.

---

# 14. NAT Gateway

A single NAT Gateway was configured to reduce infrastructure cost.

Private subnet resources can access the internet through:

```text
Private Subnet
      |
      v
 NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

This allows private EKS nodes to access required external services without making the nodes publicly accessible.

---

# 15. Amazon EKS

The application is deployed to Amazon EKS.

Cluster:

```text
Jerney-EKS
```

Kubernetes version used:

```text
1.36
```

The EKS worker nodes run inside private subnets.

```text
                Amazon EKS
                    |
          +---------+---------+
          |                   |
          v                   v
      Worker Node 1       Worker Node 2
      Private Subnet      Private Subnet
```

Both nodes were verified as `Ready`.

---

# 16. Kubernetes Architecture

The application runs inside the Kubernetes namespace:

```text
jerney
```

Resources include:

```text
Namespace
   |
   +-- Backend Deployment
   |
   +-- Backend Service
   |
   +-- Frontend Deployment
   |
   +-- Frontend Service
   |
   +-- PostgreSQL Deployment
   |
   +-- PostgreSQL Service
   |
   +-- ConfigMap
   |
   +-- Secret
   |
   +-- Ingress
```

---

# 17. Kubernetes Application Flow

The request flow is:

```text
Internet
   |
   v
AWS Application Load Balancer
   |
   v
Kubernetes Ingress
   |
   v
Frontend Service
   |
   v
Frontend Pod / Nginx
   |
   | /api
   v
Backend Service
   |
   v
Backend Pod
   |
   v
PostgreSQL Service
   |
   v
PostgreSQL Pod
```

---

# 18. Kubernetes Deployments

## Frontend

The frontend is deployed as a Kubernetes Deployment.

The frontend container serves the production React build using Nginx.

Service:

```text
frontend
Port: 80
```

## Backend

The backend is deployed as a Kubernetes Deployment.

Service:

```text
backend
Port: 5000
```

## PostgreSQL

PostgreSQL runs as a Kubernetes Deployment for this learning project.

Service:

```text
postgres
Port: 5432
```

---

# 19. Kubernetes Configuration and Secrets

Non-sensitive configuration is managed using a ConfigMap.

Sensitive database credentials are stored using a Kubernetes Secret.

The backend receives:

```text
DB_USER
DB_PASSWORD
DB_HOST
DB_PORT
DB_NAME
PORT
```

The database password is intentionally not committed to Git.

---

# 20. Argo CD / GitOps

Argo CD is used for continuous delivery and GitOps.

The basic principle is:

```text
Git Repository
      |
      v
    Argo CD
      |
      v
     EKS
```

Instead of manually applying Kubernetes manifests every time, Argo CD continuously compares the desired state stored in Git with the state running in Kubernetes.

If a change is committed to Git:

```text
Developer
    |
    v
GitHub
    |
    v
Argo CD
    |
    v
EKS
```

Argo CD synchronizes the Kubernetes resources.

---

# 21. AWS Load Balancer Controller

AWS Load Balancer Controller was installed in EKS.

Its purpose is to allow Kubernetes Ingress resources to create and manage AWS Application Load Balancers.

Flow:

```text
Kubernetes Ingress
       |
       v
AWS Load Balancer Controller
       |
       v
AWS Application Load Balancer
```

The controller uses an IAM role through AWS IAM/OIDC integration.

---

# 22. Application Load Balancer

The application uses an internet-facing AWS Application Load Balancer.

Traffic flow:

```text
User Browser
     |
     | HTTP
     v
AWS ALB
     |
     v
Kubernetes Ingress
     |
     v
Frontend Service
```

The frontend Nginx configuration proxies API requests to the backend service.

Therefore:

```text
/api/*
   |
   v
Backend
```

while normal frontend requests are served by the React application.

---

# 23. End-to-End Request Flow

When a user accesses the application:

```text
User
 |
 v
AWS Application Load Balancer
 |
 v
Kubernetes Ingress
 |
 v
Frontend Service
 |
 v
Frontend Pod / Nginx
 |
 +--------------------+
 |                    |
 | Normal request     | /api request
 |                    |
 v                    v
React Application   Backend Service
                         |
                         v
                    Backend Pod
                         |
                         v
                  PostgreSQL Service
                         |
                         v
                  PostgreSQL Pod
```

This confirms communication between all three application tiers.

---

# 24. CI/CD vs GitOps

The project separates CI and CD responsibilities.

## Continuous Integration

GitHub Actions handles:

```text
Code
 |
 v
Lint
 |
 v
SonarQube
 |
 v
Docker Build
 |
 v
Trivy
 |
 v
Docker Hub
```

## Continuous Delivery / GitOps

Argo CD handles:

```text
Git
 |
 v
Kubernetes Manifests
 |
 v
Argo CD
 |
 v
EKS
```

This separation makes the pipeline easier to understand and maintain.

---

# 25. Security Practices

The project includes several security controls:

### Static code analysis

```text
SonarQube
```

### Container vulnerability scanning

```text
Trivy
```

### Kubernetes secrets

Database credentials are stored in Kubernetes Secrets instead of being committed to Git.

### IAM

AWS Load Balancer Controller uses an IAM role instead of hard-coded AWS credentials.

### Private EKS nodes

Worker nodes run in private subnets.

### Git ignore

Sensitive and generated files such as:

```text
.env
*.tfstate
Terraform generated files
Kubernetes backend secret manifest
```

are excluded from Git.

---

# 26. Repository Deployment Flow

The complete workflow is:

```text
1. Developer changes application code
                |
                v
2. Push code to GitHub
                |
                v
3. GitHub Actions starts
                |
                v
4. Linting
                |
                v
5. SonarQube analysis
                |
                v
6. Docker image build
                |
                v
7. Trivy security scan
                |
                v
8. Push images to Docker Hub
                |
                v
9. Kubernetes manifests define desired state
                |
                v
10. Argo CD monitors Git
                |
                v
11. Argo CD synchronizes EKS
                |
                v
12. Kubernetes deploys application
                |
                v
13. AWS Load Balancer Controller
                |
                v
14. AWS Application Load Balancer
                |
                v
15. User accesses application
```

---

# 27. Infrastructure Flow

Terraform provisions:

```text
Terraform
   |
   +-- VPC
   |
   +-- Subnets
   |
   +-- Internet Gateway
   |
   +-- NAT Gateway
   |
   +-- Route Tables
   |
   +-- Security Groups
   |
   +-- IAM
   |
   +-- EKS
   |
   +-- Node Group
```

Then Kubernetes and Argo CD handle the application layer.

This creates a clear separation:

```text
Terraform
   ↓
AWS Infrastructure

Kubernetes
   ↓
Application Infrastructure

Argo CD
   ↓
GitOps Deployment

GitHub Actions
   ↓
CI / Security / Image Build
```

---

# 28. What Was Implemented in This Project

The following components were actually implemented and tested:

* GitHub repository
* React frontend
* Node.js backend
* PostgreSQL
* Dockerfiles
* Docker Compose
* GitHub Actions
* Node.js linting
* SonarQube
* Trivy
* Docker Hub
* Kubernetes manifests
* Kubernetes Deployments
* Kubernetes Services
* Kubernetes ConfigMap
* Kubernetes Secret
* Kubernetes Ingress
* Terraform
* AWS VPC
* Public and private subnets
* Internet Gateway
* NAT Gateway
* IAM roles and policies
* Amazon EKS
* EKS worker nodes
* Argo CD
* GitOps deployment
* AWS Load Balancer Controller
* AWS Application Load Balancer
* End-to-end frontend/backend/database communication

---

# 29. Components Intentionally Not Included

Observability was intentionally excluded from this project to keep the project focused.

The following will be implemented in a future project:

```text
Prometheus
Grafana
Alertmanager
Loki
Centralized Logging
Advanced Monitoring
Alerting
```

The next project can focus specifically on observability and advanced DevOps practices.

---

# 30. Interview Explanation

A concise way to explain this project in an interview:

> I worked on an end-to-end DevOps project where I containerized a React and Node.js application with PostgreSQL and deployed it on Amazon EKS.
>
> I used GitHub Actions for CI, including linting, SonarQube static analysis, Docker image builds, Trivy vulnerability scanning, and pushing images to Docker Hub.
>
> I used Terraform to provision the AWS infrastructure, including the VPC, public and private subnets, NAT Gateway, IAM roles, and EKS cluster.
>
> For deployment, I used Kubernetes manifests and Argo CD to implement GitOps. Argo CD continuously synchronizes the desired Kubernetes state from Git with the EKS cluster.
>
> I also configured the AWS Load Balancer Controller so that the Kubernetes Ingress creates an internet-facing Application Load Balancer. The ALB routes traffic to the frontend, and the frontend proxies API requests to the backend, which communicates with PostgreSQL.
>
> The result is a complete CI, security scanning, containerization, GitOps, Kubernetes, and AWS deployment workflow.

---

# 31. Final Architecture Summary

```text
                         ┌─────────────────┐
                         │    Developer    │
                         └────────┬────────┘
                                  │
                                  │ Git Push
                                  ▼
                         ┌─────────────────┐
                         │     GitHub      │
                         └────────┬────────┘
                                  │
                                  ▼
                    ┌──────────────────────────┐
                    │     GitHub Actions       │
                    │                          │
                    │  Lint                    │
                    │  SonarQube               │
                    │  Docker Build            │
                    │  Trivy                   │
                    │  Docker Hub Push         │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                         ┌─────────────────┐
                         │   Docker Hub    │
                         └────────┬────────┘
                                  │
                                  │ Images
                                  │
                                  ▼
┌─────────────────────────────────────────────────────────────┐
│                         AWS                                 │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                    VPC                                │  │
│  │                                                       │  │
│  │  Public Subnets             Private Subnets          │  │
│  │       │                           │                   │  │
│  │       ▼                           ▼                   │  │
│  │   ┌─────────┐              ┌──────────────┐          │  │
│  │   │   ALB   │              │   EKS Nodes  │          │  │
│  │   └────┬────┘              └──────┬───────┘          │  │
│  │        │                          │                   │  │
│  │        ▼                          ▼                   │  │
│  │   Kubernetes Ingress          Kubernetes              │  │
│  │                                   │                   │  │
│  │                          ┌────────┼────────┐          │  │
│  │                          │        │        │          │  │
│  │                          ▼        ▼        ▼          │  │
│  │                       Frontend  Backend  PostgreSQL   │  │
│  │                          │        │        │          │  │
│  │                          └───────►└───────►           │  │
│  │                                                       │  │
│  └───────────────────────────────────────────────────────┘  │
│                                                             │
└─────────────────────────────────────────────────────────────┘

                    ▲
                    │
                    │ GitOps
                    │
             ┌──────┴──────┐
             │   Argo CD   │
             └─────────────┘
                    ▲
                    │
                    │ Kubernetes Manifests
                    │
                 GitHub
```

---

# 32. Key DevOps Concepts Demonstrated

This project demonstrates practical knowledge of:

```text
Git
GitHub
CI/CD
GitHub Actions
DevSecOps
Static Code Analysis
Container Security
Docker
Docker Compose
Container Registry
Kubernetes
Kubernetes Networking
Ingress
Secrets
ConfigMaps
Terraform
Infrastructure as Code
AWS VPC
IAM
Amazon EKS
AWS Load Balancer Controller
Application Load Balancer
Argo CD
GitOps
Three-Tier Architecture
Cloud Deployment
```

The project provides an end-to-end example of taking application source code and transforming it into a containerized, security-scanned, Kubernetes-based AWS deployment with automated CI and GitOps-based delivery.
