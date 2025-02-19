# 🎬 Netflix Clone - DevSecOps Project

## 🚀 Project Overview
This project follows a **DevSecOps** approach to deploy and secure a Netflix clone application using **Jenkins, Docker, Kubernetes, SonarQube, and Trivy**. It includes CI/CD, security scanning, monitoring, and deployment on AWS.

![Screenshot](images/Capture.PNG)

---

## 🔹 DevOps Section
### **Step 1 — Launch AWS Infrastructure**
- Launch an **Ubuntu 22.04 T2 Large** EC2 instance.
- Install **Jenkins**, **Docker**, and required dependencies.

### **Step 2 — CI/CD Pipeline with Jenkins**
- Install required Jenkins plugins:  
  - **JDK**, **SonarQube Scanner**, **NodeJS**, **OWASP Dependency Check**
- Configure a **Declarative Pipeline** in Jenkins.
- Integrate Jenkins with GitHub.

### **Step 3 — Docker & Kubernetes Deployment**
- Build a **Docker Image** for the Netflix Clone.
- Push the image to **DockerHub**.
- Deploy the image using Docker.
- Set up **Kubernetes Master & Worker nodes** on Ubuntu 20.04.
- Deploy the application using **K8s manifests**.

### **Step 4 — Monitoring & Logging**
- Install **Prometheus & Grafana** on a monitoring server.
- Install **Prometheus Plugin** in Jenkins.
- Set up **Grafana dashboards** to monitor Jenkins & Kubernetes.

### **Step 5 — Notifications & Cleanup**
- Configure **Email Integration** in Jenkins.
- Automate EC2 **termination** after the project completion.

---

## 🔹 SecOps Section
### **Step 6 — Security Scanning & Code Quality**
- Install and configure **SonarQube** (Docker Container).
- Perform **static code analysis** on the Netflix Clone repository.

### **Step 7 — Vulnerability Scanning**
- Install **Trivy** for container security scanning.
- Scan the Docker images before deployment.

### **Step 8 — Dependency Scanning**
- Install **OWASP Dependency Check** Plugin in Jenkins.
- Scan project dependencies for security vulnerabilities.

---

## 🎯 Final Outcome
Once all steps are completed, the **Netflix Clone App** will be accessible in the browser, running on a secure and optimized DevSecOps pipeline. 🚀

