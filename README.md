# Node.js CI/CD Deployment on AWS EC2 using GitHub Actions & Docker

## Overview

This project demonstrates a **production-style CI/CD pipeline** for deploying a Dockerized Node.js application to an **AWS EC2 (Ubuntu)** server using **GitHub Actions** and **SSH-based deployment**.

The focus of this project is not just running a Node.js app, but understanding and implementing how **code is securely deployed to a real server**, covering infrastructure, authentication, permissions, and automation.

This closely mirrors how small to mid-scale backend systems are deployed in real-world environments.

---

## What This Project Demonstrates

* Secure EC2 provisioning with Ubuntu
* SSH authentication using public/private key cryptography
* Docker & Docker Compose for application runtime
* Automated deployments using GitHub Actions
* Linux permissions and Docker daemon access
* End-to-end CI/CD triggered on every push to `main`

---

## Tech Stack

* **Backend**: Node.js
* **Infrastructure**: AWS EC2 (Ubuntu)
* **Containerization**: Docker, Docker Compose
* **CI/CD**: GitHub Actions
* **Authentication**: SSH (key-based)

---

## High-Level Architecture

```text
┌──────────────────┐
│   Developer      │
│ (Local Machine)  │
└────────┬─────────┘
         │ git push (main)
         ▼
┌──────────────────────────┐
│        GitHub             │
│   Repository + Actions    │
└────────┬─────────────────┘
         │ CI/CD Trigger
         ▼
┌──────────────────────────┐
│   GitHub Actions Runner   │
│  (Ubuntu, ephemeral VM)  │
└────────┬─────────────────┘
         │ SSH (private key)
         ▼
┌──────────────────────────────────┐
│        AWS EC2 Instance           │
│        Ubuntu Server              │
│                                  │
│  ┌───────────────┐               │
│  │ Docker        │               │
│  │ Compose       │               │
│  └──────┬────────┘               │
│         │                        │
│  ┌──────▼────────┐               │
│  │ Node.js App   │               │
│  │ (Container)   │               │
│  └───────────────┘               │
└──────────────────────────────────┘
```

---

## CI/CD Flow (Step by Step)

```text
1. Code pushed to main branch
        │
        ▼
2. GitHub Actions workflow starts
        │
        ▼
3. Repository checked out
        │
        ▼
4. SSH connection established
   (using private key from secrets)
        │
        ▼
5. Commands executed on EC2:
   - git pull
   - docker compose build
   - docker compose up -d
        │
        ▼
6. Updated application runs on server
```

---

## SSH Authentication (First Principles)

This project uses **key-based SSH authentication**, not passwords.

```text
┌──────────────────────┐
│ GitHub Actions       │
│ (Client)             │
│                      │
│ Private SSH Key      │
└──────────┬───────────┘
           │ proves identity
           ▼
┌──────────────────────┐
│   SSH Handshake       │
│  (Cryptographic)     │
└──────────┬───────────┘
           │ verified against
           ▼
┌──────────────────────────────┐
│ EC2 Server                   │
│ ~/.ssh/authorized_keys       │
│ (Public SSH Key stored)      │
└──────────────────────────────┘
```

**Key points:**

* Public key lives on the server
* Private key is stored securely in GitHub Secrets
* No passwords are used
* Secrets are never committed to the repository

---

## Docker Runtime Model

```text
┌──────────────────────────────┐
│        EC2 Instance          │
│                              │
│  ┌────────────────────────┐ │
│  │ Docker Daemon (root)    │ │
│  └──────────┬─────────────┘ │
│             │               │
│  ┌──────────▼────────────┐ │
│  │ Application Container │ │
│  │ Node.js Server        │ │
│  │ Port: 3000            │ │
│  └──────────────────────┘ │
│                              │
└──────────────────────────────┘
```

---

## Linux Permissions & Security Model

```text
ubuntu user
   │
   ├── SSH access (key-based)
   │
   ├── git pull (repository)
   │
   └── docker group
        │
        ▼
Docker socket (/var/run/docker.sock)
        │
        ▼
Docker daemon (root)
```

* Docker runs as root
* Non-root access enabled via `docker` group
* CI runs in a **non-interactive SSH session**, matching real production behavior

---

## GitHub Actions Workflow

### Trigger

* Runs automatically on every push to the `main` branch

### Workflow File

```
.github/workflows/deploy.yaml
```

### Core Deployment Step

```yaml
- name: Deploy via SSH
  uses: appleboy/ssh-action@v1.0.3
  with:
    host: ${{ secrets.SSH_HOST }}
    username: ubuntu
    key: ${{ secrets.SSH_KEY }}
    script: |
      cd /home/ubuntu/nodejs-application-deploy-github-action
      git pull
      docker compose up -d --build
```

---

## Secrets Management

Configured in GitHub → Repository Settings → Secrets:

| Secret Name | Purpose                                |
| ----------- | -------------------------------------- |
| `SSH_HOST`  | EC2 public IPv4 address                |
| `SSH_KEY`   | Private SSH key used by GitHub Actions |

No secrets or credentials are stored in the repository.

---

## Why This Project Is Relevant

This project demonstrates:

* Real infrastructure setup, not mock environments
* Secure authentication using SSH keys
* Docker-based deployments
* Automated CI/CD pipelines
* Debugging of real Linux and Docker permission issues

It shows **how backend code actually reaches production**, not just how to write application logic.

---

## Possible Improvements / Extensions

* Elastic IP for stable server addressing
* Nginx reverse proxy
* Zero-downtime deployments
* Separate `deploy` user instead of `ubuntu`
* Health checks and rollback strategy
* Secrets rotation and SSH hardening

---

## Author

**Nikhil**

Software Engineer
Focused on backend systems, infrastructure, and understanding how software runs in production.
