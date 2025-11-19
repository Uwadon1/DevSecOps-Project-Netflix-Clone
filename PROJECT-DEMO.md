Netflix Clone DevSecOps: CI/CD Pipeline with Jenkins, Docker, Kubernetes & Security
Hello friends, we will be deploying a Netflix clone. We will be using Jenkins as a CICD tool and deploying our application on a Docker container and a Kubernetes Cluster and we will monitor the Jenkins and Kubernetes metrics using Grafana, Prometheus and Node exporter. I hope this blog is useful.
CLICK HERE FOR GITHUB REPOSITORY
Step 1 - Launch an Ubuntu(22.04) T2 Large Instance
Step 2 - Install Jenkins, Docker and Trivy. Create a SonarQube container using Docker.
Step 3 - Create a TMDB API Key.
Step 4 - Install Prometheus and Grafana on a new Server.
Step 5 - Install the Prometheus Plugin and integrate it with the Prometheus server.
Step 6 - Email Integration with Jenkins and plugin setup.
Step 7 - Install plugins such as JDK, SonarQube Scanner, Node.js, and OWASP Dependency Check.
Step 8 - Create a Pipeline Project in Jenkins using a Declarative Pipeline
Step 9 - Install OWASP Dependency Check Plugins
Step 10 - Docker Image Build and Push
Step 11 - Deploy the image using Docker
Step 12 - Kubernetes master and slave setup on Ubuntu (20.04)
Step 13 - Access the Netflix app on the Browser.
Step 14 - Terminate the AWS EC2 Instances.
Now, let's get started and dig deeper into each of these steps:-
STEP1: Launch an Ubuntu(24.04) T2 Large Instance
Launch an AWS and give it any name of your choice. I'll call mine "netflix-jenkins-server". Choose an Ubuntu server with 24.04 LTS.
Launch a new Ubuntu server![Launch a new Ubuntu server](./images/server-name.png)
Select T2 Large as the instance type, you can create a new key pair or use an existing one, and for the sake of this video, we will use the default VPC
Select the instance type![Select the instance type](./images/instance-type.png)
For now, enable SSH, HTTP and HTTPS settings in the Security Group. We will come back to open all the necessary ports and explain why we need them open. Select the storage size as 30 GB and click on launch instance.
Launch the server![Launch the server](./images/launch-instance.png)
Step 2 - Testing Locally
Clone the Code:
Update all the packages on your server using the 'apt update' and then clone the application's code repository onto the EC2 instance:
git clone https://github.com/Uwadon1/End-to-End-Kubernetes-CICD-Jenkins-Pipeline.git
Clone Git from the repo![Clone Git from the repo](./images/git-clone.png)
Step 3: Install Docker and Run the App Using a Container:
To test the application locally, we can either do that via NPM or Docker. I'm opting for the Docker option. Now, I need to set up Docker on the EC2 instance using the apt repository
Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker apt repository. Afterwards, you can install and update Docker from the repository.
Set up Docker's apt repository.

# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
Install the Docker packages.

To install the latest version, run:
`sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`
Install Docker on the server![Install Docker on the server](./images/install-docker.png)
Note: The Docker service starts automatically after installation. To verify that Docker is running, use:
sudo systemctl status docker

Some systems may have this behaviour disabled and will require a manual start:

sudo systemctl start docker
![Check Docker status](./images/docker-status.png)
Verify that the installation is successful by running the hello-world image:
`sudo docker run hello-world`
You have now successfully installed and started Docker Engine.
Run hello world![Run hello world](./images/hello-world.png)
This command downloads a test image and runs it in a container. When the container runs, it prints a confirmation message and exits.
To create the Docker group and add your user:

Create the Docker group.
`sudo groupadd docker`
Add your user to the Docker group.
`sudo usermod -aG docker $USER`
`newgrp docker`
`sudo chmod 777 /var/run/docker.sock`
Docker permission![Docker permission](./images/docker-permission.png)
Build and run your application using Docker containers:
docker build -t netflix .
Build Docker image![Build Docker image](./images/docker-build.png)
docker run -d --name netflix -p 8081:80 netflix:latest
Run Docker container![Run Docker container](./images/docker-run.png)
We will get a blank Netflix page because we have not included the API key for Netflix, so for the next step, we will have to go to TMDB to obtain our API key from there.
Blank Netflix page![Blank Netflix page](./images/blank-netflix.png)
Step 4: Get the API Key:
Open a web browser and navigate to TMDB (The Movie Database) website, click on "Login" and create an account.
Once your account is verified and you are logged in, go to your profile and select "Settings." Click on "API" from the left-side panel, create a new API key by clicking "Create" and accepting the terms and conditions.
Provide the required basic details and click "Submit." You will receive your TMDB API key.
Get the API key from TMDB![Get the API key from TMDB](./images/api-key.png)
Now let's recreate the Docker image with your api key:
But before then, we need to delete the previous container by running the commands below
docker stop <containerid>

&&

docker rmi -f netflix
Now, we can rebuild the container using the command below:
docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .
Build Docker image![Build Docker image](./images/docker-build2.png)
To run the container, run the command below
docker run -d -p 8081:80 netflix
Run Docker container![Run Docker container](./images/docker-run2.png)
Voila!!! We can see the Netflix homepage
![View Netflix homepage](./images/netflix-page.png)
Phase 2: Security
For this next phase, we will need to install SonarQube and Trivy:
Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.
SonarQube is an open-source platform for continuous inspection of code quality and security. It performs static code analysis to automatically detect bugs, vulnerabilities, code smells, and duplication, helping development teams identify and fix issues early in the software development lifecycle. By integrating with CI/CD pipelines, SonarQube helps ensure that code meets quality standards before release.
To install SonarQube, run the command below.
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
Install Sonarqube via Docker![Install Sonarqube via Docker](./images/install-sonar.png)
To access, type in your public IP:9000, and ensure port 9000 is open on your EC2 machine.
(By default, username & password are admin).
Login into Sonarqube![Login into Sonarqube](./images/sonar-login.png)
Once you log in, you will be prompted to change your password and login details.
To install Trivy: Trivy is an open-source security scanner that detects vulnerabilities and misconfigurations in various targets like container images, file systems, and Git repositories. It is designed to be easy to use and is praised for its speed and comprehensive capabilities, including scanning for OS package and application dependency vulnerabilities (CVEs), infrastructure as code (IaC) issues, and secrets.
sudo apt-get install wget apt-transport-https gnupg lsb-release

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt-get update

sudo apt-get install trivy -y
Install trivy![Install trivy](./images/install-trivy.png)
To scan files or images using trivy, run the following command.
`trivy fs .`
Use trivy to scan files![Use trivy to scan files](./images/trivy-fs.png)
`trivy image <imageid>`
Use trivy to scan Docker image![Use trivy to scan Docker image](./images/trivy-image.png)
Integrate SonarQube and Configure:

Integrate SonarQube with your CI/CD pipeline.
Configure SonarQube to analyze code for quality and security issues.

Phase 3: CI/CD Setup
Install Jenkins for Automation:

In this phase, we will install Jenkins on the EC2 instance to automate deployment. First, we need to install Java, because it is a prerequisite for installing Jenkins
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
java -version
Install Java![Install Java](./images/install-java.png)
#jenkins

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \

https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \

https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt-get update

sudo apt-get install jenkins -y
Install jenkins![Install jenkins](./images/install-jenkins.png)
Start Jenkins
You can enable the Jenkins service to start at boot with the command:
sudo systemctl enable jenkins

#You can start the Jenkins service with the command:

sudo systemctl start jenkins

#You can check the status of the Jenkins service using the command:

sudo systemctl status jenkins

#If everything has been set up correctly, you should see an output like this:
Jenkins Status and Enable![Jenkins Status and Enable](./images/jenkins-status.png)
Once Jenkins is installed and enabled, go to your AWS EC2 Security Group and ensure Port 8080 is open, since Jenkins works on Port 8080.
Now, grab your Public IP Address
<EC2 Public IP Address:8080>
View Jenkins homepage![View Jenkins homepage](./images/jenkins-homepage.png)
To log in to the Jenkins dashboard, run this command:
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`  to get the administrator password
Jenkins Password![Jenkins Password](./images/jenkins-password.png)
Once in, install the suggested plugins.
Jenkins plugins![Jenkins plugins](./images/jenkins-plugins.png)
You will need to create a user, fill in the necessary details and click on save and continue.
Jenkins Getting Started Screen.
Jenkins Start page![Jenkins Start page](./images/jenkins-startscreen.png)
Install Necessary Plugins in Jenkins:

For now, these are the plugins we will need to install on Jenkins. To install plugins, go to manage Jenkins, plugins, then available plugins and search them out
Install below plugins
1 Eclipse Temurin Installer (Install without restart)
2 SonarQube Scanner (Install without restart)
3 NodeJs Plugin (Install Without restart)
4 Email Extension Plugin
Jenkins plugins![Jenkins plugins](./images/jenkins-plugins.png)
Configure Java and Node.js in Global Tool Configuration
Once the plugins are installed, we will go and configure the necessary tools for Node.js and JDK.
Go to Manage Jenkins, then tools, and we will configure the tool for JDK(17) and ensure the name aligns with the name on our pipeline script.
Jdk plugins install![Jdk plugins install](./images/jdk.png)
Configure Node.Js(16) → Click on Apply and Save
We will also configure the tool for Node JS (16) and ensure the name aligns with the name on our pipeline script.
Node.JS plugins![Node.JS plugins](./images/nodejs.png)
Go to Manage Jenkins, then tools, and we will configure the tool for SonarQube Scanner and ensure the name aligns with the name on our pipeline script.
Click on Apply and Save
Configure SonarQube tools![Configure SonarQube tools](./images/sonar.png)
Create SonarQube Token
To create the token for Sonarqube and attach to the credential, go to the Sonarqube dashboard and click on my account, security and generate a token and copy the token into the Jenkins credentials
Create Sonar token![Create Sonar token](./images/sonar-token.png)
Go to Jenkins Dashboard, manage Jenkins, credentials, select Add Secret Text. It should look like this after adding your Sonar token
Jenkins credentials for Sonar![Jenkins credentials for Sonar](./images/jenkins-credential.png)
Next, we will configure our Jenkins system. The system option is used in Jenkins to configure different servers
Global Tool Configuration is used to configure different tools that we install using Plugins
We will install a sonar scanner in the tools.
Configure Sonar Scanner in the System![Configure Sonar Scanner in the System](./images/configure-sonar.png)
Create a Jenkins webhook
Configure CI/CD Pipeline in Jenkins:

Create a CI/CD pipeline in Jenkins to automate your application deployment.

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
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Uwadon1/DevSecOps-Project-Netflix-Clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
    }
}
Install Dependency-Check and Docker Tools in Jenkins
Install Dependency-Check Plugin:
Navigate to "manage Jenkins" in your Jenkins web interface, and go to manage plugins, click on the "Available" tab and search for "OWASP Dependency-Check."
Check the checkbox for "OWASP Dependency-Check" Now we will install Docker Tools and Docker Plugins also:
Search for "Docker." Click and select the following Docker-related plugins:
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Click on the "Install without restart" button to install these plugins.

Configure Owasp plugins![Configure Owasp plugins](./images/owasp-plugins.png)
Configure Dependency-Check Tool:
After installing the Dependency-Check plugin, you need to configure the tool.
Go to "Manage Jenkins", global tool configuration, then scroll down to find the section for "OWASP Dependency-Check."
Add the tool's name, e.g., "DP-Check." and select to uninstall automatically.

Configure Owasp tools![Configure Owasp tools](./images/owasp-dependencies.png)
Now, you have to configure the Docker tool, give it the name Docker and select to install automatically and save and apply.
Add DockerHub Credentials:
To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
Go to "Manage Credentials", click on "Global credentials (unrestricted)", then click on "Add Credentials" on the left side. Choose "Username with password" as the kind of credentials.
Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker-cred").
Click "OK" to save your DockerHub credentials.

Configure Docker token in GitHub secrets![Configure Docker token in GitHub secrets](./images/docker-token.png)
![Configure Dockerhub token](./images/docker-token2.png)
You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/Uwadon1/DevSecOps-Project-Netflix-Clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-scanner') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=3dc59a205938ca0b797e7fb2bbe0c942 -t netflix ."
                       sh "docker tag netflix uwadon01/netflix:latest "
                       sh "docker push uwadon01/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image uwadon01/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 uwadon01/netflix:latest'
            }
        }
    }
}

If you get Docker login failed error, run the below commands.

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
Checkout the pipeline result![Checkout the pipeline result](./images/pipeline-result.png)
Phase 4: Monitoring
Install Prometheus and Grafana:

Set up Prometheus and Grafana to monitor your application.
Installing Prometheus:
First, we need to update the packages using the commands:
sudo apt update && sudo apt upgrade -y
Next, create a dedicated Linux user for Prometheus and download Prometheus:

sudo useradd - system - no-create-home - shell /bin/false prometheus

wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
Download Prometheus![Download Prometheus](./download-prometheus.png)
2. Extract Prometheus files, move them, and create directories:
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz

cd prometheus-2.47.1.linux-amd64/

sudo mkdir -p /data /etc/prometheus

sudo mv prometheus promtool /usr/local/bin/

sudo mv consoles/ console_libraries/ /etc/prometheus/

sudo mv prometheus.yml /etc/prometheus/prometheus.yml
Unzip Prometheus![Unzip Prometheus](./unzip-prometheus.png)
List the copied files in Prometheus![List the copied files in Prometheus](./list-prometheus.png)
3. Set ownership for directories:
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
Grant prometheus user and owner permission in Prometheus![Grant Prometheus user and owner permission in Prometheus](./prometheus-permission.png)
4. Create a systemd unit configuration file for Prometheus:
sudo vi /etc/systemd/system/prometheus.service
5. Add the following content to the prometheus.service file:
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
Here's a brief explanation of the key parts in this prometheus.service file:
User and Group specify the Linux user and group under which Prometheus will run.
ExecStart is where you specify the Prometheus binary path, the location of the configuration file (prometheus.yml), the storage directory, and other settings.
web.listen-address configures Prometheus to listen on all network interfaces on port 9090.
web.enable-lifecycle allows for management of Prometheus through API calls.

List the copied files in Prometheus![List the copied files in Prometheus](./prometheus-service.png)
7. Enable start and verify Prometheus status:
sudo systemctl enable prometheus

sudo systemctl start prometheus

sudo systemctl status prometheus
View the status of Prometheus![View the status of Prometheus](./prometheus-status.png)
You can access Prometheus in a web browser using your server's IP and port 9090:
`http://<your-server-ip>:9090`
List the copied files in Prometheus![List the copied files in Prometheus](./monitoring-sg.png)
View the Prometheus dashboard![View the Prometheus dashboard](./prometheus-dashboard.png)
Installing Node Exporter:
1. Create a system user for Node Exporter and download Node Exporter. Remember to cd to home (~) before proceeding:
sudo useradd - system - no-create-home - shell /bin/false node_exporter

wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
Download Node exporter![Download Node exporter](./download-nodeexp.png)
2. Extract Node Exporter files, move the binary, and clean up:
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
Unzip the node exporter![Unzip the node exporter](./node-unzip.png)
3. Create a systemd unit configuration file for Node Exporter:
sudo vi /etc/systemd/system/node_exporter.service
4. Add the following content to the node_exporter.service file:
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
StartLimitIntervalSec=500
StartLimitBurst=5
[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter - collector.logind
[Install]
WantedBy=multi-user.target
List the copied files in Prometheus![List the copied files in Prometheus](./node-service.png)
5. Enable, start and verify Node Exporter:
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
sudo systemctl status node_exporter
Node status![Node status](./node-status.png)
6. You can access Node Exporter metrics in Prometheus.
![View the monitoring status](./monitoring-status.png)
7. To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file. Here is an example prometheus.yml configuration for your setup:
global:
scrape_interval: 15s
scrape_configs:
- job_name: 'node_exporter'
static_configs:
- targets: ['localhost:9100']
- job_name: 'jenkins'
metrics_path: '/prometheus'
static_configs:
- targets: ['<your-jenkins-ip>:<your-jenkins-port>']

8. Make sure to replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate values for your Jenkins setup.
Check the validity of the configuration file:
promtool check config /etc/prometheus/prometheus.yml
Verify Prometheus configuration![Verify Prometheus configuration](./promtool-check.png)
9. Reload the Prometheus configuration without restarting:
curl -X POST http://localhost:9090/-/reload
10. You can access Prometheus targets at:
`http://<your-prometheus-ip>:9090/targets`
NB: We can check the Prometheus target dashboard, and only Prometheus and Node exporter will be showing, Jenkins won't show until we integrate Prometheus plugins on Jenkins.
Grafana
Install Grafana on Ubuntu 24.04 and Set it up to Work with Prometheus
Step 1: Install Dependencies:
First, ensure that all necessary dependencies are installed:
sudo apt-get update

sudo apt-get install -y apt-transport-https software-properties-common
Update and Install Grafana![Update and Install Grafana](./grafana-update.png)
Step 2: Add the GPG Key:
Add the GPG key for Grafana and Grafana Repository:
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
Grafana echo![Grafana echo](./grafana-echo.png)
Step 4: Update and Install Grafana:
Update the package list and install Grafana:
sudo apt-get update
sudo apt-get -y install grafana
![Install Grafana](./grafana-install.png)
Step 5: Enable, start and verify Grafana Service status:
To automatically start Grafana after a reboot, enable the service:
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
sudo systemctl status grafana-server
Grafana status![Grafana status](./grafana-status.png)
Step 7: Access Grafana Web Interface:
Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:
`http://<your-server-ip>:3000`
You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."
View Grafana dashboard![View Grafana dashboard](./grafana-dashboard.png)
View Grafana Homepage![View Grafana Homepage](./grafana-homepage.png)
Step 8: Change the Default Password:
When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password or skip.
Step 9: Add Prometheus Data Source:
To visualize metrics, you need to add a data source. Follow these steps:
Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.
Select "Data Sources", click on the "Add data source" button. Choose "Prometheus" as the data source type.
In the "HTTP" section, set the "URL" to http://localhost:9090 (assuming Prometheus is running on the same server) or you can input the url of your Prometheus server.
Click the "Save & Test" button to ensure the data source is working.
Save the Grafana test![Save the Grafana test](./grafana-savetest.png)
Step 10: Import a Dashboard:
To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:
Click on the "+" (plus) icon in the left sidebar to open the "Create" menu. Select "Dashboard." Click on the "Import" dashboard option.

Import dashboard into Grafana![Import dashboard into Grafana](./import-dashboard.png)
Enter the dashboard code you want to import (e.g., code 1860 - to get this, Google Node exporter grafana dashboard). Click the "Load" button, select the data source you added (Prometheus) from the dropdown. Click on the "Import" button.

Input the respective code![Input the respective code](./node-code.png)
You should now have a Grafana dashboard set up to visualize metrics from Prometheus.
Grafana is a powerful tool for creating visualizations and dashboards, and you can further customize it to suit your specific monitoring needs.
That's it! You've successfully installed and set up Grafana to work with Prometheus for monitoring and visualization.
View the metrics visualization inside Grafana![View the metrics visualization inside Grafana](./visualization1.png)
To monitor Jenkins, we will need to configure Prometheus Plugin Integration into Jenkins, search for Prometheus Metrics and install it.:

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

![Install Prometheus plugin](./prometheus-plugin.png)
Next, we will need to configure the system, go to the manage dashboard and click on configure system.
Scroll down till you see Prometheus, select it and click on save and apply.
Prometheus system![Prometheus system](./prometheus-system.png)
We can check our Jenkins URL and add /prometheus to it, we should see the output below:
View the raw Prometheus metrics![View the raw Prometheus metrics](./raw-metrics.png)
Next, we will import the dashboard for Jenkins, to make it easier to view the Jenkins metrics, you can import a pre-configured dashboard. Follow these steps:
Click on the "+" (plus) icon in the left sidebar to open the "Create" menu. Select "Dashboard." Click on the "Import" dashboard option. Enter the dashboard code you want to import (e.g., code 9964 - to get this, Google Node exporter Grafana dashboard). Click the "Load" button, select the data source you added (Prometheus) from the dropdown. Click on the "Import" button.

Load the dashboard![Load the dashboard](./load-dashboard.png)
You should now have a Jenkins dashboard set up to visualize metrics from Prometheus.
Jenkins visualization dashboard in Grafana![Jenkins visualization dashboard in Grafana](./jenkins-visuals.png)
Phase 5: Notification
Implement Notification Services: Set up email notifications in Jenkins or other notification mechanisms. To implement notification, go to your Gmail account and click on Manage Google Account, then click on the Security section. Ensure you have 2 FA verification set up.

Set up Google 2FA![Set up Google 2FA](./2fa.png)
Search for app password, click on it and give it a name, then click on create to generate the code.

Generate Google passcode![Generate Google passcode](./passcode.png)
Go to manage Jenkins –> configure system there under the E-mail Notification section configure the details as shown in the below image
Configure email![Configure email](./configure-email.png)
At the bottom, there's an option to test configuration, insert your email and check your mailbox.
The sample email from Jenkins![The sample email from Jenkins](./sample-email.png)
Click on Apply and save.
Click on Manage Jenkins –> credentials and add your mail username and generated password
Set up the Gmail token in my GitHub secrets![Set up the Gmail token in my GitHub secrets](./gmail-cred.png)
Now scroll back up, and under the Extended E-mail Notification section, configure the details as shown in the images below
For the default content type, select HTML, and for the triggers, select "always" and "any build failure"
Extended email settings set up![Extended email settings set up](./extended-email.png)
Lastly, we will include a post-stage in the pipeline
post {
always {
emailext attachLog: true,
subject: "'${currentBuild.result}'",
body: "Project: ${env.JOB_NAME}<br/>" +
"Build Number: ${env.BUILD_NUMBER}<br/>" +
"URL: ${env.BUILD_URL}<br/>",
to: 'uwadon1@gmail.com', #change Your mail
attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
}
}
Click on Apply and save.
Next, we will log in to Jenkins and start to configure our Pipeline in Jenkins
Phase 6: Kubernetes
Create a Kubernetes Cluster with Nodegroups
In this phase, you'll set up a Kubernetes cluster with node groups. This will provide a scalable environment to deploy and manage your applications.
Navigate to your EKS cluster and create a cluster and give it any name, e.g: Netflix-App. Create and select the default role: AmazonEKSAutoClusterRole
Create AWS EKS![Create AWS EKS](./eks-create.png)
Create a node group and assign the necessary roles and permissions.
In your terminal, type this command to connect to your EKS: aws eks update-kubeconfig - name Netflix-App - region us-west-2
Create Node Group on your cluster![Create Node Group on your cluster](./create-nodegroup.png)
Step 2: Create a Namespace for Argo CD
kubectl create namespace argocd
Step 3: Install Argo CD using Helm (Recommended)
Add the official Argo Helm repo:
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
Install Argo CD using Helm![Install Argo CD using Helm](./argo-helm.png)
Install Argo CD into the Argo CD namespace:
helm install argocd argo/argo-cd -n argocd
To verify installation:
kubectl get pods -n argocd
You should see multiple pods like:
argocd-server
argocd-repo-server
argocd-application-controller
argocd-dex-server
Install Argo CD using Helm chart![Install Argo CD using Helm chart](./argo-install.png)
Step 4: Expose Argo CD Server
By default, the Argo CD server service is ClusterIP (internal only).
 To access it via browser, change it to LoadBalancer:
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
Then get the external IP:
kubectl get svc -n argocd
Look for:
argocd-server LoadBalancer <EXTERNAL-IP> 80:xxxx/TCP,443:xxxx/TCP
List the components of Argo CD![List the components of Argo CD](./get-argo.png)
Access the Argo CD UI via:
`https://<EXTERNAL-IP>`
Access the Argo CD UI![Access the Argo CD UI](./argo-ui.png)
🔑 Step 5: Get the Initial Admin Password
Argo CD creates a default admin password as a secret. Retrieve it with:
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d echo
Now log in with
Username: admin
Password: (the value you just decoded)

Get Argo CD secrets![Get Argo CD secrets](./argo-getsecrets.png)
Monitor Kubernetes with Prometheus
Prometheus is a powerful monitoring and alerting toolkit, and you'll use it to monitor your Kubernetes cluster. Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.
Install Node Exporter using Helm
To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:
Add the Prometheus Community Helm repository:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
2. Create a Kubernetes namespace for the Node Exporter:
kubectl create namespace prometheus-node-exporter
3. Install the Node Exporter using Helm:
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter - namespace prometheus-node-exporter
Install Prometheus Node Exporters![Install Prometheus Node Exporters](./install-node.png)
Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:
Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:
- job_name: 'Netflix'
metrics_path: '/metrics'
static_configs:
- targets: ['nodeIp:9100']
Replace 'your-job-name' with a descriptive name for your job. The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.
Don't forget to reload or restart Prometheus to apply these changes to your configuration.
To deploy an application with ArgoCD, you can follow these steps, which I'll outline in Markdown format:
Deploy Application with ArgoCD
Install ArgoCD:
You can install ArgoCD on your Kubernetes cluster by following the instructions provided in the EKS Workshop documentation.
Set Your GitHub Repository as a Source:
After installing ArgoCD, you need to set up your GitHub repository as a source for your application deployment. This typically involves configuring the connection to your repository and defining the source for your ArgoCD application. The specific steps will depend on your setup and requirements.
Create an ArgoCD Application:

name: Set the name for your application.
destination: Define the destination where your application should be deployed.
project: Specify the project the application belongs to.
source: Set the source of your application, including the GitHub repository URL, revision, and the path to the application within the repository.
syncPolicy: Configure the sync policy, including automatic syncing, pruning, and self-healing.

4. Access your Application
To Access the app, make sure port 30007 is open in your security group and then open a new tab, paste your NodeIP:30007, and your app should be running.

Final result of the Netflix app![Final result of the Netflix app](./app-final.png)
