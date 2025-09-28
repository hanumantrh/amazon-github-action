# **Amazon Website CI/CD Pipeline with GitHub Actions, SonarQube, Trivy, and Docker**

## **Overview**
This project demonstrates how to set up a **CI/CD pipeline** for an Amazon-style e-commerce website using **GitHub Actions** with a **self-hosted runner**.  

The pipeline automates the following steps:
1. **Code Quality Check** → Using **SonarQube** hosted on AWS EC2.  
2. **Vulnerability Scan** → Using **Trivy** to scan Docker images before deployment.  
3. **Docker Build & Push** → Builds an optimized Docker image and pushes it to **Docker Hub**.  
4. **Deployment** → Run the Docker container on an AWS EC2 instance.  
5. **Notifications** → Email notifications for pipeline status.

> **Why Self-Hosted Runner?**  
> A self-hosted runner gives more **control**, **faster builds**, and **direct access** to your private cloud resources like EC2 and SonarQube.

---

## **Pipeline Data Flow**

```text
Developer → GitHub Repository → GitHub Actions (Self-Hosted Runner)
                                       │
                ┌──────────────────────┼──────────────────────┐
                │                      │                      │
        SonarQube Code Analysis     Trivy Vulnerability Scan
                │                      │
                └───────────────> Docker Build & Push
                                      │
                                  Docker Hub
                                      │
                                  AWS EC2 Deploy
                                      │
                                Email Notifications
```

---
![img alt](https://github.com/hanumantrh/amazon-github-action/blob/main/image.png?raw=true)
---


---

## **How It Works**

### **1. Code Push → GitHub Actions Trigger**
- When you **push code** or create a **pull request**, GitHub Actions automatically triggers the pipeline defined in `.github/workflows/main.yml`.

---

### **2. Code Quality Analysis with SonarQube**
- **SonarQube**, running in a Docker container on an EC2 instance, analyzes:
  - Code smells  
  - Bugs  
  - Vulnerabilities  
  - Code coverage
- The results are accessible from the SonarQube web dashboard.

---

### **3. Vulnerability Scanning with Trivy**
- Trivy scans the **Docker image** for known vulnerabilities **before deploying**.
- If vulnerabilities are found, the pipeline **fails early**, preventing insecure deployments.

---

### **4. Build & Push Docker Image**
- The pipeline uses a **multi-stage Dockerfile** to create a **lightweight, production-ready image**.
- The image is tagged with:
  - `latest`  
  - Git commit SHA (for traceability).
- The image is pushed securely to **Docker Hub** using GitHub Secrets for authentication.

---

### **5. Deploy on AWS EC2**
- The EC2 instance pulls the image from Docker Hub and runs it as a container.
- Your Amazon Website is then accessible over the public IP of EC2.

---

### **6. Email Notification**
- The final stage sends an email with:
  - Pipeline status (Success/Failure)
  - Commit details
  - Links to logs or reports

---

## Implementation
## Prerequisites

- AWS EC2 instance (Ubuntu 20.04+)
- GitHub repository.
- Docker Hub account for hosting container images.
- Email account (with app password for notifications).
- Domain name (optional) for accessing your site.

---

## Ports to Enable in Security Group

| Service    | Port  |
|------------|-------|
| HTTP       | 80    |
| SSH        | 22    |
| SonarQube  | 9000  |

---
## System Update & Common Packages

```bash
sudo apt update
sudo apt upgrade -y

# Common tools
sudo apt install -y bash-completion wget git zip unzip curl jq net-tools build-essential ca-certificates apt-transport-https gnupg fontconfig
```

Reload bash completion if needed:
```bash
source /etc/bash_completion
```

**Install latest Git:**
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git -y
```

---

## Docker

Official docs: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add user to docker group (log out / in or newgrp to apply)
sudo usermod -aG docker $USER
newgrp docker
docker ps
```


Check Docker status:
```bash
sudo systemctl status docker
```

---

## Trivy (Vulnerability Scanner)

Docs: [https://trivy.dev/v0.65/getting-started/installation/](https://trivy.dev/v0.65/getting-started/installation/)

```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy


trivy --version
```

---

## SonarQube (Docker)

```bash
docker run -d --name sonarqube \
  -p 9000:9000 \
  -v sonarqube_data:/opt/sonarqube/data \
  -v sonarqube_logs:/opt/sonarqube/logs \
  -v sonarqube_extensions:/opt/sonarqube/extensions \
  sonarqube:lts-community
```

> **Access SonarQube UI:** `http://<EC2-IP>:9000` (default credentials: admin/admin – change on first login).

---

## npm Installation

Docs: [https://nodejs.org/en/download/](https://nodejs.org/en/download/)

```bash
# Download and install nvm:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

# In lieu of restarting the shell
\. "$HOME/.nvm/nvm.sh"

# Download and install Node.js:
nvm install 22

# Verify the Node.js version:
node -v # Should print "v22.18.0"
nvm current # Should print "v22.18.0"

# Verify npm version:
npm -v # Should print "10.9.3"
```

## Github Credentials to Store

| Purpose       | ID            | Type          | Notes                               |
|---------------|---------------|---------------|-------------------------------------|
| Email         | EMAIL_USER    | emailaddress@gmail.com|                                  |
| Email-app-Pass     | EMAIL_PASS   | Secret text   | From app password         |
| Docker-username    | DOCKER_USERNAME   | your-docker-id   | From your Docker Hub profile       |
| Docker-username    | DOCKER_PASSWORD   | token   | From your Docker Hub token       |
| sonar-qube    | follow the same step

---

## GitHub Actions Self-Hosted Runner Setup

GitHub Actions allows you to run workflows on your **own server** using a self-hosted runner.

### 1. Runner Requirements
- OS: Linux  
- Architecture: Match your server architecture (x64, ARM, etc.)

### 2. Download and Install Runner
```bash
# Create a folder for the runner
mkdir actions-runner && cd actions-runner

# Download the latest runner package
curl -o actions-runner-linux-x64-2.328.0.tar.gz -L \
https://github.com/actions/runner/releases/download/v2.328.0/actions-runner-linux-x64-2.328.0.tar.gz

# Optional: Validate the hash
echo "01066fad3a2893e63e6ca880ae3a1fad5bf9329d60e77ee15f2b97c148c3cd4e  actions-runner-linux-x64-2.328.0.tar.gz" | shasum -a 256 -c

# Extract the installer
tar xzf ./actions-runner-linux-x64-2.328.0.tar.gz

```
---
## Configure the Runner
```bash
# Create the runner and start the configuration experience
$ ./config.sh --url https://github.com/hanumantrh/amazon-github-action --token BTFQR2W7B4XCCBTLBOQXAGLI3FO5I

Replace YOUR_RUNNER_TOKEN with the token generated in your repository:
Settings → Actions → Runners → Add Runner → Generate Token
```
---
## Start the Runner
```bash
# Last step, run it!
$ ./run.sh

```
---
> For additional details about configuring, running, or shutting down the runner, please check out our https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners