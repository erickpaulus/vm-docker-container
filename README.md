This repository contains my personal experiment in preparing a production-ready environment for my PhD project. To be honest, I am a newbie in this area of infrastructure architecture. However, it is never too late for continious learning. 

## Production Setup Guide (Rocky Linux + Docker + GitLab CI/CD)

This guide explains how to set up a production-ready environment for a Python web application using:

- Rocky Linux VM
- Docker & Docker Compose
- GitLab CI/CD for automated deployment

---

## Requirements

- A Rocky Linux VM (on-premise or cloud)
- SSH access to the VM
- A Python web app (Flask, FastAPI, Django, etc)
- GitLab project (self-hosted or gitlab.com)

---

## 1. Update the VM
First command: updates your system.
Second command: installs the EPEL(Extra Packages for Enterprise Linux) repository, so you have access to more software packages.
```bash
sudo dnf update -y
sudo dnf install -y epel-release
```

## 2. Install Docker
What it is: The core tool that lets you build, run, and manage containers.

You use it to:
- Create container images.
- Start, stop, and manage individual containers.
- Pull images from Docker Hub.
- Use the Docker CLI (like docker run, docker build, docker pull, etc.).
  
```bash
sudo dnf install -y dnf-utils device-mapper-persistent-data lvm2
sudo dnf remove -y podman buildah
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io --nobest --allowerasing

sudo systemctl start docker
sudo systemctl enable docker
docker --version
```
(Optional) Run Docker Without Sudo
```bash
sudo usermod -aG docker $USER
# Logout and log back in to apply group change
```

## 3. Install Docker Compose v2
What it is: A tool for defining and running multi-container applications using a YAML file (docker-compose.yml).

You use it to:
- Describe multiple containers, their networks, volumes, and relationships in a single file.
- Start, stop, and manage all containers at once.
  
```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins
sudo curl -SL https://github.com/docker/compose/releases/latest/download/docker-compose-linux-x86_64 \
  -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose

docker compose version
```
## 4. Set Up SSH Key for GitLab CI/CD
Generate an SSH key on your local machine (if not already):
```bash
ssh-keygen -t rsa -b 4096 -C "gitlab-deploy"
```
Copy the public key to the VM:
```bash
ssh-copy-id youruser@your-vm-ip
```
Verify SSH login works without password:
```bash
ssh youruser@your-vm-ip
```
Then in GitLab:
- Go to Settings > CI/CD > Variables
- Add a variable:
  - SSH_PRIVATE_KEY: Paste your private key
  - Mark as protected (optional)

## 5. Install Git (if not installed)
```bash
sudo dnf install git -y
git --version
```
## 6. Create App Directory
```bash
mkdir -p ~/myapp
cd ~/myapp
```
## 7. (Optional) Open HTTP/HTTPS Ports
```bash
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --permanent --add-port=443/tcp
sudo firewall-cmd --reload
```
## 8. Verify Tools Installed
```bash
docker --version
docker compose version
git --version

```
## 9. GitLab CI/CD Setup
```bash
stages:
  - build
  - deploy

variables:
  DOCKER_DRIVER: overlay2

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD" $CI_REGISTRY
    - docker build -t $CI_REGISTRY_IMAGE:latest .
    - docker push $CI_REGISTRY_IMAGE:latest

deploy:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache openssh
  script:
    - echo "$SSH_PRIVATE_KEY" > /tmp/key && chmod 600 /tmp/key
    - ssh -o StrictHostKeyChecking=no -i /tmp/key $DEPLOY_USER@$DEPLOY_HOST <<EOF
        cd /home/$DEPLOY_USER/myapp
        git pull origin main
        docker-compose down
        docker-compose pull
        docker-compose up -d
      EOF

```
GitLab CI/CD Variables to Add
Go to: Settings > CI/CD > Variables and add:

Key | Value | Type
CI_REGISTRY_USER | Your GitLab username | Protected
CI_REGISTRY_PASSWORD | GitLab personal access token | Protected
DEPLOY_HOST | Your VM IP or domain | Protected
DEPLOY_USER | Your SSH username on VM | Protected
SSH_PRIVATE_KEY | Your SSH private key (match public key on VM) | Protected

## Next Steps
- Add your docker-compose.yml for the app
- Optionally configure Nginx or Caddy for reverse proxy
- Secure with SSL (e.g., Let's Encrypt)
- Add .env file and secrets to GitLab CI/CD securely
- Add automated test stage (pytest, flake8, etc)
