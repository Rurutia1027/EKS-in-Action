# EKS-in-Action

**EKS-in-Action** is a hands-on learning repository for mastering **Amazon Elastic Kubernetes Service (EKS)** — AWS’s managed Kubernetes offering.  
This repo contains examples, manifests, Terraform templates, and practical labs to help you deploy, manage, and scale Kubernetes workloads on AWS with confidence.

---

## 📌 Overview
Amazon EKS simplifies Kubernetes management by handling control plane operations while giving you the flexibility to manage worker nodes and workloads.  
With **EKS-in-Action**, you'll learn:
- How EKS fits into AWS's cloud-native ecosystem
- Best practices for networking, security, and scaling
- How to integrate AWS services for storage, load balancing, and observability

---

## 📚 Learning Modules

### 1. **EKS Fundamentals**
- What is EKS and common use cases
- EKS architecture & deployment options
- Required CLI tools & AWS setup

### 2. **EKS Networking**
- VPC CNI and Pod networking
- Prefix delegation & IPv6 support
- Network policies and isolation strategies

### 3. **EKS Storage**
- Using **EBS** for block storage
- Using **EFS** for shared file storage
- Other AWS storage options for Kubernetes

### 4. **EKS Secrets**
- AWS Secrets Manager & Parameter Store integration
- Kubernetes native secrets & encryption

### 5. **Load Balancing**
- AWS Load Balancer Controller
- Ingress and Gateway APIs
- VPC Lattice

### 6. **Compute & Scaling**
- Managed node groups
- AWS Fargate for serverless pods
- Karpenter for dynamic scaling

### 7. **Redundancy & Resiliency**
- IAM Roles for Service Accounts (IRSA)
- Pod identity & fine-grained access
- Multi-AZ high availability

### 8. **Upgrades & Maintenance**
- EKS monitoring and observability
- Upgrade cycles & strategies
- EKS add-ons lifecycle

---

## 🛠 Prerequisites
- AWS account with IAM permissions for EKS
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [eksctl](https://eksctl.io/)
- Terraform (optional, for IaC examples)

---

## 🎯 Who This Repo Is For
- **DevOps Engineers** looking to automate Kubernetes on AWS
- **Cloud Architects** designing secure and scalable workloads
- **Developers** deploying applications in cloud-native environments
- **SREs** managing production-grade Kubernetes clusters

---

## 📖 References
- [Amazon EKS Documentation](https://docs.aws.amazon.com/eks/)
- [eksctl CLI](https://eksctl.io/)
- [AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/)
- [Karpenter](https://karpenter.sh/)
- [AWS EKS Learning Path - KodeKloud](https://learn.kodekloud.com/user/courses/aws-eks)

---
