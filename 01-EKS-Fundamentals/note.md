# EKS Introduction 


## What is EKS 

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service by AWS.
It runs Kubernetes control plane for you, so you don't have to install, operate, or maintain it yourself. You just focus on your apps -- AWS takes care of keeping Kubernetes available and secure. 

## When and Why to Use EKS 
Use EKS when you want to run Kubernetes workloads in AWS without managing all the underlying control plane infrastructure. 
It's greate when you need: 
- Scalability: Run apps that can grow or shrink automatically
- Security: Integrate with AWS IAM, VPC, and other security tools. 
- Reliability: Highly available clusters across multiple AWS zones. 
- Integration: Easy connection to AWS services like S3, RDS, DynamoDB, and more. 


## Kubernetes Cluster in AWS With EC2 

In EKS, you can run your Kubernetes workloads on Amazon EC2 instances. 
EKS manages the control plane, and you provide the worker nodes by launching EC2 instances in your AWS account. 

This setup gives you:
- Full control over nodes -- choose instance types, sizes, and OS.
- Flexibility -- install custom software or configurations.
- Cost control -- use On-Demand, Reserved, or Spot instances. 


## EKS Architecture 

When we create an EKS cluster, AWS sets up several key components for us, but only for the control plane -- we still decide how to run worker nodes. 

Here is what we get in base architecture:

### Managed Control Plane (AWS handles this)
- Kubernetes API Server - the entry point for all cluster commands (kubectl, deployments, scaling, etc).
- etcd - Kubernetes' key-value database that stores cluster state and configuration
- Controller Manager & Scheduler - core Kubernetes processes for managing workloads and scheduling pods
- Cluster Endpoint - a secure, public or private endpoint to connect to your cluster 


These components are multi-AZ (spread across multiple Availability Zones) for high availability and are managed, patched, and monitored by AWS. 

### Networking Services 
- VPC integration - EKS plugs into your existing or new VPC for networking 
- Security Groups - control inbound/outbound traffic for control plane and nodes
- IAM Integration - AWS identity and Access Management for controlling who can do that in the cluster
- Cluster DNS - provided via CoreDNS for service discovery inside the cluster 

### Worker Node Options (optional)
- Amazon EC2 - you create node groups to run workloads 
- AWS Fargate - serverless compute for running pods without managing servers
- You can mix EC2 and Fargate in the same cluster 


### Tools Needed for EKS 

#### eksctl : kubectl like CLI for Amazon EKS 
The easy button for learning, testing, and demoing Amazon EKS:
- Install complex applications and dependencies with a single command.
- Extensive application catalog (over 50 CNCF, open source and related projects)
- Customize application installs easily with simple command line flags. 
- Query and search AWS resources with over 60 kubectl-lik get commands. 

#### AWS IAM Authenticator for Kubernetes 
A tool to use AWS IAM credentials to authenticate to a Kubernetes clsuter. The initial work on this tool was driven by Heptio. The project receives contributons from multiple community engineers and is currently maintained by Heptio and Amazon EKS OSS Engineers. 


### Summary 
Creating an EKS cluster gives you a fully managed, highly available Kubernetes control plane connected to your AWS networking and security. You then decide whether to attach EC2 nodes, use Fargate, or both to run workloads. 