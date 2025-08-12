# How CNI Works

## What's CNI ? 

CNI stands for Container Network Interface. It's a specification and a set of libraries for configuring network interfaces in Linux containers. In Kubernetes, CNI plugins are used to provide networking for pods -- assigning IPs, routing traffic, and enabling communicaiton. 

## How CNI Works in EKS (Amazon Elastic Kubernetes Service)

- Amazon EKS uses Amazon VPC CNI Plugin as its default networking plugin. 
- This plugin allows Kubernetes pods to have IP addresses from the VPC subsets (the same IP range your AWS EC2 instances use).
- Pods are treated as first-class network entities in your VPC, with their own routable IP addresses. 

## Key Points About AWS VPC CNI Plugin:
### Pod IP assignment 
- When a pod is created, the VPC CNI plugin allocates an Elastic Network Interface (ENI) from the VPC. 
- The ENI is attached to the EC2 instance node. 
- Each pod gets an IP address from the VPC subnet. 

### Networking 
- Pods can communicate directly with other pods, services, or AWS resoruces within the VPC using standard VPC routing. 
- This allows seamless integration between Kubernentes workloads and AWS services. 

### Scalability 
- Each EC2 instance has a limit on how many ENIs and IPs it can have. 
- The CNI plugin manages pod IP assignment based on those limits. 
- If an instance runs out of free IPs, new pods can't be scheduled on that node. 

### Performance & Security 
- Using VPC native networking means low latency and high throughput.
- You can apply VPC security groups and network ACLs to pods indirectly via their ENIs


## Why VPC & CNI Important ? 
Pods have native AWS VPC IPs, which means they can: 
- Use AWS security features easily.
- Avoid NAT or overlay networks (like Flannel or Calico's overlays)
- Have consistent IP reachability within the VPC


## Summary 
- CNI in EKS is an AWS VPC-aware networking plugin. 
- It assigns real VPC IPs to pods by attaching ENIs to nodes. 
- This provides high-performance, secure, and seamless pod networking integrated wiht AWS infrastructure. 