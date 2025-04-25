# vm-docker-container

## 🚀 Production Setup Guide (Rocky Linux + Docker + GitLab CI/CD)

This guide explains how to set up a production-ready environment for a Python web application using:

- ✅ Rocky Linux VM
- 🐳 Docker & Docker Compose
- 🔐 GitLab CI/CD for automated deployment

---

## 🧰 Requirements

- A Rocky Linux VM (on-premise or cloud)
- SSH access to the VM
- A Python web app (Flask, FastAPI, Django, etc)
- GitLab project (self-hosted or gitlab.com)

---

## ⚙️ 1. Update the VM

```bash
sudo dnf update -y
sudo dnf install -y epel-release
