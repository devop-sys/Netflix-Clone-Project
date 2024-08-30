![image](https://github.com/user-attachments/assets/06ce385f-38ac-4072-b163-4696e4ba7843)
#

# **Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!**
###  **Phase 1: Starting With Dev Part**- How to run application locally

**Step 1: Launch EC2 (Ubuntu 22.04):**
1) Provision an EC2 instance on AWS with Ubuntu 22.04.
2) The instance type must be T2 large because we are going to integrate a lot of dependencies, such as SonarQube, Trivy, and many others.
3) Connect to the instance using SSH. 

**Step 2: Clone the Code:**
1. Update all the packages and then clone the code.
2. Clone your application's code repository onto the EC2 instance

        git clone https://github.com/devop-sys/Netflix-Clone-Project.git
**Step3: Installing Docker and deploying the application in a container.**  
1. Set up Docker on the EC2 instance:
      
       sudo apt-get update
       sudo apt-get install docker.io -y
       sudo usermod -aG docker $USER  # Replace with your system's username, e.g., 'ubuntu'
       newgrp docker
       sudo chmod 777 /var/run/docker.sock
       docker version
2. Leverage Docker containers to build and execute your application.

       docker build -t netflix .
       docker run -d --name netflix -p 8081:80 netflix:latest

       #to delete
       docker stop <containerid>
       docker rmi -f netflix
   First check if your port is open or not.

   After running the Dockerfile, the page will appear blank without any movies because the API key wasn't integrated correctly.

   So stop docker server and remove it.

       docker stop "image ID or container name"
       docker rm "image ID or container name"
       docker ps "image ID or container name"

   Now we don't have any server running.
 
**Step 4: Obtain the API Key.**
1. Open a web browser and navigate to TMDB (The Movie Database) website.
2. Click on "Login" and create an account.
3. Once logged in, go to your profile and select "Settings."
4. Click on "API" from the left-side panel.
5. Create a new API key by clicking "Create" and accepting the terms and conditions.
6. Provide the required basic details and click "Submit."
7. You will receive your TMDB API key.

Now recreate the Docker image with your api key:
     
      docker build --build-arg TMDB_V3_API_KEY=<your-api-key> -t netflix .

**Phase 2: Implementing security measures in our application.**
1. **Install SonarQube and trivy**

   Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

    **SonarQube**

         docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
   To access:

   **publicIP:9000** (by default username & password is admin)

   **To install Trivy:**

        sudo apt-get install wget apt-transport-https gnupg lsb-release
        wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
        echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
        sudo apt-get update
        sudo apt-get install trivy

   to scan image using trivy

        trivy image <imageid>
2. **Set up and configure SonarQube integration.**

   i) Integrate SonarQube with your CI/CD pipeline.

   ii) Configure SonarQube to analyze code for quality and security issues.

**Phase 3: CI/CD Configuration**
   
1) **Install Jenkins for Automation:**

   i) Install Jenkins on the EC2 instance to automate deployment: Install Java

          sudo apt update
          sudo apt install fontconfig openjdk-17-jre
          java -version
          openjdk version "17.0.8" 2023-07-18
          OpenJDK Runtime Environment (build 17.0.8+7-Debian-1deb12u1)
          OpenJDK 64-Bit Server VM (build 17.0.8+7-Debian-1deb12u1, mixed mode, sharing)

          #jenkins
          sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
          https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
          echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
          https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
          /etc/apt/sources.list.d/jenkins.list > /dev/null
          sudo apt-get update
          sudo apt-get install jenkins
          sudo systemctl start jenkins
          sudo systemctl enable jenkins
   ii) Access Jenkins in a web browser using the public IP of your EC2 instance.

   **publicIp:8080**
2) **Install Necessary Plugins in Jenkins:**

   Goto Manage Jenkins →Plugins → Available Plugins →

   Install below plugins

   1 Eclipse Temurin Installer (Install without restart)

   2 SonarQube Scanner (Install without restart)

   3 NodeJs Plugin (Install Without restart)

   4 Email Extension Plugin
### **Set up Java and Node.js in the Global Tool Configuration.**

Go to Manage Jenkins → Tools → Install JDK(17) and NodeJs(16)→ Click on Apply and Save

### **SonarQube**

Create the token

Go to Jenkins Dashboard → Manage Jenkins → Credentials → Add Secret Text. It should look like this

After adding sonar token

Click on Apply and Save

**The Configure System** option is used in Jenkins to configure different server

**Global Tool Configuration** is used to configure different tools that we install using Plugins

We will install a sonar scanner in the tools.

Create a Jenkins webhook

 1) Configure CI/CD Pipeline in Jenkins:

    i) Create a CI/CD pipeline in Jenkins to automate your application deployment.

 ```
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
                git branch: 'main', url: 'https://github.com/devop-sys/Netflix-Clone-Project.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage("quality gate") {
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
    }
}
```
**Install Dependency-Check and Docker Tools in Jenkins**

**Install Dependency-Check Plugin:**

i) Go to "Dashboard" in your Jenkins web interface.

ii) Navigate to "Manage Jenkins" → "Manage Plugins."

iii) Click on the "Available" tab and search for "OWASP Dependency-Check."

iv) Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.

**Configure Dependency-Check Tool:**

i) After installing the Dependency-Check plugin, you need to configure the tool.

ii) Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."

iii) Find the section for "OWASP Dependency-Check."

iv) Add the tool's name, e.g., "DP-Check."

v) Save your settings.

**Install Docker Tools and Docker Plugins:**

i) Go to "Dashboard" in your Jenkins web interface.

ii) Navigate to "Manage Jenkins" → "Manage Plugins."

iii) Click on the "Available" tab and search for "Docker."

iv) Check the following Docker-related plugins:

 1) Docker
 2) Docker Commons
 3) Docker Pipeline
 4) Docker API
 5) docker-build-step

v) Click on the "Install without restart" button to install these plugins.

**Add DockerHub Credentials:**

i) To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
   1) Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
   2) Click on "System" and then "Global credentials (unrestricted)."
   3) Click on "Add Credentials" on the left side.
   4) Choose "Secret text" as the kind of credentials.
   5) Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
   6) Click "OK" to save your DockerHub credentials.

Now, you have installed the Dependency-Check plugin, configured the tool, and added Docker-related plugins along with your DockerHub credentials in Jenkins. You can now proceed with configuring your Jenkins pipeline to include these tools and credentials in your CI/CD process.

```

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
                git branch: 'main', url: 'https://github.com/devop-sys/Netflix-Clone-Project.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
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
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=<yourapikey> -t netflix ."
                       sh "docker tag netflix nasi101/netflix:latest "
                       sh "docker push nasi101/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image nasi101/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
    }
}


If you get docker login failed errorr

sudo su
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```
**Phase 4: Monitoring**

**Install Prometheus and Grafana:**

Set up Prometheus and Grafana to monitor your application.

**Installing Prometheus:**

First, create a dedicated Linux user for Prometheus and download Prometheus:

     sudo useradd --system --no-create-home --shell /bin/false prometheus
     wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz

Extract Prometheus files, move them, and create directories:

      tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
      cd prometheus-2.47.1.linux-amd64/
      sudo mkdir -p /data /etc/prometheus
      sudo mv prometheus promtool /usr/local/bin/
      sudo mv consoles/ console_libraries/ /etc/prometheus/
      sudo mv prometheus.yml /etc/prometheus/prometheus.yml
