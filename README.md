# End-to-End CI/CD Pipeline using Jenkins, Maven & Docker (AWS)

This project demonstrates a complete **CI/CD pipeline implementation** using **Jenkins, Maven, Docker, and AWS EC2**.  
The pipeline automatically builds a Java web application, packages it as a WAR artifact, deploys it to a Docker container, verifies application health, and performs automatic rollback if deployment fails.

---

## 🚀 Features

- Automated **CI/CD pipeline using Jenkins**
- **Maven-based build automation**
- **Docker containerized deployments**
- **Artifact versioning (.war files)**
- **Docker image versioning using Jenkins build numbers**
- **Automated deployment with container replacement**
- **Health check validation after deployment**
- **Automatic rollback to previous stable container**
- **GitHub webhook integration for commit-to-deploy automation**

---

## 🏗 Architecture

The system consists of **two AWS EC2 instances**:

### Jenkins Server
Responsible for CI/CD orchestration.

Installed tools:
- Jenkins
- Java
- Maven
- Git
- Maven Integration Plugin
- SSH Agent Plugin

Responsibilities:
- Pull source code from GitHub
- Build the application using Maven
- Generate WAR artifact
- Transfer artifact to Docker host
- Trigger deployment via SSH

---

### Docker Host
Responsible for containerization and runtime deployment.

Installed tools:
- Docker
- Open inbound port **8086**

Responsibilities:
- Receive WAR artifact from Jenkins
- Maintain artifact history (`war_hist`)
- Build Docker image
- Deploy container
- Perform health checks
- Handle rollback if needed

---

## ⚙️ Pipeline Workflow

1. Developer pushes code to GitHub
2. GitHub webhook triggers Jenkins pipeline
3. Jenkins pulls the latest code
4. Maven builds the project and generates `.war`
5. Jenkins transfers artifact to Docker host via SSH
6. Artifact stored with **timestamp + build number**
7. Docker image built with **version tag**
8. Existing container stopped and replaced
9. Health check verifies deployment
10. If unhealthy → **automatic rollback to previous image**

---

## 📂 Project Structure
<img width="1269" height="1030" alt="Jenkins-CICD-Diagram" src="https://github.com/user-attachments/assets/4a65bc12-20d0-49e0-b380-d18976a193ce" />

