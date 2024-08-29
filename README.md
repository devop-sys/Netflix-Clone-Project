![image](https://github.com/user-attachments/assets/06ce385f-38ac-4072-b163-4696e4ba7843)
#

# **Deploy Netflix Clone on Cloud using Jenkins - DevSecOps Project!**
## **Phase 1: Starting With Dev Part**- How to run application locally

**Step 1: Launch EC2 (Ubuntu 22.04):**
1) Provision an EC2 instance on AWS with Ubuntu 22.04.
2) The instance type must be T2 large because we are going to integrate a lot of dependencies, such as SonarQube, Trivy, and many others.
3) Connect to the instance using SSH. 

**Step 2: Clone the Code:**
1. Update all the packages and then clone the code.
2. Clone your application's code repository onto the EC2 instance
git clone https://github.com/devop-sys/Netflix-Clone-Project.git
