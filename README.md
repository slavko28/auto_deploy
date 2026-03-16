[![Build & Deploy to Raspberry Pi](https://github.com/slavko28/auto_deploy/actions/workflows/deploy.yml/badge.svg)](https://github.com/slavko28/auto_deploy/actions/workflows/deploy.yml)


# Auto Deploy to Raspberry Pi via [Tailscale](https://tailscale.com/)

This project provides a complete CI/CD pipeline for deploying **Java Spring Boot** applications to a **Raspberry Pi 4 (ARM64)** using **GitHub Actions**, **Tailscale VPN**, and **Docker**.

### [GitHub Actions](https://github.com/features/actions) → [Tailscale VPN](https://tailscale.com/) → [Raspberry Pi 4](https://www.raspberrypi.com) → [Docker](https://www.docker.com)

## 🚀 Architecture
The deployment follows this secure workflow:
1. **Push:** Code is pushed to the GitHub repository.
2. **Build:** GitHub Actions builds the project JAR file.
3. **Tunnel:** A secure Tailscale VPN connection is established between GitHub Actions and the Raspberry Pi.
4. **Deploy:** The pipeline SSHs into the Pi, builds a native ARM64 Docker image, and restarts the container.

## 🛠️ Prerequisites
* **Hardware:** Raspberry Pi 4 (ARM64) running Raspberry Pi OS (64-bit).
* **Software:** Docker installed on the Pi.
* **Networking:** A [Tailscale](https://tailscale.com/) account for secure remote access.
* **Stack:** Java 17+ (Maven/Gradle) Spring Boot project.

## ⚙️ Setup Instructions

### 1. Raspberry Pi Configuration
* **Create a Deployer User:**
  ```bash
  sudo useradd -m deployer
  sudo usermod -aG docker deployer

* **SSH Setup:**
Generate an Ed25519 SSH key on your local machine. Add the public key to /home/deployer/.ssh/authorized_keys on the Pi and save the private key for GitHub Secrets.
* **Tailscale:**
Install Tailscale on the Pi and assign it the tag tag:pi in the Admin Console.
    ```bash
    curl -fsSL [https://tailscale.com/install.sh](https://tailscale.com/install.sh) | sh
    sudo tailscale up --advertise-tags=tag:pi

### 2. Tailscale ACL & OAuth
Update your Tailscale Access Control Policy to allow the CI runner to communicate with the Pi:
  * ACL: Grant tag:ci access to tag:pi on port 22.
  * OAuth: Create an [OAuth Client](https://tailscale.com/s/oauth-clients) with Devices:

    Write scope and the tag:ci tag.
   
  **Important:** Copy the Client Secret immediately; it is only shown once.
  
### 3. GitHub Secrets
Add the following variables to your GitHub Repository (Settings > Secrets and variables > Actions):

| Secret Name   | Description  |
|---------------|--------------|
|TAILSCALE_OAUTH_CLIENT_ID|OAuth Client ID from Tailscale|
|TAILSCALE_OAUTH_CLIENT_SECRET|OAuth Client Secret from Tailscale|
|DEPLOY_HOST|The Tailscale IP of your Raspberry Pi (100.x.x.x)|
|DEPLOY_USER|deployer|
|DEPLOY_SSH_KEY|The full private SSH key (including BEGIN/END lines)|

## 🐳 Dockerization

To ensure compatibility with the Raspberry Pi's ARM64 architecture, your Dockerfile must use a compatible base image.
In this case amazoncorretto
   ```bash
   # CRITICAL: amazoncorretto supports ARM64 (Raspberry Pi)
   FROM amazoncorretto:17-al2-jdk
   WORKDIR /app
   COPY *.jar app.jar
   EXPOSE 8080
   ENTRYPOINT ["java", "-jar", "app.jar"]
   ```
## 📈 Usage
The workflow is triggered automatically on every push to the main or dev_deploy branches. You can track the progress in the Actions tab of this repository.


