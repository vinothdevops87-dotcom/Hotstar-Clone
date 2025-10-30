# 🚀 DevSecOps Project: Hotstar Clone (with Quality Gate Integration)

## 🧩 Overview
This project demonstrates a **complete DevSecOps pipeline** for a **Hotstar Clone** application.  
It includes **SonarQube Quality Gate checks**, **Docker-based deployment**, and **Terraform-managed AWS EC2 infrastructure** — fully automated via **GitHub Actions**.

---

## 🏗️ Architecture Overview

**Pipeline Flow:**
1. Developer pushes code to GitHub main branch.
2. GitHub Actions triggers:
   - SonarQube analysis (code quality, bugs, vulnerabilities)
   - Quality Gate verification
   - Docker image build and push to Docker Hub
   - Deployment to EC2 only if Quality Gate passes ✅

---

## 🧱 Tech Stack

| Category | Tools Used |
|-----------|-------------|
| Cloud | AWS EC2 |
| IaC | Terraform |
| CI/CD | GitHub Actions |
| Code Quality | SonarQube |
| Containerization | Docker |
| OS | Ubuntu/Linux |
| Frontend | React.js |
| SCM | Git & GitHub |

---

## ☁️ Infrastructure Setup (Terraform)

### 1️⃣ Terraform Configuration

**main.tf**
```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "devsecops_ec2" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  key_name      = "your-key-name"

  tags = {
    Name = "DevSecOps-EC2"
  }
} 
2️⃣ Commands
bash
Copy code
terraform init
terraform plan
terraform apply -auto-approve
Get the EC2 public IP after Terraform completes.

🔐 SonarQube Setup
1️⃣ Run SonarQube (Local or Remote)
bash
Copy code
docker run -d --name sonarqube -p 9000:9000 sonarqube:lts
Access → http://localhost:9000

2️⃣ Create Project & Token
Create project named hotstar_clone

Generate a SONAR_TOKEN

Note your SONAR_HOST_URL (e.g., http://<ec2-ip>:9000)

3️⃣ Set Up Quality Gate
Go to SonarQube → Quality Gates

Use the default or create a custom Quality Gate (e.g., “No Critical Issues”)

Ensure your project is associated with that gate

⚙️ GitHub Actions (with Quality Gate)
Create: .github/workflows/devsecops-pipeline.yml

name: deploy-hotstar-clone

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      # Run SonarQube scan
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@v2
        with:
          projectBaseDir: .
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

      # Enforce Quality Gate (fail pipeline if not passed)
      - name: Wait for SonarQube Quality Gate
        uses: sonarsource/sonarqube-quality-gate-action@v1
        timeout-minutes: 10
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_HUB_USERNAME }}
          password: ${{ secrets.DOCKER_HUB_ACCESS_TOKEN }}

      - name: Build and Push Docker Image
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ secrets.DOCKER_HUB_USERNAME }}/hotstar-clone:latest

      - name: Deploy to Remote Server via SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.SSH_REMOTE_HOST }}
          username: ${{ secrets.SSH_REMOTE_USER }}
          key: ${{ secrets.REMOTE_SSH_KEY }}
          port: ${{ secrets.SSH_REMOTE_PORT }}
          script: |
            docker pull ${{ secrets.DOCKER_HUB_USERNAME }}/hotstar-clone:latest
            docker stop hotstar-clone || true
            docker rm hotstar-clone || true
            docker run -d -p 3000:80 --name hotstar-clone ${{ secrets.DOCKER_HUB_USERNAME }}/hotstar-clone:latest
🚀 Deployment Steps
1️⃣ Push your code to main
2️⃣ Pipeline automatically runs:

SonarQube analysis

Quality Gate check

Docker image build + push

EC2 deployment (only if gate passes)

3️⃣ Access app at:

cpp
Copy code
http://<EC2-Public-IP>:3000
📸 Recommended Screenshots
✅ GitHub Actions workflow (successful run)

🧠 SonarQube Quality Gate result

🌐 Hotstar Clone running on EC2

🧠 Key Learnings
✔️ End-to-End DevSecOps pipeline
✔️ Quality Gate enforcement for code integrity
✔️ Infrastructure as Code using Terraform
✔️ Dockerized deployment for consistency
✔️ Continuous integration via GitHub Actions

📚 Future Improvements
Add Snyk for dependency scanning

Integrate ArgoCD for GitOps

Manage secrets with HashiCorp Vault

Add Nginx as reverse proxy

🌐 Links
🔗 GitHub Repository: Hotstar Clone
🔗 Live Server: http://<EC2-Public-IP>:3000
🔗 Documentation: Available in Word format

