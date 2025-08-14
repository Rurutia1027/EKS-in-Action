# How CNI Works

## VPC Content 

### Get EKS VPC Info 
```bash 
$ eksdemo get vpc
+-----------------------+--------------------------------------+--------------------------------------+-----------------+-------------+
| Id                    | Name                                 | IPv4 CIDR(s)                         | IPv6 CIDR(s)    |
+-----------------------+--------------------------------------+--------------------------------------+-----------------+-------------+
| vpc-068d84bd223c3afd6 | eksctl-kodekloud-cluster/VPC          | 192.168.0.0/16                        | -               |
| vpc-7f711214          | *                                    | 172.31.0.0/16                         | -               |
+-----------------------+--------------------------------------+--------------------------------------+-----------------+-------------+
* Indicates default VPC
```
### `eksdemo get vpc ` Output Explanation 
- **Id**: 
The unique AWS VPC (Virtual Private Cloud) identifier. Each VPC has its own networking configuration. 
- **Name**: 
The human-readable name for the VPC.
-- `eksctl-kodekloud-cluster/VPC` -> VPC created for the EKS cluster. 
-- `*` -> Marks the **default VPC** in the AWS region. 

- **IPv4 (CIDRs)**
The IP address range (in CIDR notation) allocated for the VPC.
Example: `192.168.0.0/16` means the VPC has IP addresses from `192.168.0.0` to `192.168.255.255`. 

- **IPv6 (CIDRs)**
IPv6 address range (if assigned). Here, it's -- because no IPv6 range is set. 

- **Indicates default VPC**
Default AWS-provided VPC in each region; automatically created. 


## Subnet Content 

```bash 
$ eksdemo get subnets
+-----------------------+-----------+-------------------+-------+-------------+
| Id                    | Zone      | IPv4 CIDR         | Free  | IPv6 CIDR   |
+-----------------------+-----------+-------------------+-------+-------------+
| subnet-0ea8d9460384a28ec | us-west-2d | 192.168.0.0/19   | 8186  | -           |
| subnet-7e711215          | us-west-2c | 172.31.0.0/20    | 4091  | -           |
| subnet-0985f0fac4aa78207 | us-west-2d | 192.168.96.0/19  | 8186  | -           |
| subnet-2fd03704          | us-west-2d | 172.31.48.0/20   | 4091  | -           |
| subnet-7d711216          | us-west-2b | 172.31.32.0/20   | 4091  | -           |
| subnet-01c03526c3f1b6af6 | us-west-2a | 192.168.128.0/19 | 8162  | -           |
| subnet-0a663a22dfed643a3 | us-west-2a | 192.168.160.0/19 | 8175  | -           |
| subnet-01945f721bd2414f8 | us-west-2a | 192.168.32.0/19  | 8187  | -           |
| subnet-087eb1ba567a91170 | us-west-2a | 192.168.64.0/19  | 8187  | -           |
| subnet-7c711217          | us-west-2a | 172.31.16.0/20   | 4091  | -           |
+-----------------------+-----------+-------------------+-------+-------------+
```
### `eksdemo get subnets ` Output Explanation 

- **Id**
Unique AWS Subnet ID within the VPC. Each subnet is tied to a specific available zone. 

- **Zone**
AWS Availability Zone (AZ) where the subnet resides. Example: `us-west-2a`.

- **IPv4 CIDR**
The IP address range for that subnet, a smaller portion of the VPC CIDR block. 

- **Free**
Number of available IP addresses left in the subnet for resource allocation (pod, nodes, services, etc.).

- **IPv6 CIDR**
IPv6 range for the subnet; here all are -- (no IPv6 configured). 
 










----

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