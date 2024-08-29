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

   Integrate SonarQube with your CI/CD pipeline.

   Configure SonarQube to analyze code for quality and security issues.
