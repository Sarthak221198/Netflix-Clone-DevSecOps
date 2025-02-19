# 🎬 Netflix Clone - DevSecOps Project

## 🚀 Project Overview
This project follows a **DevSecOps** approach to deploy and secure a Netflix clone application using **Jenkins, Docker, Kubernetes, SonarQube, and Trivy**. It includes CI/CD, security scanning, monitoring, and deployment on AWS.

## Architecture Overview

![Screenshot](images/Capture.PNG)

---

## 🔹 Infra Build and tools prerequisites

## Step 1 — Launch AWS Infrastructure
- Launch an **Ubuntu 22.04 T2 Large**  with an elastic IP associated with the VM (to have the same public IP incase of mutiple restarts)

## Step 2 — Installtion of Jenkins, docker and trivy

**Jenkins**
 is an open-source automation server used for CI/CD (Continuous Integration/Continuous Deployment). It automates the build, test, and deployment process of software development and integrates with various tools like Git, Docker, Kubernetes, and Trivy.

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
- Access the application by entering your EC2 Public IP Address followed by port 8080.
```sh

 http://<<EC2-PublicIP>>:8080/

```
- Jenkins Dashboard view

![Screenshot](images/image.png)

**Docker**
 is a containerization platform that allows developers to package applications and dependencies into lightweight, portable containers. It enables efficient deployment, scaling, and management of applications in different environments.

```sh
vi docker.sh

sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER   #my case is ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock

```
**SonarQube**
 is a continuous code quality and security analysis tool that detects bugs, vulnerabilities, and code smells in source code. It integrates with CI/CD pipelines and supports multiple programming languages.

- After installing Docker, deploy a SonarQube container (ensure port 9000 is open in the security group).

```sh
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

```
- Sonarqube Dashboard view

![Screenshot](images/image3.png)

**Trivy** 
 is an open-source vulnerability scanner for containers, file systems, and code repositories. It detects vulnerabilities, misconfigurations, and exposed secrets. It is widely used in DevOps pipelines for security compliance.

```sh
vi trivy.sh

sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

```

## Step 3 — Obtain the TMDB API Key

To integrate with The Movie Database (TMDB) and access movie-related data, you need an API key. The TMDB API allows developers to fetch movie details, ratings, images, and more for applications. Follow the steps below to generate your API key:

- Use a VPN (For Users in India) – Since TMDB is currently inaccessible in India, enable a VPN and connect to a different region where the website is available.
- Visit the TMDB Website – Open a web browser and navigate to TMDB (The Movie Database).
- Sign In or Register – Click on "Login" and sign in to your account. If you don’t have one, create a new account.
- Access API Settings – Once logged in, go to your profile settings by clicking on your profile icon and selecting “Settings.”
- Generate an API Key – In the left-side menu, click on “API” and choose the option to create a new API key.
- Accept Terms & Provide Details – Agree to the terms and conditions, fill in the required details, and submit your request.
- Retrieve Your API Key – Once approved, your unique API key will be generated and displayed.

Why is the TMDB API Key Needed?
- The API key acts as an authentication mechanism that allows your application to access TMDB’s vast database of movies, TV shows, and related metadata. It enables fetching movie details, searching for films, displaying posters, and integrating movie-related content into your project.

## 🔹 Monitoring Tools installation

## Step 1 — Installing Prometheus
- Launch an **Ubuntu 22.04 T2 Large**  with an elastic IP associated with the VM (to have the same public IP incase of mutiple restarts)

**Prometheus** 
 is an open-source monitoring and alerting tool designed for recording real-time metrics in a time-series database. It is widely used for infrastructure monitoring, particularly in cloud environments and Kubernetes clusters.

To ensure Prometheus runs efficiently as a background service, we will:

- Create a dedicated system user for Prometheus.
- Download and configure Prometheus.
- Set up a systemd service to manage Prometheus automatically.

To run Prometheus with restricted privileges, we create a dedicated Linux user:

```sh
sudo useradd --system --no-create-home --shell /bin/false prometheus
```
This prevents Prometheus from logging into the system and enhances security.

Extract Prometheus files, move them, and create directories:

```sh
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
```

To ensure Prometheus has the correct permissions, set ownership for the configuration and data directories:

```sh
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

To manage Prometheus as a background service, we define a systemd unit file:

```sh
sudo nano /etc/systemd/system/prometheus.service

[Unit]
# Description of the service
Description=Prometheus  

# Ensures Prometheus starts after the network is available
Wants=network-online.target  
After=network-online.target  

# Service restart settings
StartLimitIntervalSec=500  
StartLimitBurst=5  

[Service]
# Run Prometheus as the prometheus user
User=prometheus  
Group=prometheus  

# Run the service as a simple process
Type=simple  

# Restart settings in case of failure
Restart=on-failure  
RestartSec=5s  

# Command to start Prometheus with required configurations
ExecStart=/usr/local/bin/prometheus \  
  --config.file=/etc/prometheus/prometheus.yml \  # Specify the configuration file
  --storage.tsdb.path=/data \  # Define the directory to store time-series data
  --web.console.templates=/etc/prometheus/consoles \  # Location for web console templates
  --web.console.libraries=/etc/prometheus/console_libraries \  # Console libraries
  --web.listen-address=0.0.0.0:9090 \  # Listen on all network interfaces on port 9090
  --web.enable-lifecycle  # Enable API-based management  

[Install]
# Ensure the service starts on boot
WantedBy=multi-user.target  

# Enable Prometheus to start on system boot
sudo systemctl enable prometheus  

# Start the Prometheus service
sudo systemctl start prometheus  

# Check the status of the Prometheus service
sudo systemctl status prometheus  

```
Access Prometheus in a web browser using the server's IP and port 9090:


- Prometheus Dashboard view

We can see the local host present as the data source which is up and running on port 9090

![Screenshot](images/images4.png)

## Step 2 — Installing Node Exporter

**Node Exporter** 

 is an essential component of Prometheus-based monitoring. It is a lightweight agent that collects and exposes system-level metrics from a Linux server, making them available for Prometheus to scrape and analyze.

```sh
# Create a dedicated system user for Node Exporter without a home directory or login shell
sudo useradd --system --no-create-home --shell /bin/false node_exporter  

# Download the latest Node Exporter package
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz  

# Extract the downloaded archive
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz  

# Move the Node Exporter binary to a system-wide location
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/  

# Remove the extracted files to clean up unnecessary data
rm -rf node_exporter*  

# Create a systemd service file to manage Node Exporter as a system service
sudo nano /etc/systemd/system/node_exporter.service  

```
## Step 3 — Integrating Jenkins and NodeExporter with Prometheus

Integrating Jenkins with Prometheus to monitor the CI/CD pipeline.

To enable Prometheus to scrape metrics from both Node Exporter and Jenkins, update the prometheus.yml configuration file as shown below:

```sh
sudo cat /etc/prometheus/prometheus.yml

# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node_export"
    static_configs:
      - targets: ["localhost:9100"]
  - job_name: "jenkins"
    metrics_path: "/prometheus"
    static_configs:
      - targets: ["<your-jenkins-ip>:8080"]

```
Run the following command to check if the configuration file is correctly formatted:

```sh
promtool check config /etc/prometheus/prometheus.yml
```

Instead of restarting the Prometheus service, apply the new configuration dynamically:

```sh
curl -X POST http://localhost:9090/-/reload
```

Installing the Prometheus plugin in Jenkins is required to enable metric scraping by Prometheus.

![Screenshot](images/images5.png)

We can verify that the targets are healthy and visible on the Prometheus application

```sh
http://<ip>:9090/targets
```
![Screenshot](images/images6.PNG)

## Step 2 — Installing Grafana

**Grafana** 

 is essential for visualizing and analyzing metrics collected by Prometheus from Jenkins and Node Exporter. While Prometheus efficiently gathers and stores time-series data, its built-in UI is limited in terms of visualization. Grafana enhances monitoring by providing interactive dashboards, real-time graphs, and alerts, making it easier to track system performance. It allows users to customize views, correlate metrics, and receive notifications based on predefined thresholds. By integrating Grafana into our setup, we create a complete monitoring solution where Prometheus handles data collection, and Grafana presents that data in an intuitive and insightful manner for better decision-making and troubleshooting.


``` sh

# Install Dependencies
First, ensure that all necessary dependencies are installed:

sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common

# Add the GPG Key
Add the GPG key for Grafana:

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

# Add Grafana Repository
Add the repository for Grafana stable releases:

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

# Update and Install Grafana
Update the package list and install Grafana:

sudo apt-get update
sudo apt-get -y install grafana

# Enable and Start Grafana Service
To automatically start Grafana after a reboot, enable the service:

sudo systemctl enable grafana-server

Then, start Grafana:

sudo systemctl start grafana-server

# Check Grafana Status
Verify the status of the Grafana service to ensure it's running correctly:

sudo systemctl status grafana-server

# Step 7: Access Grafana Web Interface
Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. Example:

http://<your-server-ip>:3000

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."

# Change the Default Password
When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.

# Add Prometheus Data Source
To visualize metrics, add a data source:

1. Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.
2. Select "Data Sources."
3. Click on the "Add data source" button.
4. Choose "Prometheus" as the data source type.
5. In the "HTTP" section:
   - Set the "URL" to http://localhost:9090 (assuming Prometheus is running on the same server).
6. Click the "Save & Test" button to ensure the data source is working.


# Import a Dashboard
To make it easier to view metrics, you can import a pre-configured dashboard:

1. Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.
2. Select "Dashboard."
3. Click on the "Import" dashboard option.
4. Enter the dashboard code you want to import (e.g., code 1860).
5. Click the "Load" button.
6. Select the data source you added (Prometheus) from the dropdown.
7. Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.

# Configure Prometheus Plugin Integration
Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

```

![Screenshot](images/images7.png)

![Screenshot](images/images8.png)


- Prometheus server monitoring over grafana

![Screenshot](images/images9.png)

- Jenkins Job monitoring over grafana

![Screenshot](images/images10.png)

## Step 2 — Email Integration With Jenkins and Plugin Setup

Setting up email notifications in Jenkins for pipeline job status

- Install Email Extension Plugin in Jenkins

![Screenshot](images/images11.png)

- Go to your Gmail and click on your profile and generate app password

- Once the plugin is installed in Jenkins, click on manage Jenkins –> configure system there under the E-mail Notification section configure the details as shown in the below image

![Screenshot](images/images12.PNG)

Click on Manage Jenkins–> credentials and add your mail username and generated app password

![Screenshot](images/images13.png)

- Now under the Extended E-mail Notification section configure the details as shown in the below images

![Screenshot](images/images14.png)

![Screenshot](images/images15.png)

Now, we will receive notifications for every successful and failed build in Jenkins via Gmail, ensuring effective 24/7 monitoring.

- The following pipeline job will ensure that we receive an email with the Trivy FS image scan details, including the attached report.

```sh
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'sarthakmamgain44@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
```
## 🔹 Plugins Tools installation on Jenkins like JDK, Sonarqube Scanner, NodeJs, OWASP Dependency Check

## Step 1 Install below plugins

- SonarQube Scanner (Install without restart)
- NodeJs Plugin (Install Without restart)
- Eclipse Temurin Installer (Install without restart)

## Step 2 Configure Java and Nodejs in Global Tool Configuration

- Goto Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

![Screenshot](images/images16.png)

![Screenshot](images/images17.png)

## Step 3 Configure Sonar Server in Manage Jenkins

- Grab the Public IP Address of your EC2 Instance, Sonarqube works on Port 9000, so <Public IP>:9000. Goto your Sonarqube Server. Click on Administration → Security → Users → Click on Tokens and Update Token → Give it a name → and click on Generate Token

![Screenshot](images/images18.png)

Goto Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

![Screenshot](images/images19.png)

- Now, go to Dashboard → Manage Jenkins → System and Add like the below image.

![Screenshot](images/images20.png)

The Configure System option is used in Jenkins to configure different server

Global Tool Configuration is used to configure different tools that we install using Plugins

- We will install a sonar scanner in the tools.

![Screenshot](images/images21.png)

In the Sonarqube Dashboard add a quality gate also

Administration–> Configuration–>Webhooks

![Screenshot](images/images22.png)

## Step 4  Install OWASP Dependency Check Plugins

- GotoDashboard → Manage Jenkins → Plugins → OWASP Dependency-Check. Click on it and install it without restart.

![Screenshot](images/images23.PNG)

First, we configured the Plugin and next, we had to configure the Tool

Goto Dashboard → Manage Jenkins → Tools →

![Screenshot](images/images24.png)

## Step 5  Docker Image Build and Push

We need to install the Docker tool in our system, Goto Dashboard → Manage Plugins → Available plugins → Search for Docker and install these plugins

Docker, Docker Common, Docker Pipeline, Docker API, docker-build-step

Add DockerHub Username and Password under Global Credentials

![Screenshot](images/images25.PNG)

## Step 5  With all plugins installed, now we can run the pipeline

```sh

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Sarthak221198/Netflix-Clone-DevSecOps.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix
                    '''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                        sh '''
                        docker build --build-arg TMDB_V3_API_KEY=4d838e49fd1660a70b4325ca7667fa4c --build-arg API_URL="https://api.themoviedb.org/3" -t netflix .
                        docker tag netflix sarthak2211/netflix:latest
                        docker push sarthak2211/netflix:latest
                        '''
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image sarthak2211/netflix > trivyimage.txt"
            }
        }
        
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 sarthak2211/netflix'
            }
        }
    }
    post {
        always {
            emailext attachLog: true,
                subject: "'${currentBuild.result}'",
                body: "Project: ${env.JOB_NAME}<br/>" +
                      "Build Number: ${env.BUILD_NUMBER}<br/>" +
                      "URL: ${env.BUILD_URL}<br/>",
                to: 'sarthakmamgain44@gmail.com',
                attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}

```
![Screenshot](images/images26.png)

- Email notification for the success build

![Screenshot](images/images27.PNG)

## 🎯 Final Outcome
Once all steps are completed, the **Netflix Clone App** will be accessible in the browser, running on a secure and optimized DevSecOps pipeline. 🚀

