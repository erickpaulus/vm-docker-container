# vm-docker-container

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

```bash
sudo dnf update -y
sudo dnf install -y epel-release
```

## 2. Install Docker
```bash
sudo dnf install -y dnf-utils device-mapper-persistent-data lvm2
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io

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
