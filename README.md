---
# DevOps CI/CD Pipeline Using Jenkins and Argo CD
![Tests](https://github.com/user-attachments/assets/90ffedf7-1cc6-4637-a713-06179f3e1978)
---

This project demonstrates a CI/CD pipeline that automates the process of building, testing, and deploying a Dockerized application. Jenkins handles the Continuous Integration (CI) by performing build and static code analysis,
while Argo CD manages Continuous Deployment (CD) by automatically updating Kubernetes clusters. This setup improves code reliability, reduces deployment time, and ensures applications are always up to date.
  ##  Table of Contents:
```markdown
1. Introduction
2. Workflow
3. Prerequisites
4. Pipeline Configuration
	- Jenkins Setup on EC2
5. Step-by-Step Guide
	- Generate Artifacts Using Maven
	- SonarQube Configuration
	- Docker Image Creation and Management
	- Argo CD Setup
6. Final Deployment
7. Future Improvements
8. Conclusion
```


## Workflow
The pipeline follows this process:
1. **Source Code Commit**: A developer pushes code to the Git repository, which triggers the CI/CD pipeline. This automates the integration of changes into the production environment, improving collaboration and code quality.
2. **Jenkins CI**: Jenkins triggers the build, runs tests, and performs static code analysis using Maven and SonarQube.
3. **Docker Image Creation**: If all tests pass, Jenkins packages the application into a Docker image and pushes it to DockerHub, ensuring the application can be run in a consistent environment.
4. **Argo CD Deployment**: Argo CD pulls the updated Docker image and deploys it to a Kubernetes cluster, keeping the application up to date without manual intervention.

## Prerequisites
- AWS account with an EC2 instance running Jenkins.
- Kubernetes cluster (Minikube, microk8s, etc.) running locally.
- DockerHub account for storing Docker images.
- GitHub repository with your application code and manifests.

## Pipeline Configuration

### Jenkins Setup on EC2
1. **Install Jenkins on EC2**:
    ```bash
    sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
    echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
    sudo apt-get update
    sudo apt-get install jenkins
    sudo systemctl enable jenkins
    sudo systemctl start jenkins
    sudo systemctl status jenkins
    ```
2. Place the `Jenkinsfile` at the root of your repository.
3. Use the initial admin password to configure Jenkins:
    ```bash
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
   Access Jenkins at `<EC2_IP>:8080` and configure the Git repository URL and branch.
   Install docker pipeline plugin and sonarqube scanner plugin.
4. Update DockerHub and Github credentials

![Screenshot from 2024-10-04 19-12-20](https://github.com/user-attachments/assets/c33d50f2-d6a4-4e4c-8fbd-3d4b455dca34)
![Screenshot from 2024-10-04 19-14-27](https://github.com/user-attachments/assets/35c4d6fb-4fca-43dd-9c4f-defe767a0ca0)



## Step-by-Step Guide

### Generate Artifacts Using Maven
1. Build the application using the command below. This will compile your Java code, run tests, and package the code into a .jar file.
    ```bash
    mvn clean package
    ```
2. To verify your application locally, run the jar file using the following command:
    ```bash
    java -jar target/spring-boot-web.jar
    ```
   Access the application at `localhost:8080`.

3. To run as a Docker container, follow these steps:
   
   Build a Docker image:
    ```bash
    docker build -t jenkins-cicd-pipeline:v1 .
    ```
    Run the container:
    ```bash
    docker run -d -p 8010:8080 -t jenkins-cicd-pipeline:v1
    ```
   Access the application at `localhost:8010`.

### SonarQube Configuration
1. **Install SonarQube on EC2**:
    ```bash
    sudo apt install unzip
    sudo adduser sonarqube
    sudo su - sonarqube
    wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
    unzip sonarqube-9.4.0.54424.zip
    chmod -R 755 sonarqube-9.4.0.54424
    chown -R sonarqube:sonarqube sonarqube-9.4.0.54424
    cd sonarqube-9.4.0.54424/bin/linux-x86-64/
    sudo ./sonar.sh start
    ```
   Access SonarQube at `http://<EC2_IP>:9000`.
![Screenshot from 2024-10-04 19-01-17](https://github.com/user-attachments/assets/ac88549d-2138-4522-9c34-68826504dd6d)


   **Note:** SonarQube 9.4 is not fully compatible with Java versions beyond 11, so you need to install and switch to Java 11:
    ```bash
    sudo apt update
    sudo apt install openjdk-11-jdk
    sudo update-alternatives --config java
    ```

3. **SonarQube and Jenkins Integration**: Generate a token in SonarQube and add it to Jenkins credentials as a secret text.
![Screenshot from 2024-10-04 19-12-20](https://github.com/user-attachments/assets/b8827aad-00ea-4891-b0a5-a476069684b4)



### Docker Image Creation and Management
1. **Install Docker on EC2**:
    ```bash
    sudo apt install docker.io
    ```
2. **Push Docker Image to DockerHub**:
    ```bash
    docker build -t your_image_name .
    docker push your_dockerhub_username/your_image_name:tag
    ```
### Build the pipeline 
**If build fails**:
Enable the "Wipe workspace before build starts" option on jenkins or
Wipe the workspace manually:
```bash
sudo rm -rf /var/lib/jenkins/workspace/jenkins-pipeline/*
```
**Note:** Restart Jenkins before triggering another build

**Snap of the deployment.yml file before the build**:
![Screenshot from 2024-10-04 20-42-13](https://github.com/user-attachments/assets/f2fb77a5-920c-4da9-ac20-003ca89f27e1)



**After the build the replaceImageTag is updated with build number**:
![Screenshot from 2024-10-04 20-45-54](https://github.com/user-attachments/assets/e34259c7-351d-4d85-8690-32766245c1c1)



**The change is pushed automatically to github along with a commit message**: 
![Screenshot from 2024-10-04 20-46-18](https://github.com/user-attachments/assets/38687ca6-4cce-48a1-8799-d154c5f1de9a)



**Snap of the successful build continouous integration part**:
![Screenshot from 2024-10-04 20-47-35](https://github.com/user-attachments/assets/fd5b1339-acc5-4b63-8974-63878c8f0b7b)

**Snap of image being updated and pushed to dockerhub along with updated tag number**:
![Screenshot from 2024-10-04 20-50-08](https://github.com/user-attachments/assets/e0d4d363-751c-47bf-9e37-2557faff433e)
---
---



**Up until now we are done with the Continous Integration part,now for the continuous delivery portion we will use Argo CD**
### Argo CD deployment

Argo CD ensures continuous synchronization of the Kubernetes cluster with the latest application image. This setup provides:
- **Automatic rollbacks** in case of failure.
- **Version control for deployments** by pulling updates directly from GitHub.
- **Security** by maintaining the desired state of the application and automatically reverting changes made by unauthorized users.


  ```bash
    kubectl apply -f argocd-basic.yml
  ```
    
  To Access the Argo CD UI from the browser using the URL provided by Minikube run:
   ```bash
   minikube service list
   ```
  ![Screenshot from 2024-10-04 21-04-57](https://github.com/user-attachments/assets/c6275c52-b69f-4087-b20c-7cfdc8f68167)

    
   ![Screenshot from 2024-10-04 21-05-14](https://github.com/user-attachments/assets/82624a6c-546a-4a32-8674-eff99094c13d)

   Username is admin and 
   password can be accessed by running:
   ```bash
   kubectl get secret
   kubectl edit secret example-argocd-cluster
   ```
   copy the secret and run
   ```bash
   echo <secret> | base64 -d
   ```
 Use the password to login to Argo CD

### Deploy the application in Argo CD:
   - Create an Argo CD app with your GitHub repository as the source and the `deployment.yml` file for deployment.
     ![Screenshot from 2024-10-04 22-29-12](https://github.com/user-attachments/assets/d9f229d7-587c-4e89-b9de-d2a334362d6d)
Notice how the application has been automatically created on k8s cluster

## Final Deployment

Once the pipeline executes successfully:
- Jenkins updates the deployment configuration in your GitHub repository by changing the Docker image tag.
- Argo CD monitors the GitHub repository and automatically applies these changes to the Kubernetes cluster.
- **This ensures that your application is always running the latest version, reducing downtime and manual interventions.**


## Future Improvements
- **Implementing Monitoring**: Using tools like Prometheus and Grafana for monitoring your Kubernetes cluster.
- **Scaling the Pipeline**: Adding stages for integration with cloud services like AWS S3, databases, etc.


## Conclusion
This CI/CD pipeline automates the entire process of building, testing, and deploying applications. It uses Jenkins for Continuous Integration and Argo CD for Continuous Deployment, ensuring that your Kubernetes clusters always run the latest, stable versions of your applications.

---
