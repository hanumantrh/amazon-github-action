# GitHub Actions Self-Hosted Runnner Amazon Website CICD DevOps — Setup Guide


## For more projects, check out  
## [https://harishnshetty.github.io/projects.html](https://harishnshetty.github.io/projects.html)
## [youtube-Link](https://www.youtube.com/@devopsHarishNShetty)

---
![img alt](https://github.com/harishnshetty/GitHub-Action-Amazon-DevSecOps/blob/bd54e7fd5d2ce24b50614b51ceb390dc6f860334/img.png)
---



---

## Table of Contents

- [Ports to Enable in Security Group](#ports-to-enable-in-security-group)
- [Prerequisites](#prerequisites)
- [System Update & Common Packages](#system-update--common-packages)
- [Docker](#docker)
- [Trivy (Vulnerability Scanner)](#trivy-vulnerability-scanner)
- [SonarQube (Docker)](#sonarqube-docker)
- [npm Installation](#npm-installation)



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
