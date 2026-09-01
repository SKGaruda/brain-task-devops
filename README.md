# Brain Tasks App – AWS DevOps CI/CD Deployment

## 📌 Project Overview

This project demonstrates an end-to-end DevOps implementation for deploying a containerized web application to Amazon EKS using AWS managed services and Kubernetes.

The application is containerized using Docker, stored in Amazon Elastic Container Registry (ECR), and deployed to an Amazon EKS cluster using Kubernetes manifests.

A complete CI/CD pipeline is implemented using AWS CodePipeline and AWS CodeBuild. Whenever changes are pushed to the GitHub repository, CodePipeline triggers CodeBuild to build the Docker image, push it to ECR, and deploy the latest application version to EKS.

### Application Repository

GitHub Repository:

https://github.com/SKGaruda/brain-task-devops

---

# 🏗️ Architecture

```text
                         Developer
                             │
                             │ git push
                             ▼
                    ┌─────────────────┐
                    │     GitHub      │
                    │ Source Control  │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │  AWS CodePipeline│
                    │     Source      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   AWS CodeBuild │
                    │                 │
                    │ Docker Build    │
                    │ ECR Push        │
                    │ kubectl Deploy  │
                    └───────┬─────────┘
                            │
              ┌─────────────┴─────────────┐
              │                           │
              ▼                           ▼
      ┌─────────────────┐        ┌─────────────────┐
      │   Amazon ECR    │        │   Amazon EKS    │
      │                 │        │                 │
      │ Docker Image    │        │ Kubernetes      │
      │ Repository      │        │ Cluster         │
      └─────────────────┘        └────────┬────────┘
                                         │
                                         ▼
                               ┌──────────────────┐
                               │ Kubernetes       │
                               │ Deployment       │
                               │ brain-task-app   │
                               └────────┬─────────┘
                                        │
                                        ▼
                               ┌──────────────────┐
                               │ Kubernetes       │
                               │ LoadBalancer     │
                               │ Service          │
                               └────────┬─────────┘
                                        │
                                        ▼
                                  Internet Users
```

---

# 🛠️ Technologies Used

| Category            | Technology        |
| ------------------- | ----------------- |
| Source Control      | Git / GitHub      |
| Cloud Platform      | AWS               |
| Containerization    | Docker            |
| Container Registry  | Amazon ECR        |
| Kubernetes          | Kubernetes        |
| Kubernetes Platform | Amazon EKS        |
| CI/CD               | AWS CodePipeline  |
| Build Automation    | AWS CodeBuild     |
| Monitoring          | Amazon CloudWatch |
| Application Server  | Nginx             |
| Application Port    | 3000              |
| Infrastructure      | AWS EKS           |
| Deployment          | Kubernetes YAML   |

---

# 📁 Project Structure

```text
brain-task-devops/
│
├── Dockerfile
├── nginx.conf
├── buildspec.yml
├── README.md
│
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
│
└── dist/
    └── application build files
```

---

# 🚀 Project Implementation

## Step 1 – Clone the Application

Clone the original application repository:

```bash
git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git
```

Navigate to the project directory:

```bash
cd Brain-Tasks-App
```

---

# Step 2 – Install Dependencies

Install the required application dependencies:

```bash
npm install
```

---

# Step 3 – Build the Application

Build the frontend application:

```bash
npm run build
```

This generates the production build files in the `dist/` directory.

---

# Step 4 – Dockerize the Application

A Dockerfile was created to package the application into a container image.

The application is served using Nginx.

Example Dockerfile:

```dockerfile
FROM nginx:alpine

RUN rm -rf /usr/share/nginx/html/*

COPY dist/ /usr/share/nginx/html/

COPY nginx.conf /etc/nginx/conf.d/default.conf
```

The application is configured to run on port 3000.

---

# Step 5 – Build Docker Image

Build the Docker image:

```bash
docker build -t brain-task-app .
```

Verify the image:

```bash
docker images
```

---

# Step 6 – Run Application Locally

Run the container:

```bash
docker run -d \
  --name brain-task-app \
  -p 3000:3000 \
  brain-task-app:latest
```

Verify the running container:

```bash
docker ps
```

Open the application:

```text
http://localhost:3000
```

---

# Step 7 – Create Amazon ECR Repository

Create an ECR repository:

```bash
aws ecr create-repository \
  --repository-name brain-task-app \
  --region ap-south-1
```

Verify the repository:

```bash
aws ecr describe-repositories \
  --repository-names brain-task-app \
  --region ap-south-1
```

---

# Step 8 – Authenticate Docker with ECR

Retrieve the AWS account ID:

```bash
aws sts get-caller-identity
```

Authenticate Docker:

```bash
aws ecr get-login-password \
  --region ap-south-1 | \
docker login \
  --username AWS \
  --password-stdin \
  874841217466.dkr.ecr.ap-south-1.amazonaws.com
```

---

# Step 9 – Tag Docker Image

Tag the image:

```bash
docker tag brain-task-app:latest \
874841217466.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
```

---

# Step 10 – Push Docker Image to ECR

Push the image:

```bash
docker push \
874841217466.dkr.ecr.ap-south-1.amazonaws.com/brain-task-app:latest
```

Verify:

```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

---

# Step 11 – Create Amazon EKS Cluster

An EKS cluster was created in the AWS Mumbai region:

```text
Region: ap-south-1
Cluster: brain-task-eks
```

Verify the cluster:

```bash
aws eks describe-cluster \
  --name brain-task-eks \
  --region ap-south-1 \
  --query "cluster.status" \
  --output text
```

Expected:

```text
ACTIVE
```

---

# Step 12 – Configure kubectl

Update the local kubeconfig:

```bash
aws eks update-kubeconfig \
  --region ap-south-1 \
  --name brain-task-eks
```

Verify connectivity:

```bash
kubectl get nodes
```

---

# Step 13 – Kubernetes Deployment

The application is deployed using:

```text
k8s/deployment.yaml
```

The Deployment manages the application Pods.

Apply the deployment:

```bash
kubectl apply -f k8s/deployment.yaml
```

Verify:

```bash
kubectl get deployments
```

Expected deployment:

```text
brain-task-app
```

Check Pods:

```bash
kubectl get pods
```

---

# Step 14 – Kubernetes Service

The application is exposed through:

```text
k8s/service.yaml
```

The service type is:

```text
LoadBalancer
```

Apply the service:

```bash
kubectl apply -f k8s/service.yaml
```

Verify:

```bash
kubectl get svc brain-task-service
```

The service forwards:

```text
Port 80 → TargetPort 3000
```

---

# Step 15 – Verify Application

Check the service:

```bash
kubectl get svc brain-task-service
```

Example:

```text
NAME                 TYPE           EXTERNAL-IP
brain-task-service   LoadBalancer   <AWS-LOAD-BALANCER-DNS>
```

The application can then be accessed using the LoadBalancer DNS name.

---

# 🔄 CI/CD Pipeline

The CI/CD pipeline is implemented using:

```text
GitHub
   ↓
AWS CodePipeline
   ↓
AWS CodeBuild
   ↓
Docker Build
   ↓
Amazon ECR
   ↓
Amazon EKS
   ↓
Kubernetes Deployment
   ↓
Kubernetes LoadBalancer
```

---

# 🏗️ AWS CodeBuild

The CodeBuild project is:

```text
brain-task-build
```

CodeBuild performs the following operations:

1. Retrieves source code from GitHub.
2. Retrieves AWS account information.
3. Validates the ECR repository.
4. Authenticates Docker with Amazon ECR.
5. Installs kubectl.
6. Updates kubeconfig for the EKS cluster.
7. Builds the Docker image.
8. Tags the Docker image.
9. Pushes the image to ECR.
10. Applies Kubernetes Deployment.
11. Applies Kubernetes Service.
12. Verifies Kubernetes rollout.
13. Displays Pod and Service information.

---

# 📄 Buildspec

The CI/CD commands are defined in:

```text
buildspec.yml
```

Main phases:

```text
PRE_BUILD
    ↓
Authenticate with ECR
    ↓
Configure kubectl
    ↓
BUILD
    ↓
Build Docker image
    ↓
POST_BUILD
    ↓
Push image to ECR
    ↓
Deploy to EKS
    ↓
Verify deployment
```

---

# 🔐 IAM and EKS Access

The CodeBuild service role is configured with the required permissions to:

* Access Amazon ECR
* Authenticate with ECR
* Push Docker images
* Describe the EKS cluster
* Access EKS through the configured EKS access entry
* Execute Kubernetes deployment commands

Least-privilege IAM permissions should be preferred when implementing the project in a production environment.

---

# 📊 Monitoring

AWS CloudWatch Logs are used to monitor CodeBuild execution.

CodeBuild logs include:

* Source download
* Docker build
* ECR authentication
* ECR image push
* kubectl execution
* Kubernetes deployment
* Kubernetes rollout status

CloudWatch log group:

```text
/aws/codebuild/brain-task-build
```

Kubernetes application logs can be viewed using:

```bash
kubectl logs <pod-name>
```

---

# 🧪 Useful Verification Commands

Check EKS cluster:

```bash
aws eks describe-cluster \
  --name brain-task-eks \
  --region ap-south-1 \
  --query "cluster.status"
```

Check nodes:

```bash
kubectl get nodes
```

Check deployments:

```bash
kubectl get deployments
```

Check Pods:

```bash
kubectl get pods
```

Check service:

```bash
kubectl get svc
```

Check rollout:

```bash
kubectl rollout status deployment/brain-task-app
```

Check application logs:

```bash
kubectl logs <pod-name>
```

Check ECR images:

```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

---

# 🧹 Cleanup / Resource Deletion

AWS resources should be deleted after testing to avoid unnecessary charges.

## Delete Kubernetes Service

```bash
kubectl delete service brain-task-service
```

This removes the AWS LoadBalancer associated with the Kubernetes Service.

## Delete Kubernetes Deployment

```bash
kubectl delete deployment brain-task-app
```

## Delete EKS Access Entry

If required:

```bash
aws eks delete-access-entry \
  --cluster-name brain-task-eks \
  --principal-arn arn:aws:iam::874841217466:role/service-role/codebuild-brain-task-service-role \
  --region ap-south-1
```

## Delete EKS Cluster

Only after deleting the Kubernetes resources:

```bash
aws eks delete-cluster \
  --name brain-task-eks \
  --region ap-south-1
```

Verify:

```bash
aws eks describe-cluster \
  --name brain-task-eks \
  --region ap-south-1
```

## Delete ECR Repository

If the project is no longer required:

```bash
aws ecr delete-repository \
  --repository-name brain-task-app \
  --region ap-south-1 \
  --force
```

## Delete CodeBuild Project

```bash
aws codebuild delete-project \
  --name brain-task-build \
  --region ap-south-1
```

## Delete CodePipeline

```bash
aws codepipeline delete-pipeline \
  --name brain-task-pipeline \
  --region ap-south-1
```

---

# 🎯 Key DevOps Concepts Demonstrated

* Git version control
* GitHub repository management
* Docker containerization
* Docker image creation
* Amazon ECR
* Amazon EKS
* Kubernetes Deployments
* Kubernetes Services
* Kubernetes LoadBalancer
* kubectl
* AWS CodeBuild
* AWS CodePipeline
* CI/CD automation
* IAM roles and permissions
* EKS access entries
* CloudWatch Logs
* AWS CLI
* Linux shell commands
* Container image lifecycle
* Automated Kubernetes deployment

---

# 📸 Screenshots

The following screenshots can be added to document the implementation:

1. GitHub repository
2. Docker image
3. ECR repository
4. ECR image
5. EKS cluster ACTIVE status
6. EKS nodes
7. Kubernetes Pods
8. Kubernetes Deployment
9. Kubernetes LoadBalancer Service
10. Application running through LoadBalancer
11. CodeBuild successful execution
12. CodePipeline successful execution
13. CloudWatch CodeBuild logs

---

# 🏆 Project Outcome

Successfully implemented an automated CI/CD pipeline for a containerized web application.

The solution automates the complete deployment lifecycle:

```text
Code Change
    ↓
GitHub
    ↓
CodePipeline
    ↓
CodeBuild
    ↓
Docker Build
    ↓
ECR
    ↓
EKS
    ↓
Kubernetes
    ↓
LoadBalancer
    ↓
Application
```

This project demonstrates practical experience with AWS cloud services, Docker, Kubernetes, CI/CD automation, IAM, and monitoring.
