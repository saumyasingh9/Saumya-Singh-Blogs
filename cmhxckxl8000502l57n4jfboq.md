---
title: "Deploying a Full Microservices E-Commerce Application on AWS EKS"
datePublished: Thu Nov 13 2025 11:30:25 GMT+0000 (Coordinated Universal Time)
cuid: cmhxckxl8000502l57n4jfboq
slug: deploying-a-full-microservices-e-commerce-application-on-aws-eks
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1763032982341/749b0300-4259-4c26-939e-092c69c9831d.webp

---

### Introduction

In this blog, I’ll walk through how I deployed a **production-ready, microservices-based e-commerce platform** on **Amazon EKS (Elastic Kubernetes Service)** using a full **DevOps automation pipeline**.

The project integrates **Terraform** for Infrastructure as Code, **Jenkins** for CI/CD, **Argo CD** for GitOps-based deployment, **Amazon ECR** for container registry, and **Route 53 + ACM** for secure domain hosting with HTTPS.

The goal of this project was to simulate a **real-world enterprise-level deployment** — automating everything from infrastructure provisioning and container image building to monitoring and secure delivery.

---

## 🏗️ Architecture Overview

This e-commerce platform consists of **11 independent microservices**, each running in its own Docker container and deployed as separate Kubernetes deployments on EKS:

* **emailservice**
    
* **checkoutservice**
    
* **recommendationservice**
    
* **frontend**
    
* **paymentservice**
    
* **productcatalogservice**
    
* **cartservice**
    
* **loadgenerator**
    
* **currencyservice**
    
* **shippingservice**
    
* **adservice**
    

These microservices communicate through APIs and are managed through **Kubernetes deployments, services, and ingress** resources.

The complete infrastructure runs on **AWS**, and the DevOps automation stack looks like this:

| Layer | Tools / Services |
| --- | --- |
| **Infrastructure** | Terraform, AWS EC2, VPC, S3 |
| **Container Orchestration** | Amazon EKS, kubectl, eksctl, Helm |
| **CI/CD Automation** | Jenkins, Argo CD |
| **Container Registry** | Amazon ECR |
| **Domain & Security** | Route 53, AWS Certificate Manager (ACM) |
| **Monitoring** | Prometheus, Grafana |
| **Image Security** | Trivy |

---

## 🏗️ Step 1: Infrastructure Setup with Terraform

The first stage was provisioning all cloud resources in AWS using **Terraform modules** to maintain modularity and reusability.

1. **S3 Buckets for Terraform State**
    
    * Created remote S3 buckets to store Terraform state files securely.
        
    * Enabled DynamoDB-based state locking to prevent concurrent edits.
        
2. **Network and Compute Setup**
    
    * Provisioned a **VPC, subnets, and security groups**.
        
    * Created a **Jumphost EC2 instance** to access and manage the EKS cluster.
        
    * Verified connectivity and tool installation on EC2 (Jenkins, Docker, kubectl, eksctl, etc.).
        
3. **EKS Cluster Provisioning**
    
    * Deployed a **production-grade EKS cluster** using Terraform.
        
    * Managed cluster configuration with `aws eks update-kubeconfig` for easy `kubectl` access.
        

This Infrastructure as Code approach made the environment **reproducible, scalable, and version-controlled**.

---

## ⚙️ Step 2: Continuous Integration with Jenkins

After setting up the infrastructure, I configured **Jenkins on the EC2 jumphost** to automate build and deployment pipelines.

1. **Jenkins Setup**
    
    * Installed required plugins: Pipeline, Docker, Git, AWS CLI, Kubernetes.
        
    * Created credentials for GitHub PAT and AWS access keys.
        
2. **Terraform Automation Pipelines**
    
    * Created a Jenkins pipeline (`eks-terraform`) to automatically apply Terraform code to create the EKS cluster.
        
    * Another pipeline (`ecr-terraform`) for creating **Amazon ECR** repositories for each service.
        
3. **Docker Build & Push Pipelines**  
    For each of the 11 microservices, I built an individual Jenkins pipeline:
    
    * Clone the GitHub repository
        
    * Build the Docker image using the microservice’s `Dockerfile`
        
    * Run **Trivy** for image vulnerability scanning
        
    * Push the clean image to **Amazon ECR**
        

This ensured every code update resulted in an updated Docker image stored securely in ECR, ready for deployment.

---

## Step 3: Continuous Deployment with ArgoCD (GitOps)

Once all images were in ECR, I moved to the deployment phase using **ArgoCD**, a GitOps-based CD tool.

1. **Installing ArgoCD**
    
    * Created a dedicated `argocd` namespace in the cluster.
        
    * Installed ArgoCD using the official manifest files from GitHub.
        
    * Exposed the ArgoCD server using a **LoadBalancer service** to access it via a public endpoint.
        
2. **Access & Configuration**
    
    * Retrieved the initial ArgoCD admin password from Kubernetes secrets.
        
    * Logged in to the ArgoCD web UI using the external load balancer DNS.
        
3. **Creating ArgoCD Application**
    
    * Linked the GitHub repo (`Microservices-E-Commerce-eks-project`) in ArgoCD.
        
    * Deployed all Kubernetes manifests located in the `kubernetes-files` directory.
        
    * Enabled **automatic synchronization** to ensure that any change pushed to GitHub immediately reflected in the EKS cluster.
        

ArgoCD made deployment fully **declarative and version-controlled**, improving transparency and reducing manual effort.

---

## 🌐 Step 4: Domain Setup & HTTPS with Route 53 + ACM

To make the frontend accessible via a secure, custom domain, I configured **Route 53 and AWS Certificate Manager**.

1. **Domain Configuration**
    
    * Registered and linked the domain in **AWS Route 53**.
        
    * Created a **Public Hosted Zone** and added the Route 53 nameservers in Hostinger.
        
2. **SSL Certificate**
    
    * Requested a public SSL certificate in **AWS ACM**.
        
    * Chose **DNS validation** and automatically created the CNAME records in Route 53.
        
3. **Load Balancer HTTPS Setup**
    
    * Configured the **Classic Load Balancer (CLB)** with HTTPS listener on port 443.
        
    * Attached the ACM certificate to the load balancer.
        
    * Updated security groups to allow HTTPS traffic.
        
4. **DNS Record Mapping**
    
    * Created an A record in Route 53 pointing the domain to the frontend LoadBalancer DNS.
        

✅ Finally, visiting the domain web page loaded the frontend securely with HTTPS.

---

## 📊 Step 5: Monitoring & Observability

To monitor application performance and infrastructure health, I integrated **Prometheus and Grafana** with the EKS cluster.

* **Prometheus** scraped Kubernetes metrics, service statuses, and pod performance data.
    
* **Grafana** provided a visual dashboard to observe CPU, memory, and network metrics across all microservices.
    
* Enabled alerting rules for pod restarts, node failures, and latency spikes.
    

This observability setup helped ensure **high availability, reliability, and proactive issue detection**.

---

## ✅ Final Outcomes

By the end of this project, I had achieved a **fully automated, production-ready microservices deployment** pipeline.

**Key Achievements:**

* 🚀 Automated CI/CD pipeline with Jenkins and ArgoCD
    
* 🧱 Infrastructure as Code using Terraform modules
    
* 🐳 Secure container builds and image scans using Trivy
    
* 🌐 HTTPS-enabled domain setup via Route 53 + ACM
    
* 📈 Real-time monitoring using Prometheus & Grafana
    
* 🔄 Scalable EKS-based deployment of 11 microservices
    

---

## 💡 Learning Experience

This project helped me gain **hands-on experience with cloud-native architecture, DevOps automation, and Kubernetes orchestration**.  
It strengthened my understanding of how to:

* Build scalable infrastructure using Terraform
    
* Automate delivery pipelines with Jenkins and ArgoCD
    
* Secure web applications with HTTPS and domain integration
    
* Monitor workloads effectively in a microservices ecosystem
    

Through this implementation, I simulated a real-world DevOps workflow where infrastructure, code, and operations come together to deliver a seamless application lifecycle.

---

### 🔗 GitHub Repository

[saumyasingh9/Deploy-Production-ready-11-Microservices-E-Commerce-Platform-on-AWS-EKS: End-to-end deployment of a production-grade Microservices-based E-Commerce Platform on Amazon Elastic Kubernetes Service (EKS) using Terraform, Jenkins, ArgoCD, Docker, Prometheus, and Grafana. It includes 11 microservices, automated CI/CD pipelines, GitOps with ArgoCD, infrastructure-as-code (IaC), and real-time monitoring — all deployed securely](https://github.com/saumyasingh9/Deploy-Production-ready-11-Microservices-E-Commerce-Platform-on-AWS-EKS)

---