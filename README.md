# End-to-End CI/CD Pipeline for Automated Web Hosting on AWS

![GitHub Workflow Status](https://img.shields.io/badge/Pipeline-Passing-brightgreen?style=for-the-badge)
![Platform](https://img.shields.io/badge/Platform-AWS-orange?style=for-the-badge)
![Tools](https://img.shields.io/badge/Tools-Jenkins%20%7C%20Docker-blue?style=for-the-badge)
![Language](https://img.shields.io/badge/Language-Groovy%20%7C%20Shell-yellow?style=for-the-badge)

This repository documents the setup of a complete CI/CD pipeline that automates the deployment of a static website. By integrating **GitHub, Jenkins, Docker, and AWS EC2**, this project achieves a zero-touch deployment workflow where every `git push` to the main branch automatically updates the live website.

---

## 📋 Table of Contents

1.  [Project Overview](#-project-overview)
2.  [Architecture Diagram](#-architecture-diagram)
3.  [Core Features](#-core-features)
4.  [Technology Stack](#-technology-stack)
5.  [Prerequisites](#-prerequisites)
6.  [Step-by-Step Configuration Guide](#-step-by-step-configuration-guide)
    * [Part 1: AWS Infrastructure Setup](#part-1-aws-infrastructure-setup)
    * [Part 2: Jenkins Master Setup with Docker](#part-2-jenkins-master-setup-with-docker)
    * [Part 3: Configure EC2 as a Jenkins Agent Node](#part-3-configure-ec2-as-a-jenkins-agent-node)
    * [Part 4: Create the Jenkins Pipeline](#part-4-create-the-jenkins-pipeline)
7.  [How to Use](#-how-to-use)
8.  [Troubleshooting Common Issues](#-troubleshooting-common-issues)
9.  [Future Enhancements](#-future-enhancements)
10. [License](#-license)

---

## 🚀 Project Overview

The primary goal is to build a robust, automated pipeline that eliminates manual deployment tasks. The system monitors a GitHub repository and, upon detecting a change, securely fetches the latest code, transfers it to a web server, and makes the updates live. This project serves as a practical demonstration of modern DevOps principles.

**Live Site URL:** `http://<YOUR_EC2_ELASTIC_IP>`

---

## 📊 Architecture Diagram

The flow of the pipeline is as follows:

1.  **Developer** pushes new code to the **GitHub Repository**.
2.  A **GitHub Webhook** instantly triggers the **Jenkins Pipeline**.
3.  The **Jenkins Master** (running in a Docker container) orchestrates the job.
4.  It delegates the execution to the **Jenkins Agent** running on the **AWS EC2 Instance** via a secure SSH connection.
5.  The agent pulls the source code, copies the web files (`index.html`, `styles.css`) to the Apache webroot, and restarts the Apache service.
6.  The **user** can immediately see the updated website.

![Architecture Diagram](assets/architecture-diagram.png)

---

## ✨ Core Features

* **Zero-Touch Deployment:** Fully automated builds and deployments triggered by `git push`.
* **Infrastructure as Code (IaC):** The pipeline is defined in a `Jenkinsfile`, making it version-controlled, reproducible, and transparent.
* **Scalable Jenkins Architecture:** Utilizes a master/agent model, allowing the addition of more agents for different environments or tasks.
* **Secure by Design:** Employs SSH key-pair authentication for secure, passwordless communication between the master and agent.
* **Containerized & Portable:** Jenkins runs in a Docker container, making it easy to set up, backup, and migrate.

---

## 🛠️ Technology Stack

| Component           | Technology                                                                                                                                                             |
| ------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cloud Provider** | **AWS** (EC2, Elastic IP, Security Groups)                                                                                                                             |
| **CI/CD Tool** | **Jenkins** |
| **Containerization**| **Docker** |
| **Version Control** | **Git & GitHub** |
| **Web Server** | **Apache2** |
| **OS / Environment**| **Ubuntu 22.04 LTS** |
| **Scripting** | **Groovy** (Declarative Pipeline), **Shell Script** |

---

## ✅ Prerequisites

Before you begin, ensure you have the following:
* An **AWS Account** with permissions to create EC2 instances and security groups.
* **Git** installed on your local machine.
* **Docker Desktop** installed and running on your local machine.
* A **GitHub Account** and a new repository for this project.
* A code editor like **VS Code**.

---

## 🔧 Step-by-Step Configuration Guide

### Part 1: AWS Infrastructure Setup

1.  **Launch EC2 Instance:**
    * Navigate to the EC2 Dashboard in your AWS Console.
    * Launch a new instance with the following settings:
        * **AMI:** Ubuntu 22.04 LTS
        * **Instance Type:** `t2.micro` (Free Tier eligible)
        * **Key Pair:** Create a new key pair and download the `.pem` file. You will need this to SSH into your instance.

    ![EC2 Instance Launch](assets/ec2-launch.png)

2.  **Configure Security Group:**
    * Create a new security group with the following **inbound rules**:
        | Type         | Protocol | Port Range | Source        | Description                  |
        |--------------|----------|------------|---------------|------------------------------|
        | HTTP         | TCP      | 80         | `0.0.0.0/0`   | Allows web traffic           |
        | SSH          | TCP      | 22         | `My IP`       | Secure access for you        |
        | Custom TCP   | TCP      | 50000      | `0.0.0.0/0`   | For Jenkins Agent connection |

    ![Security Group Configuration](assets/security-group.png)

3.  **Allocate an Elastic IP:**
    * In the EC2 dashboard, go to "Elastic IPs".
    * Allocate a new address and associate it with your newly created EC2 instance. This provides a static public IP.

    ![Elastic IP Allocation](assets/elastic-ip.png)

4.  **Install Apache and Java on EC2:**
    * Connect to your EC2 instance using your `.pem` key:
        ```bash
        ssh -i /path/to/your-key.pem ubuntu@<YOUR_ELASTIC_IP>
        ```
    * Update the package manager and install Apache2 and Java (required for the Jenkins agent):
        ```bash
        sudo apt-get update -y
        sudo apt-get install -y apache2 openjdk-11-jre
        sudo systemctl start apache2
        sudo systemctl enable apache2
        ```

### Part 2: Jenkins Master Setup with Docker

1.  **Run Jenkins Docker Container:**
    * On your local machine, run the following command to start the Jenkins master:
        ```bash
        docker run -d --name jenkins-server \
          -p 8080:8080 -p 50000:50000 \
          -v jenkins_home:/var/jenkins_home \
          jenkins/jenkins:lts-jdk11
        ```

    ![Jenkins Docker Setup](assets/jenkins-docker.png)

2.  **Initial Jenkins Setup:**
    * Get the initial admin password from the container logs:
        ```bash
        docker exec jenkins-server cat /var/jenkins_home/secrets/initialAdminPassword
        ```
    * Open your browser and navigate to `http://localhost:8080`.
    * Paste the password, install suggested plugins, and create your admin user.

    ![Jenkins Initial Setup](assets/jenkins-initial-setup.png)

### Part 3: Configure EC2 as a Jenkins Agent Node

1.  **Generate SSH Keys in Jenkins Container:**
    * Open a shell inside your running Jenkins container:
        ```bash
        docker exec -it jenkins-server bash
        ```
    * Inside the container, generate an SSH key pair. Press Enter for all prompts:
        ```bash
        ssh-keygen -t rsa
        ```
    * Display and copy the public key:
        ```bash
        cat ~/.ssh/id_rsa.pub
        ```

    ![SSH Key Generation](assets/ssh-keys.png)

2.  **Authorize the Public Key on EC2:**
    * SSH back into your EC2 instance.
    * Append the public key you just copied to the `authorized_keys` file:
        ```bash
        echo "PASTE_YOUR_PUBLIC_KEY_HERE" >> ~/.ssh/authorized_keys
        ```
    * Set the correct permissions for the SSH directory and file:
        ```bash
        chmod 700 ~/.ssh
        chmod 600 ~/.ssh/authorized_keys
        ```

3.  **Add the Agent Node in Jenkins UI:**
    * Go to **Dashboard > Manage Jenkins > Manage Nodes and Clouds > New Node**.
    * **Node Name:** `aws-ec2-agent`
    * Select **Permanent Agent**.
    * **Remote root directory:** `/home/ubuntu/jenkins-agent`
    * **Launch method:** `Launch agents via SSH`.
    * **Host:** Your EC2 instance's Elastic IP.
    * **Credentials:** Click **Add > Jenkins**.
        * **Kind:** `SSH Username with private key`.
        * **Username:** `ubuntu`.
        * **Private Key:** Select `Enter directly` and paste the private key from the Jenkins container (`docker exec jenkins-server cat ~/.ssh/id_rsa`).
    * **Host Key Verification Strategy:** `Non-verifying Verification Strategy`.
    * Save and wait for the agent to connect. You should see a "Success" message in the node's logs.

    ![Jenkins Agent Configuration](assets/jenkins-agent-config.png)

### Part 4: Create the Jenkins Pipeline

1.  **Create a New Pipeline Job:**
    * In the Jenkins Dashboard, click **New Item**.
    * Enter a name (e.g., `Static-Site-Deployment`) and select **Pipeline**.

    ![Jenkins Pipeline Creation](assets/jenkins-pipeline-create.png)

2.  **Configure the Pipeline:**
    * In the project configuration page, scroll down to the **Pipeline** section.
    * **Definition:** `Pipeline script from SCM`.
    * **SCM:** `Git`.
    * **Repository URL:** Enter the HTTPS URL of your GitHub repository.
    * **Branch Specifier:** `*/main` or `*/master`.
    * **Script Path:** `Jenkinsfile` (this should be the name of the file in your repo).
    * Save the project.

    ![Jenkins Pipeline Configuration](assets/jenkins-pipeline-config.png)

---

## 🚀 How to Use

Once the setup is complete, the pipeline is ready to use:

1.  **Clone** this repository to your local machine.
2.  **Make a change** to `index.html` or `styles.css`.
3.  **Commit and push** the changes to the `main` branch:
    ```bash
    git add .
    git commit -m "feat: Updated website content"
    git push origin main
    ```
4.  Watch the pipeline automatically trigger and execute in your Jenkins dashboard. Within a minute, your changes will be live on your EC2 web server!

---

## 🔍 Troubleshooting Common Issues

* **Issue:** Jenkins agent fails to connect.
    * **Solution:** Double-check that the Security Group allows traffic on port 50000. Verify that the SSH public key was correctly added to `authorized_keys` on the EC2 instance and that file permissions (`700` for `.ssh`, `600` for `authorized_keys`) are correct.

* **Issue:** The pipeline fails at the deployment step with "Permission Denied".
    * **Solution:** The `ubuntu` user may not have permission to write to `/var/www/html`. SSH into the EC2 instance and run `sudo chown -R ubuntu:ubuntu /var/www/html` to grant ownership.

* **Issue:** Git push does not trigger the pipeline.
    * **Solution:** This requires a GitHub Webhook. Ensure your Jenkins server is publicly accessible. For local setups, use a tool like `ngrok` to expose your `localhost:8080` to the internet and use the `ngrok` URL in your GitHub webhook settings.

---

## 🔮 Future Enhancements

* **Infrastructure as Code with Terraform:** Use Terraform to provision the entire AWS infrastructure (EC2, Security Groups, Elastic IP) automatically.
* **Add a Testing Stage:** Incorporate a "Test" stage in the `Jenkinsfile` to run validation checks (e.g., HTML linting, link checking) before deployment.
* **Enable HTTPS:** Use Let's Encrypt and Certbot to add a free SSL certificate to the Apache server for secure HTTPS traffic.
* **Dynamic Environments:** Parameterize the Jenkins job to deploy to different environments (e.g., staging, production) based on the branch name.

---

## 📄 License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
