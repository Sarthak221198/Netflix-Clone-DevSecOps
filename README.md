# 🎬 Netflix Clone - DevSecOps Project

## 🚀 Project Overview
This project follows a **DevSecOps** approach to deploy and secure a Netflix clone application using **Jenkins, Docker, Kubernetes, SonarQube, and Trivy**. It includes CI/CD, security scanning, monitoring, and deployment on AWS.

## Architecture Overview

![Screenshot](images/Capture.PNG)

---

## 🔹 DevOps Section
### **Step 1 — Launch AWS Infrastructure**
- Launch an **Ubuntu 22.04 T2 Large**  with an elastic IP associated with the VM
- Install **Jenkins**, **Docker**, and required dependencies.
```sh
vi jenkins.sh

#!/bin/bash
sudo apt update -y
#sudo apt upgrade -y
wget -O - https://packages.adoptium.net/artifactory/api/gpg/key/public | tee /etc/apt/keyrings/adoptium.asc
echo "deb [signed-by=/etc/apt/keyrings/adoptium.asc] https://packages.adoptium.net/artifactory/deb $(awk -F= '/^VERSION_CODENAME/{print$2}' /etc/os-release) main" | tee /etc/apt/sources.list.d/adoptium.list
sudo apt update -y
sudo apt install temurin-17-jdk -y
/usr/bin/java --version
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
                  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
                  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
                              /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo systemctl start jenkins
sudo systemctl status jenkins

sudo chmod 777 jenkins.sh
./jenkins.sh    # this will installl jenkins

```

- After installing Jenkins, update your AWS EC2 Security Group to allow inbound traffic on port 8080, as      Jenkins operates on this port.
- Access the application by entering your EC2 Public IP Address followed by port 8080 (<EC2 Public IP Address:8080>).

```sh
vi docker.sh

sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock

```
- After installing Docker, deploy a SonarQube container (ensure port 9000 is open in the security group).

```sh
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

```


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

