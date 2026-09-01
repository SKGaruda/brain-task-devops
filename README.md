# Brain Tasks -- End-to-End AWS DevOps Project

[![Brain Tasks Application](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)

## 📌 Project Overview

**Brain Tasks** is a containerized web application deployed on AWS using a fully automated DevOps workflow.

The project demonstrates how source code moves from a GitHub repository through Docker image creation, Amazon ECR, AWS CodePipeline/CodeBuild, Amazon EKS, Kubernetes, and finally to a publicly accessible LoadBalancer endpoint — with no manual deployment steps once code is pushed.

Build and deployment execution is monitored through **Amazon CloudWatch Logs**.

---

## 🎯 Project Objectives

- Containerize the Brain Tasks application using Docker, served through Nginx.
- Build and push versioned Docker images to Amazon ECR.
- Provision an Amazon EKS cluster for container orchestration.
- Implement CI/CD automation using AWS CodePipeline and AWS CodeBuild.
- Deploy the application to EKS using Kubernetes manifests.
- Manage application replicas using a Kubernetes Deployment.
- Expose the application through a Kubernetes LoadBalancer service.
- Monitor build and deployment activity through CloudWatch Logs.
- Demonstrate a working, source-triggered DevOps pipeline end to end.

---

## 🏗️ Architecture

```
                    Developer
                        |
                        v
                 git push (GitHub)
                        |
                        v
               AWS CodePipeline (Source)
                        |
                        v
                 AWS CodeBuild
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
   Docker Build     ECR Push     kubectl Deploy
                        |             |
                        v             v
                 Amazon ECR      Amazon EKS
                                      |
                             +--------+--------+
                             |                 |
                        Deployment          Service
                       brain-task-app     LoadBalancer
                             |                 |
                          2 Pods               v
                             |          Public AWS ELB
                             +----------------> Brain Tasks App
                                                |
                                                v
                                         End User Browser

Monitoring:
AWS CodeBuild / CodePipeline
        |
        v
Amazon CloudWatch Logs
        |
        v
Build phase status, Docker build/push logs, deployment output
```

---

## 🛠️ Technology Stack

| Category               | Technology                       |
| ----------------------- | --------------------------------- |
| Application             | Brain Tasks Web App               |
| Cloud                    | AWS                                |
| Containerization         | Docker                             |
| Image Registry           | Amazon ECR                         |
| CI/CD                    | AWS CodePipeline + AWS CodeBuild   |
| Container Orchestration  | Kubernetes                         |
| Kubernetes Platform      | Amazon EKS                         |
| Load Balancing           | Kubernetes LoadBalancer / AWS ELB  |
| Monitoring               | Amazon CloudWatch Logs             |
| Application Server       | Nginx                              |
| Application Port         | 3000                                |
| Version Control          | Git / GitHub                       |

---

# 🔄 End-to-End DevOps Workflow

## Phase 1 -- Application Validation

The application was first cloned, built, and verified locally to confirm it worked correctly before containerization.

```bash
git clone https://github.com/Vennilavanguvi/Brain-Tasks-App.git
cd Brain-Tasks-App
npm install
npm run build
```

[![Application Running Locally](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)

---

## Phase 2 -- Docker Containerization

A Dockerfile was created to package the built application into an Nginx-served container image.

```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY dist/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf
```

```bash
docker build -t brain-task-app .
docker run -d --name brain-task-app -p 3000:3000 brain-task-app:latest
```

The container was verified running locally on `http://localhost:3000`.

---

# ☁️ AWS Infrastructure

## Phase 3 -- Amazon ECR

An ECR repository was created to store versioned Docker images.

```bash
aws ecr create-repository \
  --repository-name brain-task-app \
  --region ap-south-1
```

Repository verified via the AWS CLI:

[![ECR Repository Details](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/ECR-REPO-DETAILS.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/ECR-REPO-DETAILS.jpg)

Image pushed and confirmed in the ECR console:

[![ECR Repo Created Successfully](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/AWS-CODEBUILD-ECR-REPO-CREATED-SUCCESSFULL.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/AWS-CODEBUILD-ECR-REPO-CREATED-SUCCESSFULL.jpg)

---

## Phase 4 -- Amazon EKS Cluster

An Amazon EKS cluster named `brain-task-eks` was created in the `ap-south-1` (Mumbai) region using `eksctl`, including a managed node group with 2 nodes.

```bash
aws eks describe-cluster \
  --name brain-task-eks \
  --region ap-south-1 \
  --query "cluster.status" \
  --output text
```

Cluster creation log — VPC-CNI, kube-proxy, and CoreDNS add-ons installed, managed node group provisioned, 2 nodes ready:

[![EKS Cluster Creation Successful](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/EKS-CLUSTER-CREATION-SUCCESSFULL.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/EKS-CLUSTER-CREATION-SUCCESSFULL.jpg)

Cluster status confirmed as `ACTIVE`:

[![EKS Cluster Active](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/EKS-CLUSTER-ACTIVE.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/EKS-CLUSTER-ACTIVE.jpg)

kubeconfig was updated to connect kubectl to the cluster:

```bash
aws eks update-kubeconfig --region ap-south-1 --name brain-task-eks
kubectl get nodes
```

---

# ⚙️ AWS CodePipeline + CodeBuild CI/CD

## Phase 5 -- Pipeline and Build Project

A CodePipeline (`brain-task-pipeline`) was configured with GitHub as the source stage and AWS CodeBuild (`brain-task-build`) as the build stage. Every push to the GitHub repository triggers the pipeline automatically.

CodeBuild performs the following, defined in `buildspec.yml`:

1. Retrieves source code from GitHub.
2. Retrieves AWS account information.
3. Validates the ECR repository.
4. Authenticates Docker with Amazon ECR.
5. Installs `kubectl` and updates kubeconfig for the EKS cluster.
6. Builds and tags the Docker image.
7. Pushes the image to ECR.
8. Applies the Kubernetes Deployment and Service manifests.
9. Verifies the Kubernetes rollout.
10. Displays Pod and Service status.

Pipeline triggered and running:

[![CodePipeline Created and Trigger Initiated](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEPIPELINE-CREATED-AND-TRIGGER-INITIATED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEPIPELINE-CREATED-AND-TRIGGER-INITIATED.jpg)

Pipeline completed — Source and Build stages both succeeded:

[![CodePipeline Completed Successfully](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEPIPELINE-COMPLETED-SUCCEEDED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEPIPELINE-COMPLETED-SUCCEEDED.jpg)

CodeBuild build status:

[![CodeBuild Succeeded](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-SUCCEEDED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-SUCCEEDED.jpg)

---

## Phase 6 -- Build Execution (CloudWatch Logs)

Each CodeBuild run streams logs to CloudWatch under the log group `/aws/codebuild/brain-task-build`, covering source download, Docker build, ECR authentication and push, and Kubernetes deployment.

[![CodeBuild Logs — Docker Build & Tag](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS1.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS1.jpg)

[![CodeBuild Logs — Push to ECR](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS2.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS2.jpg)

[![CodeBuild Logs — Post-Build Phase](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS3.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS3.jpg)

[![CodeBuild Logs — Artifact Upload](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS4.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS4.jpg)

Final combined status across CodePipeline, CodeBuild, and CloudWatch:

[![Final CodePipeline/CodeBuild/CloudWatch Status](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/FINAL-CODEPIPELINE-CODEBUILD-CLOUDWATCH-LOGS-STATUS.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/FINAL-CODEPIPELINE-CODEBUILD-CLOUDWATCH-LOGS-STATUS.jpg)

---

# ☸️ Kubernetes Deployment and Exposure

## Phase 7 -- Kubernetes Deployment

The application is deployed to EKS via `k8s/deployment.yaml`, applied automatically by CodeBuild as part of the pipeline.

```bash
kubectl apply -f k8s/deployment.yaml
kubectl get deployments
kubectl get pods
```

Application confirmed running on the EKS cluster:

[![Application Running on EKS Cluster](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-EKS-CLUSTER.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-EKS-CLUSTER.jpg)

---

## Phase 8 -- Kubernetes LoadBalancer

The application is exposed through a Kubernetes `LoadBalancer` service (`brain-task-service`), defined in `k8s/service.yaml` and applied by CodeBuild.

```bash
kubectl apply -f k8s/service.yaml
kubectl describe svc brain-task-service
```

### AWS LoadBalancer Details

| Property           | Value                                                              |
| ------------------- | -------------------------------------------------------------------|
| Kubernetes Service  | `brain-task-service`                                                |
| Service Type        | `LoadBalancer`                                                      |
| AWS Region          | `ap-south-1`                                                        |
| Port                | `80`                                                                 |
| Target Port         | `3000`                                                               |
| NodePort            | `32338`                                                              |
| Load Balancer DNS   | `afd310c2b685a4ff8944108248ca46b4-386370546.ap-south-1.elb.amazonaws.com` |

[![LoadBalancer Service Details](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/LOADBALANCER.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/LOADBALANCER.jpg)

The application was verified accessible through the LoadBalancer DNS in a browser, confirming the full pipeline — from GitHub push to a publicly reachable application — works end to end.

[![Application Running via CodeBuild/CodePipeline Deploy](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-CODEBUILD-CODEPIPELINE.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-CODEBUILD-CODEPIPELINE.jpg)

---

# 🔍 Validation Commands

## Check EKS Cluster
```bash
aws eks describe-cluster \
  --name brain-task-eks \
  --region ap-south-1 \
  --query "cluster.status" \
  --output text
```

## Check Nodes
```bash
kubectl get nodes
```

## Check Deployments
```bash
kubectl get deployments
```

## Check Pods
```bash
kubectl get pods
```

## Check Service
```bash
kubectl get svc brain-task-service
```

## Check Rollout
```bash
kubectl rollout status deployment/brain-task-app
```

## Check ECR Images
```bash
aws ecr list-images \
  --repository-name brain-task-app \
  --region ap-south-1
```

## Check Application Logs
```bash
kubectl logs <pod-name>
```

---

# 📸 Project Evidence

### 1. Application Running Locally
[![Local Application](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-LOCALLY.jpg)

### 2. ECR Repository Details (CLI)
[![ECR Repo Details](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/ECR-REPO-DETAILS.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/ECR-REPO-DETAILS.jpg)

### 3. ECR Repository Created and Image Pushed
[![ECR Repo Created](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/AWS-CODEBUILD-ECR-REPO-CREATED-SUCCESSFULL.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/AWS-CODEBUILD-ECR-REPO-CREATED-SUCCESSFULL.jpg)

### 4. EKS Cluster Creation Log
[![EKS Cluster Creation](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/EKS-CLUSTER-CREATION-SUCCESSFULL.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/EKS-CLUSTER-CREATION-SUCCESSFULL.jpg)

### 5. EKS Cluster Active Status
[![EKS Cluster Active](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/EKS-CLUSTER-ACTIVE.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/EKS-CLUSTER-ACTIVE.jpg)

### 6. CodePipeline Triggered
[![Pipeline Triggered](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEPIPELINE-CREATED-AND-TRIGGER-INITIATED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEPIPELINE-CREATED-AND-TRIGGER-INITIATED.jpg)

### 7. CodePipeline Completed
[![Pipeline Completed](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEPIPELINE-COMPLETED-SUCCEEDED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEPIPELINE-COMPLETED-SUCCEEDED.jpg)

### 8. CodeBuild Succeeded
[![CodeBuild Succeeded](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-SUCCEEDED.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-SUCCEEDED.jpg)

### 9. CodeBuild CloudWatch Logs (1–4)
[![Logs 1](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS1.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS1.jpg)
[![Logs 2](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS2.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS2.jpg)
[![Logs 3](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS3.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS3.jpg)
[![Logs 4](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/CODEBUILD-LOGS4.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/CODEBUILD-LOGS4.jpg)

### 10. Final Combined Status
[![Final Status](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/FINAL-CODEPIPELINE-CODEBUILD-CLOUDWATCH-LOGS-STATUS.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/FINAL-CODEPIPELINE-CODEBUILD-CLOUDWATCH-LOGS-STATUS.jpg)

### 11. Application Running on EKS Cluster
[![App on EKS](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-EKS-CLUSTER.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-EKS-CLUSTER.jpg)

### 12. LoadBalancer Service Details
[![LoadBalancer](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/LOADBALANCER.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/LOADBALANCER.jpg)

### 13. Application Running via CodePipeline/CodeBuild Deploy
[![App via Pipeline](https://github.com/SKGaruda/brain-task-devops/raw/main/screenshots/APPLICATION-RUNNING-CODEBUILD-CODEPIPELINE.jpg)](/SKGaruda/brain-task-devops/blob/main/screenshots/APPLICATION-RUNNING-CODEBUILD-CODEPIPELINE.jpg)

---

# 🔐 DevOps / Security Practices Demonstrated

- IAM roles used for CodeBuild instead of hard-coded AWS credentials.
- EKS access entries used to grant the CodeBuild service role cluster permissions.
- Docker image versioning through ECR.
- Kubernetes declarative configuration (Deployment + Service manifests).
- Replica-based application deployment.
- Fully automated CI/CD — no manual build or deploy steps once code is pushed.
- Centralized build/deploy logging through CloudWatch.
- Least-privilege IAM permissions recommended for production use.

---

# 🚀 Key Project Outcomes

```
Code Change
    ↓
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
    ↓
Public Application
```

The final application is deployed on Amazon EKS and accessible through an AWS LoadBalancer, with the entire build-to-deploy cycle automated through CodePipeline and CodeBuild, and monitored via CloudWatch Logs.

---

# 🧹 Cleanup / Cost Control

AWS resources should be removed after testing to avoid unnecessary charges.

```bash
kubectl delete service brain-task-service
kubectl delete deployment brain-task-app
aws eks delete-access-entry \
  --cluster-name brain-task-eks \
  --principal-arn arn:aws:iam::<account-id>:role/service-role/codebuild-brain-task-service-role \
  --region ap-south-1
aws eks delete-cluster --name brain-task-eks --region ap-south-1
aws ecr delete-repository --repository-name brain-task-app --region ap-south-1 --force
aws codebuild delete-project --name brain-task-build --region ap-south-1
aws codepipeline delete-pipeline --name brain-task-pipeline --region ap-south-1
```

Also verify no unnecessary EKS clusters, LoadBalancers, NAT Gateways, Elastic IPs, or EBS volumes are still running.

---

# ✅ Final Project Status

| Component               | Status                |
| ------------------------ | ---------------------- |
| Application               | ✅ Working              |
| Docker Image               | ✅ Built                |
| Amazon ECR                 | ✅ Image Pushed         |
| EKS Cluster                | ✅ Active               |
| EKS Nodes                  | ✅ Ready (2 nodes)      |
| CodePipeline                | ✅ Successful           |
| CodeBuild                   | ✅ Succeeded            |
| Kubernetes Deployment        | ✅ Running              |
| Kubernetes Pods               | ✅ Running              |
| LoadBalancer                   | ✅ Accessible           |
| CloudWatch Logs                 | ✅ Configured           |
| Project Documentation             | ✅ Completed            |

---

## ⭐ Final Architecture Summary

**Brain Tasks is a complete, self-triggered AWS DevOps pipeline covering:**

**GitHub → CodePipeline → CodeBuild → Docker → Amazon ECR → Amazon EKS → Kubernetes → LoadBalancer → CloudWatch**

---

# 📦 Final Submission Checklist

```
brain-task-devops/
├── README.md
├── Dockerfile
├── nginx.conf
├── buildspec.yml
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
└── screenshots/
```

### Evidence Included
- GitHub repository link
- README.md
- ECR repository/image
- EKS cluster and node evidence
- CodePipeline and CodeBuild successful execution
- CloudWatch build logs
- Kubernetes Deployment and Pods
- Kubernetes LoadBalancer service
- Verified LoadBalancer DNS
- Screenshots folder

### Final LoadBalancer Information

```
Service Type: Kubernetes LoadBalancer (AWS ELB)
Region: ap-south-1
Port: 80
Target Port: 3000
DNS: afd310c2b685a4ff8944108248ca46b4-386370546.ap-south-1.elb.amazonaws.com
```

The application returned a working response through the public LoadBalancer endpoint.