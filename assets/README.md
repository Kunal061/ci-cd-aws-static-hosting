# Assets Folder

This folder contains documentation screenshots for the CI/CD AWS Static Hosting project. Each screenshot corresponds to specific steps in the setup and configuration process.

## Screenshot Reference Guide

### screenshot-2025-10-16-18-25-41.png
**Topic:** CI/CD Pipeline Architecture Overview  
**Description:** Complete CI/CD architecture diagram showing the integration between GitHub, Jenkins (Docker), EC2 agent, and Apache web server. Illustrates the automated deployment workflow from code push to live website.

**Referenced in README as:** Architecture Diagram  
**Section:** 📊 Architecture

---

### screenshot-2025-10-16-18-26-06.png
**Topic:** AWS Security Group Configuration  
**Description:** AWS EC2 Security Group inbound rules configuration showing ports for SSH (22), HTTP (80), Jenkins UI (8080), and Jenkins agent communication (50000). Essential for enabling proper network access to the Jenkins master and web server.

**Referenced in README as:** Security Group Configuration  
**Section:** Part 1: AWS Infrastructure Setup → 1.2 Configure Security Group

---

### screenshot-2025-10-16-18-26-26.png
**Topic:** AWS Elastic IP Allocation  
**Description:** AWS Elastic IP allocation and association with EC2 instance. Shows the persistent public IP address configuration that ensures the Jenkins master and web server remain accessible even after instance restarts.

**Referenced in README as:** Elastic IP Allocation  
**Section:** Part 1: AWS Infrastructure Setup → 1.3 Allocate Elastic IP

---

### screenshot-2025-10-16-18-26-33.png
**Topic:** Jenkins Initial Setup and Dashboard  
**Description:** Jenkins initial configuration dashboard showing the setup wizard, plugin installation interface, and admin user creation. Displays the Jenkins home page after successful Docker deployment.

**Referenced in README as:** Jenkins Initial Setup  
**Section:** Part 2: Jenkins Master Setup with Docker → 2.3 Jenkins Initial Configuration

---

### screenshot-2025-10-16-18-26-47.png
**Topic:** Jenkins Agent Node Configuration  
**Description:** Jenkins agent node setup screen showing the configuration for connecting the EC2 instance as a Jenkins build agent. Includes settings for SSH connection, remote root directory, labels, and launch method.

**Referenced in README as:** Jenkins Agent Configuration  
**Section:** Part 3: Configure EC2 as Jenkins Agent → 3.3 Add Agent Node in Jenkins

---

### screenshot-2025-10-16-18-27-04.png
**Topic:** Jenkins Pipeline Job Creation  
**Description:** Jenkins interface for creating a new pipeline job. Shows the "New Item" page where the pipeline project is configured with name, type, and basic settings.

**Referenced in README as:** Jenkins Pipeline Creation  
**Section:** Part 4: Create the Jenkins Pipeline → 4.1 Create New Pipeline Job

---

### screenshot-2025-10-16-18-34-25.png
**Topic:** Jenkins Pipeline Configuration with GitHub Integration  
**Description:** Complete Jenkins pipeline configuration showing GitHub webhook triggers, SCM settings, repository URL, branch specification, and Jenkinsfile path. Demonstrates the connection between GitHub repository and Jenkins automation.

**Referenced in README as:** Jenkins Pipeline Configuration  
**Section:** Part 4: Create the Jenkins Pipeline → 4.2 Configure Pipeline

---

## How Screenshots Are Used

These screenshots are embedded in the main project README to:
- **Visual Documentation**: Provide step-by-step visual guidance for setup
- **Verification**: Help users confirm they're on the right screen during configuration
- **Troubleshooting**: Serve as reference points when diagnosing issues
- **Learning**: Illustrate the complete CI/CD pipeline architecture

## Screenshot Naming Convention

Screenshots follow the pattern: `screenshot-YYYY-MM-DD-HH-MM-SS.png`  
This timestamp-based naming ensures chronological ordering and prevents naming conflicts.

## Maintenance Notes

- Screenshots should be updated if UI changes significantly
- Each screenshot should be under 500KB for optimal loading
- PNG format is preferred for clarity and text readability
- All screenshots are referenced in the main README.md using relative paths
