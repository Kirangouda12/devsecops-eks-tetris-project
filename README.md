# 🚀 End-to-End DevSecOps Kubernetes Project

![DevSecOps](https://img.shields.io/badge/DevSecOps-Pipeline-brightgreen)
![Kubernetes](https://img.shields.io/badge/Kubernetes-EKS%201.28-blueviolet)
![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-orange)
![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-blue)
![Terraform](https://img.shields.io/badge/Terraform-IaC-9cf)
![Docker](https://img.shields.io/badge/Docker-Containerization-blue)
![SonarQube](https://img.shields.io/badge/SonarQube-Code%20Quality-yellow)
![Trivy](https://img.shields.io/badge/Trivy-Security%20Scan-red)
![AWS](https://img.shields.io/badge/AWS-EKS%20%7C%20EC2%20%7C%20S3-FF9900)

---

## 📌 Project Overview

This project demonstrates a **production-grade DevSecOps CI/CD pipeline** for deploying a Tetris web application on **AWS EKS (Kubernetes v1.28)**.

Security is integrated at every stage of the pipeline — from static code analysis and dependency scanning to container image vulnerability checks — following the **"Shift Left Security"** principle.

The entire infrastructure is provisioned as code using **Terraform**, with remote state stored in **S3 + DynamoDB**.

---

## 🏗️ Architecture

![Infrastructure Diagram](assets/Infra.gif)

---

## 📂 Repository Structure

```
├── EKS-TF/                    # Terraform IaC for AWS EKS cluster
│   ├── vpc.tf                 # VPC, subnets, internet gateway
│   ├── eks-cluster.tf         # EKS cluster (v1.28)
│   ├── eks-node-group.tf      # Worker node group
│   ├── iam-role.tf            # IAM roles for EKS & nodes
│   ├── iam-policy.tf          # IAM policies
│   ├── backend.tf             # S3 + DynamoDB remote state
│   ├── provider.tf            # AWS provider config
│   ├── variables.tf           # Input variables
│   └── variables.tfvars       # Variable values
│
├── Jenkins-Server-TF/         # Terraform IaC for Jenkins EC2 server
│   ├── ec2.tf                 # t2.2xlarge EC2 instance (30GB)
│   ├── vpc.tf                 # VPC & networking for Jenkins
│   ├── iam-role.tf            # IAM role for EC2
│   ├── iam-policy.tf          # IAM policies
│   ├── iam-instance-profile.tf
│   ├── gather.tf              # Data sources (AMI lookup)
│   ├── backend.tf             # Remote state config
│   ├── provider.tf
│   ├── variables.tf
│   ├── variables.tfvars
│   └── tools-install.sh       # Bootstraps: Jenkins, Docker, SonarQube,
│                              #             AWS CLI, kubectl, Terraform, Trivy
│
├── Jenkins-Pipeline-Code/     # Jenkins CI/CD pipeline definitions
│   ├── Jenkinsfile-TetrisV1   # Pipeline: build → scan → push → deploy V1
│   ├── Jenkinsfile-TetrisV2   # Pipeline: build → scan → push → deploy V2
│   └── Jenkinsfile-EKS-Terraform  # Pipeline: EKS infra provision/destroy
│
├── Manifest-file/             # Kubernetes manifests (ArgoCD GitOps)
│   └── deployment-service.yml # Deployment (3 replicas) + LoadBalancer Service
│
├── Tetris-V1/                 # Tetris app v1 (React) + Dockerfile
├── Tetris-V2/                 # Tetris app v2 (React, enhanced UI) + Dockerfile
└── assets/                    # Architecture diagrams
```

---

## 🛠️ Tech Stack

| Category | Tools |
|----------|-------|
| **Cloud** | AWS (EKS, EC2, S3, DynamoDB, VPC, IAM) |
| **IaC** | Terraform (remote state: S3 + DynamoDB) |
| **CI/CD** | Jenkins |
| **GitOps / CD** | ArgoCD |
| **Containerization** | Docker |
| **Orchestration** | Kubernetes (EKS v1.28, 3 replicas) |
| **Code Quality** | SonarQube |
| **Security Scanning** | Trivy (filesystem + image), OWASP Dependency-Check |
| **Application** | React.js (Node.js) |

---

## 🔄 CI/CD Pipeline Flow

```
Developer Push
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│                    JENKINS PIPELINE                      │
│                                                         │
│  1. Clean Workspace                                     │
│  2. Checkout Code from GitHub                           │
│  3. SonarQube Analysis  ──► Quality Gate Check          │
│  4. npm install (dependency resolution)                 │
│  5. OWASP Dependency-Check Scan                         │
│  6. Trivy Filesystem Scan                               │
│  7. Docker Image Build                                  │
│  8. Docker Image Push → DockerHub                       │
│  9. Trivy Image Scan                                    │
│ 10. Update Manifest (deployment-service.yml) + Git Push │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
              ┌──────────────────┐
              │     ArgoCD       │  ◄── Watches Manifest-file/
              │  (GitOps Sync)   │       deployment-service.yml
              └────────┬─────────┘
                       │
                       ▼
              ┌──────────────────┐
              │   AWS EKS        │
              │  (3 Replicas)    │
              │  LoadBalancer    │
              └──────────────────┘
```

---

## ⚙️ Infrastructure Setup

### Step 1 — Provision Jenkins Server

```bash
cd Jenkins-Server-TF
terraform init
terraform fmt
terraform validate
terraform plan -var-file="variables.tfvars"
terraform apply -var-file="variables.tfvars" -auto-approve
```

> This spins up a `t2.2xlarge` EC2 instance and auto-installs: Jenkins, Docker, SonarQube (container), AWS CLI, kubectl, Terraform, and Trivy via `tools-install.sh`.

### Step 2 — Provision EKS Cluster

```bash
cd EKS-TF
terraform init
terraform fmt
terraform validate
terraform plan -var-file="variables.tfvars"
terraform apply -var-file="variables.tfvars" -auto-approve
```

> Creates EKS cluster v1.28 with managed node groups, VPC, IAM roles, and stores state in S3 + DynamoDB.

### Step 3 — Configure kubectl

```bash
aws eks update-kubeconfig --region us-east-1 --name <cluster-name>
kubectl get nodes
```

### Step 4 — Install ArgoCD on EKS

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

### Step 5 — Run Jenkins Pipelines

- Run `Jenkinsfile-EKS-Terraform` to manage EKS infra via Jenkins
- Run `Jenkinsfile-TetrisV1` to build, scan, and deploy Tetris V1
- Run `Jenkinsfile-TetrisV2` to build, scan, and deploy Tetris V2

---

## 🔒 Security Stages in Pipeline

| Stage | Tool | Purpose |
|-------|------|---------|
| Static Analysis | SonarQube | Code quality & bug detection |
| Quality Gate | SonarQube | Block pipeline on low quality score |
| Dependency Scan | OWASP Dependency-Check | Known CVEs in npm packages |
| Filesystem Scan | Trivy | Vulnerabilities in source files |
| Image Scan | Trivy | CVEs in built Docker image |

---

## 🚀 Getting Started (Clone & Run)

```bash
# Clone the repository
git clone https://github.com/Kirangouda12/<repo-name>.git
cd <repo-name>

# Start with Jenkins Server provisioning
cd Jenkins-Server-TF
terraform init && terraform apply -var-file="variables.tfvars" -auto-approve
```

---

## 📋 Prerequisites

- AWS CLI configured with appropriate IAM permissions
- Terraform >= 0.13.0
- kubectl installed
- Docker installed
- An S3 bucket and DynamoDB table for Terraform remote state

---

## 👤 Author

**Kirangouda**
- GitHub: [@Kirangouda12](https://github.com/Kirangouda12)
- BE/CSE (AI & ML) Graduate | DevSecOps Enthusiast

---

## 📄 License

This project is licensed under the Apache-2.0 License — see the [LICENSE](LICENSE) file for details.
