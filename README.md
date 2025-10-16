# End-to-End CI/CD Pipeline for Automated Web Hosting on AWS

![GitHub Workflow Status](https://img.shields.io/badge/Pipeline-Passing-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-AWS-orange?style=for-the-badge)
![Tools](https://img.shields.io/badge/Tools-Jenkins%20%7C%20Docker-blue?style=for-the-badge)
![Language](https://img.shields.io/badge/Language-Groovy%20%7C%20Shell-yellow?style=for-the-badge)

This repository documents a complete CI/CD pipeline that automates static website deployment. By integrating **GitHub, Jenkins, Docker, and AWS EC2**, this project achieves zero-touch deployment where every `git push` to the main branch automatically updates the live website.

---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [Architecture](#-architecture)
3. [Core Features](#-core-features)
4. [Technology Stack](#-technology-stack)
5. [Prerequisites](#-prerequisites)
6. [Configuration Guide](#-configuration-guide)
   * [Part 1: AWS Infrastructure Setup](#part-1-aws-infrastructure-setup)
   * [Part 2: Jenkins Master with Docker](#part-2-jenkins-master-setup-with-docker)
   * [Part 3: Configure Jenkins Agent](#part-3-configure-ec2-as-jenkins-agent)
   * [Part 4: Create Jenkins Pipeline](#part-4-create-the-jenkins-pipeline)
7. [Usage](#-usage)
8. [Troubleshooting](#-troubleshooting)
9. [Future Enhancements](#-future-enhancements)
10. [License](#-license)

---

## 🚀 Project Overview

This project demonstrates modern DevOps principles by building a robust, automated pipeline that eliminates manual deployment tasks. The system monitors a GitHub repository and, upon detecting changes, securely fetches the latest code, transfers it to a web server, and deploys updates automatically.

**Live Site URL:** `http://<your_ec2_elastic_ip>`

---

## 📊 Architecture

### Pipeline Flow

1. **Developer** pushes code to the **GitHub Repository**
2. **GitHub Webhook** triggers the **Jenkins Pipeline**
3. **Jenkins Master** (Docker container) orchestrates the job
4. **Jenkins Agent** (EC2 instance) clones the repo
5. Code is deployed to `/var/www/html` on the web server
6. **Apache** serves the updated static site

![CI/CD Architecture Overview](assets/screenshot-2025-10-16-18-25-41.png)
*Complete CI/CD pipeline architecture showing GitHub, Jenkins, Docker, and AWS EC2 integration*

---

## ✨ Core Features

- **Automated Deployment**: Push to GitHub → Live in minutes
- **Master-Agent Architecture**: Jenkins master in Docker, agent on EC2
- **Secure Communication**: SSH key-based authentication
- **Webhook Integration**: Instant pipeline triggers
- **Scalable Infrastructure**: Easy to extend with multiple agents
- **Production-Ready**: Apache web server with proper permissions

---

## 🛠 Technology Stack

| Component | Technology |
|-----------|------------|
| **Version Control** | GitHub |
| **CI/CD Server** | Jenkins |
| **Containerization** | Docker |
| **Cloud Provider** | AWS EC2 |
| **Web Server** | Apache2 |
| **Scripting** | Groovy (Jenkinsfile), Bash |

---

## 📦 Prerequisites

- **AWS Account** with EC2 access
- **GitHub Account** with repository admin rights
- **Local Machine** with Docker installed
- **Basic Knowledge** of Linux, Git, and Docker
- **Network Access** for webhook communication

---

## 📖 Configuration Guide

### Part 1: AWS Infrastructure Setup

#### 1.1 Launch EC2 Instance

1. Navigate to **AWS EC2 Console** → **Launch Instance**
2. Select **Ubuntu Server 22.04 LTS (HVM)**
3. Instance type: **t2.micro** (free tier eligible)
4. Create or select a key pair for SSH access
5. Launch the instance

#### 1.2 Configure Security Group

Add the following inbound rules:

| Type | Port | Source | Purpose |
|------|------|--------|----------|
| SSH | 22 | Your IP | SSH access |
| HTTP | 80 | 0.0.0.0/0 | Web traffic |
| Custom TCP | 8080 | Your IP | Jenkins UI |
| Custom TCP | 50000 | Your IP | Jenkins agent communication |

![Security Group Configuration](assets/screenshot-2025-10-16-18-26-06.png)
*Security group rules for Jenkins and web server access*

#### 1.3 Allocate Elastic IP

1. Go to **EC2** → **Elastic IPs** → **Allocate Elastic IP**
2. Associate the Elastic IP with your EC2 instance
3. Note the public IP address for later use

![Elastic IP Allocation](assets/screenshot-2025-10-16-18-26-26.png)
*Elastic IP associated with EC2 instance for persistent addressing*

#### 1.4 Install Apache Web Server

SSH into your EC2 instance:

```bash
ssh -i your-key.pem ubuntu@<your_elastic_ip>

# Update packages
sudo apt update && sudo apt upgrade -y

# Install Apache
sudo apt install apache2 -y

# Verify Apache is running
sudo systemctl status apache2

# Set permissions
sudo chown -R ubuntu:ubuntu /var/www/html
```

---

### Part 2: Jenkins Master Setup with Docker

#### 2.1 Install Docker (Local Machine)

```bash
# For Ubuntu/Debian
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

# Add user to docker group
sudo usermod -aG docker $USER
newgrp docker
```

#### 2.2 Run Jenkins in Docker

```bash
# Pull Jenkins LTS image
docker pull jenkins/jenkins:lts

# Create volume for persistence
docker volume create jenkins_home

# Run Jenkins container
docker run -d \
  --name jenkins-master \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
```

#### 2.3 Jenkins Initial Configuration

1. Access Jenkins at `http://localhost:8080`
2. Enter the initial admin password
3. Install suggested plugins
4. Create an admin user

![Jenkins Initial Setup](assets/screenshot-2025-10-16-18-26-33.png)
*Jenkins dashboard showing initial configuration and plugin installation*

---

### Part 3: Configure EC2 as Jenkins Agent

#### 3.1 Generate SSH Key Pair

On your local machine (inside Jenkins container):

```bash
# Access Jenkins container
docker exec -it jenkins-master bash

# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -C "jenkins-agent" -f ~/.ssh/jenkins_agent_key -N ""

# Display public key (copy this)
cat ~/.ssh/jenkins_agent_key.pub

# Display private key (needed for Jenkins credentials)
cat ~/.ssh/jenkins_agent_key
```

#### 3.2 Configure EC2 for Agent Connection

SSH into your EC2 instance:

```bash
# Add Jenkins public key to authorized_keys
echo "<paste_public_key_here>" >> ~/.ssh/authorized_keys

# Set correct permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Install Java (required for Jenkins agent)
sudo apt install openjdk-17-jre -y

# Verify Java installation
java -version
```

#### 3.3 Add Agent Node in Jenkins

1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Nodes**
2. Click **New Node**
3. Configure:
   - **Name**: `ec2-web-agent`
   - **Remote root directory**: `/home/ubuntu/jenkins`
   - **Labels**: `web-deployment`
   - **Launch method**: Launch agents via SSH
   - **Host**: Your EC2 Elastic IP
   - **Credentials**: Add SSH private key
   - **Host Key Verification Strategy**: Non verifying

4. Save and verify the agent connects successfully

![Jenkins Agent Configuration](assets/screenshot-2025-10-16-18-26-47.png)
*Jenkins agent node configuration showing EC2 connection details*

---

### Part 4: Create the Jenkins Pipeline

#### 4.1 Create New Pipeline Job

1. Go to **Jenkins Dashboard** → **New Item**
2. Enter name: `web-deployment-pipeline`
3. Select **Pipeline** project type
4. Click **OK**

![Jenkins Pipeline Creation](assets/screenshot-2025-10-16-18-27-04.png)
*Creating a new Jenkins pipeline for automated deployment*

#### 4.2 Configure Pipeline

1. Under **Build Triggers**, enable:
   - ☑️ **GitHub hook trigger for GITScm polling**

2. Under **Pipeline**, select:
   - **Definition**: Pipeline script from SCM
   - **SCM**: Git
   - **Repository URL**: `https://github.com/<your-username>/ci-cd-aws-static-hosting.git`
   - **Branch**: `*/main`
   - **Script Path**: `Jenkinsfile`

![Jenkins Pipeline Configuration](assets/screenshot-2025-10-16-18-34-25.png)
*Pipeline configuration with GitHub integration and Jenkinsfile path*

#### 4.3 Create Jenkinsfile

Create a `Jenkinsfile` in your repository root:

```groovy
pipeline {
    agent {
        label 'web-deployment'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }
        
        stage('Deploy to Web Server') {
            steps {
                echo 'Deploying website...'
                sh '''
                    # Remove old files
                    sudo rm -rf /var/www/html/*
                    
                    # Copy new files
                    sudo cp -r * /var/www/html/
                    
                    # Set permissions
                    sudo chown -R www-data:www-data /var/www/html
                    sudo chmod -R 755 /var/www/html
                    
                    echo "Deployment completed successfully!"
                '''
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline executed successfully! Website is live.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
```

#### 4.4 Configure GitHub Webhook

1. Go to your GitHub repository → **Settings** → **Webhooks**
2. Click **Add webhook**
3. Configure:
   - **Payload URL**: `http://<jenkins_ip>:8080/github-webhook/`
   - **Content type**: `application/json`
   - **Events**: Just the push event
4. Save the webhook

---

## 🎯 Usage

1. **Clone** the repository:
   ```bash
   git clone https://github.com/<your-username>/ci-cd-aws-static-hosting.git
   cd ci-cd-aws-static-hosting
   ```

2. **Make changes** to `index.html` or `styles.css`

3. **Commit and push**:
   ```bash
   git add .
   git commit -m "feat: Update website content"
   git push origin main
   ```

4. **Watch** the pipeline auto-trigger in Jenkins and deploy your changes live!

---

## 🔍 Troubleshooting

### Jenkins Agent Connection Issues

**Problem:** Agent fails to connect

**Solutions:**
- Verify Security Group allows port 50000
- Check SSH key permissions (700 for `.ssh`, 600 for `authorized_keys`)
- Ensure public key is correctly added to EC2 `authorized_keys`
- Test SSH connection manually: `ssh -i jenkins_agent_key ubuntu@<ec2_ip>`

### Permission Denied During Deployment

**Problem:** Pipeline fails with "Permission Denied" error

**Solutions:**
- Grant ownership: `sudo chown -R ubuntu:ubuntu /var/www/html`
- Update Jenkinsfile to use `sudo` for copy operations
- Verify ubuntu user has sudo privileges without password

### Webhook Not Triggering Pipeline

**Problem:** Pushing to GitHub doesn't trigger the pipeline

**Solutions:**
- Ensure Jenkins is publicly accessible (use ngrok for local testing)
- Verify webhook URL is correct: `http://<jenkins_ip>:8080/github-webhook/`
- Check GitHub webhook delivery status for errors
- Enable "GitHub hook trigger" in Jenkins job configuration

### Apache Not Serving Updated Content

**Problem:** Changes deployed but old content still visible

**Solutions:**
- Clear browser cache (Ctrl+Shift+R)
- Restart Apache: `sudo systemctl restart apache2`
- Verify file permissions: `ls -la /var/www/html`
- Check Apache error logs: `sudo tail -f /var/log/apache2/error.log`

---

## 🔮 Future Enhancements

- **Infrastructure as Code**: Use Terraform to automate AWS resource provisioning
- **Automated Testing**: Add HTML validation and link checking stages
- **SSL/TLS Support**: Implement Let's Encrypt for HTTPS
- **Multi-Environment Deployment**: Support staging and production environments
- **Monitoring & Alerts**: Integrate CloudWatch for pipeline monitoring
- **Docker-based Deployment**: Containerize the web application
- **Blue-Green Deployment**: Implement zero-downtime deployment strategy

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**Built with ❤️ for DevOps automation**
