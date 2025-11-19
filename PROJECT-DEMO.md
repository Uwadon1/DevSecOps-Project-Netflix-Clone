# Netflix Clone DevSecOps: CI/CD Pipeline with Jenkins, Docker, Kubernetes & Security


Here's a comprehensive README.md file for your Netflix Clone DevSecOps CI/CD Pipeline project:

```markdown
# Netflix Clone DevSecOps: CI/CD Pipeline with Jenkins, Docker, Kubernetes & Security

![DevSecOps Pipeline](https://img.shields.io/badge/DevSecOps-CI%2FCD%20Pipeline-blue)
![Docker](https://img.shields.io/badge/Container-Docker-green)
![Kubernetes](https://img.shields.io/badge/Orchestration-Kubernetes-orange)
![Security](https://img.shields.io/badge/Security-SonarQube%2FTrivy-red)

A comprehensive DevSecOps project demonstrating end-to-end CI/CD pipeline for a Netflix clone application with integrated security scanning, containerization, Kubernetes deployment, and monitoring.

## 📋 Project Overview

This project implements a complete DevSecOps pipeline that:
- **Builds** a Netflix clone application
- **Scans** for vulnerabilities using SonarQube, OWASP Dependency Check, and Trivy
- **Containerizes** the application using Docker
- **Deploys** to Kubernetes cluster
- **Monitors** using Prometheus and Grafana
- **Automates** everything through Jenkins CI/CD

## 🏗️ Architecture

```
Git Repository → Jenkins Pipeline → Security Scanning → Docker Build → Kubernetes Deployment → Monitoring
     ↓              ↓                    ↓                  ↓                 ↓                  ↓
  Netflix      Automated CI/CD     SonarQube/Trivy    Containerized    EKS Cluster       Prometheus +
   Code         with Jenkins        OWASP DC          Application                       Grafana Dashboards
```

## 🚀 Prerequisites

- AWS Account with EC2 access
- Docker Hub account
- TMDB API account
- Basic knowledge of Linux, Docker, and Kubernetes

## 📁 Project Structure

```
netflix-devsecops/
├── Jenkinsfile                 # Jenkins pipeline configuration
├── Dockerfile                  # Docker container definition
├── kubernetes/                 # Kubernetes manifests
├── monitoring/                 # Prometheus & Grafana configs
├── src/                       # Application source code
└── README.md                  # Project documentation
```

## 🛠️ Installation & Setup

### Phase 1: Infrastructure Setup

#### Step 1: Launch EC2 Instance
```bash
# Ubuntu 24.04 LTS, T2 Large, 30GB storage
# Security Groups: SSH (22), HTTP (80), HTTPS (443), Jenkins (8080)
```

#### Step 2: Clone Repository & Initial Setup
```bash
git clone https://github.com/Uwadon1/End-to-End-Kubernetes-CICD-Jenkins-Pipeline.git
cd End-to-End-Kubernetes-CICD-Jenkins-Pipeline
```

### Phase 2: Security Tools Installation

#### Install Docker
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Configure Docker permissions
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock
```

#### Install SonarQube
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
# Access at: http://<server-ip>:9000
# Default credentials: admin/admin
```

#### Install Trivy
```bash
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y
```

### Phase 3: CI/CD Setup with Jenkins

#### Install Java & Jenkins
```bash
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y

# Jenkins installation
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update
sudo apt-get install jenkins -y

sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

#### Configure Jenkins Plugins
Install the following plugins:
- Eclipse Temurin Installer
- SonarQube Scanner
- NodeJs Plugin
- Email Extension Plugin
- OWASP Dependency-Check
- Docker Pipeline
- Prometheus Metrics

#### Configure Tools in Jenkins
1. **JDK 17**: Name as 'jdk17'
2. **Node.js 16**: Name as 'node16'  
3. **SonarQube Scanner**: Name as 'sonar-scanner'
4. **Docker Tool**: Name as 'docker'

#### Set Up Credentials
1. **SonarQube Token**: Add as secret text
2. **Docker Hub**: Add username/password credentials
3. **Email**: Configure SMTP settings for notifications

### Phase 4: Monitoring Stack

#### Install Prometheus
```bash
sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/

sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
```

#### Install Node Exporter
```bash
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
```

#### Install Grafana
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

sudo apt-get update
sudo apt-get -y install grafana

sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

### Phase 5: Kubernetes Deployment

#### Create EKS Cluster
```bash
# Using AWS Management Console or AWS CLI
aws eks update-kubeconfig --name Netflix-App --region us-west-2
```

#### Install ArgoCD
```bash
kubectl create namespace argocd
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
helm install argocd argo/argo-cd -n argocd

# Expose ArgoCD UI
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

## 🔧 Configuration

### TMDB API Key Setup
1. Visit [TMDB](https://www.themoviedb.org/)
2. Create account and verify
3. Go to Settings → API
4. Create new API key
5. Use in Docker build: `--build-arg TMDB_V3_API_KEY=your-api-key`

### Jenkins Pipeline
The main Jenkins pipeline includes:
- Code checkout
- SonarQube analysis
- Quality gate check
- Dependency installation
- OWASP Dependency Check scan
- Trivy file system scan
- Docker build and push
- Trivy image scan
- Deployment to container
- Email notifications

### Prometheus Configuration
Update `/etc/prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<jenkins-ip>:<jenkins-port>']
```

## 📊 Monitoring & Dashboards

### Grafana Dashboards
1. **Node Exporter Dashboard**: Import dashboard ID `1860`
2. **Jenkins Dashboard**: Import dashboard ID `9964`
3. **Kubernetes Dashboard**: Custom monitoring setup

### Access Points
- **Jenkins**: `http://<server-ip>:8080`
- **SonarQube**: `http://<server-ip>:9000`
- **Prometheus**: `http://<server-ip>:9090`
- **Grafana**: `http://<server-ip>:3000`
- **ArgoCD**: `https://<loadbalancer-ip>`
- **Netflix App**: `http://<node-ip>:30007`

## 🛡️ Security Features

- **Static Code Analysis**: SonarQube for code quality and security
- **Dependency Scanning**: OWASP Dependency Check for vulnerability detection
- **Container Security**: Trivy for Docker image scanning
- **Secrets Management**: Secure credential handling in Jenkins
- **Network Security**: Proper security group configurations

## 📧 Notifications

Configure email notifications in Jenkins:
- SMTP server: `smtp.gmail.com`
- Port: `587`
- Use app passwords for Gmail with 2FA enabled

## 🚀 Deployment Pipeline

The complete pipeline workflow:
1. **Code Commit** → Triggers Jenkins pipeline
2. **Security Scanning** → SonarQube + Trivy + OWASP
3. **Quality Gate** → Pass/fail based on metrics
4. **Docker Build** → Containerize application
5. **Security Scan** → Trivy image scanning
6. **Registry Push** → Push to Docker Hub
7. **Kubernetes Deployment** → Deploy to EKS via ArgoCD
8. **Monitoring** → Prometheus metrics collection
9. **Notification** → Email status updates

## 🧹 Cleanup

```bash
# Terminate EC2 instances
# Delete EKS cluster
# Remove Docker images
# Clean up Jenkins workspace
```

## 🐛 Troubleshooting

### Common Issues
1. **Docker permission denied**: 
   ```bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```

2. **SonarQube not accessible**: Check port 9000 in security groups

3. **Pipeline failures**: Check Jenkins logs and workspace permissions

4. **TMDB API issues**: Verify API key and rebuild with correct key

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Commit changes
4. Push to branch
5. Create Pull Request

## 📄 License

This project is for educational purposes as part of DevSecOps learning.

## 🙏 Acknowledgments

- Netflix for the UI inspiration
- TMDB for movie data API
- Jenkins, Docker, Kubernetes communities
- Prometheus & Grafana for monitoring solutions

---

**⭐ Star this repo if you found it helpful!**
```

This README.md provides:

1. **Comprehensive overview** of the entire project
2. **Step-by-step instructions** with code blocks
3. **Clear architecture** and workflow diagrams
4. **Troubleshooting section** for common issues
5. **Visual badges** for better presentation
6. **Well-organized sections** for easy navigation
7. **Security emphasis** throughout the documentation
8. **Monitoring and alerting** configurations
9. **Cleanup instructions** for resource management

The README is structured to guide users from setup to deployment while emphasizing the DevSecOps practices integrated throughout the pipeline.
