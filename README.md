# CI/CD Pipeline Setup with Jenkins, Maven, Docker, SonarQube, and EKS

This tutorial outlines the process of setting up a **CI/CD pipeline** using **Jenkins**, **Maven**, **Docker**, **SonarQube**, and **Elastic Kubernetes Service (EKS)**. This pipeline will be triggered by changes to the application repository in **GitHub**, performing tasks like **code building**, **testing**, and **deployment**.

---

## 1. **Install and Configure Jenkins Master & Agent**

### Jenkins Master Setup:
1. **Launch EC2 Instance for Jenkins Master:**
   - Create a new EC2 instance in AWS.
   - Use **Ubuntu** as the OS and **t2.micro** as the instance type.
   - Allow port **8080** for Jenkins in security settings.
   
2. **SSH into the Instance:**
   - SSH into the Jenkins Master EC2 instance.

3. **System Updates & Hostname:**
   ```bash
   sudo apt-get update
   sudo hostnamectl set-hostname Jenkins-Master
   sudo reboot
  
4. **Install Java & Jenkins:**

Install Java (required for Jenkins) and Jenkins following the official Jenkins installation guide.

```bash
sudo apt install openjdk-11-jdk
sudo apt install wget
wget -q -O - https://pkg.jenkins.io/keys/jenkins.io.key | sudo tee /etc/apt/trusted.gpg.d/jenkins.asc
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable/ / > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
5. **Access Jenkins UI**

Access Jenkins by navigating to `http://<Jenkins_Machine_IP>:8080`.

6. **Create Admin User in Jenkins**

Create an admin user in the Jenkins setup wizard (e.g., username: `clouduser`).
### Jenkins Agent Setup

#### Launch EC2 Instance for Jenkins Agent:
1. Create another EC2 instance named `Jenkins-Agent` and install Java and Docker.

2. **Install Docker:**
```bash
sudo apt install docker.io
sudo usermod -aG docker $USER
```
3. **Set Up SSH for Communication:**
- Generate SSH keys on Jenkins Master and copy the public key to the Jenkins Agent.
4. **Add Jenkins Agent Node:**
- In Jenkins, add the Agent node and configure it to communicate via SSH.

---

## 2. Integrate Maven and GitHub with Jenkins

1. **Install Maven and GitHub Plugins:**
- Install Maven Integration and GitHub plugins from Jenkins Plugin Manager.

2. **Configure Maven and Java in Jenkins:**
- Go to Manage Jenkins > Global Tool Configuration.
- Configure Maven (e.g., Maven 3.8.1) and Java (e.g., Java 17).

3. **Add GitHub Credentials:**
- Go to Manage Jenkins > Manage Credentials and add GitHub credentials (username and personal access token).

4. **Create the Jenkins Pipeline Script (Jenkinsfile):**
- In your GitHub repository, create a Jenkinsfile.
- Define the Jenkins agent and tools (Java 17, Maven).
- Create the following pipeline stages:
  - Clean workspace.
  - Checkout code from GitHub.
  - Build using Maven.
  - Test using Maven.

5. **Create Jenkins Job:**
- Create a Jenkins job and configure it to run the pipeline.
  
---

## 3. Set Up and Integrate SonarQube for Static Code Analysis
### SonarQube Setup:
1. **Launch EC2 Instance for SonarQube:**
   
- Launch an EC2 instance and install PostgreSQL and SonarQube.
  
2. **Configure Database & Start SonarQube:**
   
- Set up SonarQube with PostgreSQL, adjust memory settings, and start SonarQube.
  
3. **Access SonarQube UI:**

- Access SonarQube via http://<SonarQube_IP>:9000.
  
4. **Create Admin Account in SonarQube.**

#### SonarQube Integration with Jenkins:

1. **Generate SonarQube Token:**

- Generate a token from SonarQube and add it to Jenkins credentials.
2. **Install SonarQube Scanner Plugin:**

- Install SonarQube Scanner plugin in Jenkins.
3. **Configure SonarQube in Jenkins:**

- Go to Manage Jenkins > Configure System and add SonarQube server details.
4. **Add SonarQube Analysis in Jenkins Pipeline:**

- Add a SonarQube stage to analyze the code.
5. **Configure Webhook in SonarQube to notify Jenkins about analysis results.**

---

##4.Docker Integration: Build & Push Docker Images
Docker Setup:
1. **Install Docker Plugins in Jenkins:**

 - Install Docker Commons and Docker Pipeline plugins.
2. **Add Docker Hub Credentials:**

 - Add Docker Hub credentials in Jenkins.
3. **Define Environment Variables in the Jenkins pipeline for application name, Docker Hub username, etc.**

4. **Add Docker Image Build & Push Stage:**

 - In the Jenkins pipeline, add a stage to build the Docker image and push it to Docker Hub.

---

## 5. Trivy Scan & Cleanup Docker Artifacts
1. **Trivy Setup:**
 - Install Trivy in Jenkins to scan Docker images for vulnerabilities.

 - Add Trivy Scan Stage in the pipeline to scan the latest Docker image.

2. **Cleanup Artifacts:**
 - Cleanup Stage to remove any build artifacts and Docker images after the job completes.

---

## 6. Set Up Kubernetes with EKS
**Create EKS Bootstrap Server:**

  - Launch EC2 Instance for the EKS bootstrap server.
  - Install AWS CLI, kubectl, and eksctl on the bootstrap server.
  - Configure AWS CLI with aws configure.
   
Create EKS Cluster:

Run the following command to create an EKS cluster:

```bash
eksctl create cluster --name my-cluster --region us-west-2 --nodegroup-name standard-workers --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 4 --managed
```

Verify EKS cluster nodes:

```bash
kubectl get nodes
```
---

## 7. Continuous Deployment with ArgoCD on EKS

- Install ArgoCD.
- Create Namespace for ArgoCD.
  
  ```bash kubectl create namespace argocd ```
  
- Install ArgoCD using YAML configuration.
  
   ```bash kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml ```
  
- Expose ArgoCD API Server externally using a LoadBalancer.
  
  ```bash kubectl expose deployment argocd-server --type=LoadBalancer --name=argocd-server -n argocd ```
  
- Extract ArgoCD Admin Password.
  
  ```bash kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo ```
  
- Login to ArgoCD and change the password.
- Add EKS Cluster to ArgoCD.
  
  ```bash argocd cluster add <EKS_CLUSTER_NAME> --name virtualtechbox-eks-cluster ```

---

## 8. GitOps Setup with ArgoCD and GitHub Repository

 - Add GitHub Repository containing Kubernetes manifests to ArgoCD via UI.
 - Create Application in ArgoCD for deployment, specifying the GitHub repo and EKS as the destination.
 - Verify Deployment by checking Kubernetes pods.
 - Verify External DNS to access the deployed application.

---

## 9. Automate Deployment Using Jenkins CI/CD

 - Create Jenkins CD Job for continuous deployment, triggered by GitHub repository changes.
 - Configure Jenkinsfile in the GitOps repository to handle deployment.
 - Test the Pipeline by making a commit to the GitHub repository, triggering the Jenkins CI job, which will deploy changes via ArgoCD.
 - Verify Deployment by checking the application URL and confirming changes.

---

             This setup provides a comprehensive CI/CD pipeline with Jenkins, Maven, Docker, SonarQube, and Kubernetes (EKS), allowing automated build, test, and deployment processes with integrated code quality analysis and security scanning.






