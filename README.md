# Azure VM Continuous Deployment with GitHub Actions

This repository features an automated CI/CD pipeline using **GitHub Actions**. Every time code is pushed to the `main` branch, the workflow triggers, connects to an active **Azure VM** via SSH, pulls the latest public image from **Docker Hub**, and deploys the container.

---

## 🛠️ Prerequisites

Before the workflow can run successfully, ensure you have configured the following:

### 1. Azure VM Setup
* **Docker Installed:** Docker must be running on your VM (e.g., `sudo apt update && sudo apt install docker.io -y`).
* **Networking/Firewall:** The Azure Network Security Group (NSG) must allow inbound traffic on your application's public port (e.g., port `80`) and allow SSH access (port `22`) from GitHub's IP runners or all sources.
* **SSH Access:** You must have a valid SSH Private Key associated with the VM's admin user.

### 2. Docker Hub
* A repository hosting your **public container image** (e.g., `your-dockerhub-username/your-image-name`).
* *Note: Ensure your separate build pipeline updates this public image before this deployment workflow triggers.*

---

## 🔐 GitHub Secrets Configuration

To securely connect to your Azure infrastructure, you must add the following **Repository Secrets** in GitHub (**Settings** > **Secrets and variables** > **Actions** > **New repository secret**):

| Secret Name | Description | Example Value |
| :--- | :--- | :--- |
| `VM_HOST` | The public IP address or DNS label of your Azure VM. | `52.168.1.45` |
| `VM_USERNAME` | The admin login user for your VM. | `azureuser` |
| `VM_SSH_KEY` | The entire contents of your private SSH key (`.pem` or `id_rsa`). | `-----BEGIN OPENSSH PRIVATE KEY-----...` |

---

## 📦 Workflow Configuration

The deployment lifecycle is handled by `.github/workflows/deploy.yml`. 

### What it does:
1. **Triggers** automatically on any push to the `main` branch.
2. Uses the `appleboy/ssh-action` action to open a secure shell session on the VM.
3. Stops and removes the active container (`my-app`) if it exists to avoid port conflicts.
4. Pulls the latest copy of your public image from Docker Hub.
5. Launches the new container mapping host port `80` to container port `8080`.
6. Prunes dangling Docker images to preserve VM disk space.

---

## 🚀 Triggering Deployment

To deploy your application:
1. Push your updated image to Docker Hub.
2. Push your code changes or merge a Pull Request into the `main` branch:
   ```bash
   git add .
   git commit -m "deploy: update container application"
   git push origin main
   ```
3. Monitor execution progress under the **Actions** tab of your GitHub repository.
