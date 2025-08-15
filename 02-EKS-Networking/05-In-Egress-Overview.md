# EKS & K8S Ingress and Egress Overview 

## Basic Definitions

Ingress & Egress are all traffic with directions. 
First, i have to say: both Ingress & Egress are traffic, ingress -- traffic flow coming from client/node&pod external world -> Node/Pod internal combined with several filter rules via ENI, Security Groups and even VPCs. 

### Ingress 
- **General Kubernetes context**: Ingress represents the **incoming traffic** to your cluster from outside. It usually defines **HTTP/S routing rules,** letting external clients reach services inside the cluster. 
- **Key Point**: Ingress is **layer 7(application layer)** traffic management. It handles hostnames, paths, TLS termination, etc. 
- **Typeical Kubernetes objects involved:**
> Ingress resource (defines rules)
> IngressController (the actual controller that implements the rules, e.g., NGINX, AWS, ALB Ingress Controller).

### Egress 
- **General Kubernetes context**: Egress is **outgoing traffic** leaving the cluster, typically from pods to external endpoints (like APIs, databases, or the internet).
- **Key point**: Egress is mostly **layer 3/4 (IP and port)** traffic. It can be controlled via **NetworkPolicies**.

## Ports and Traffic Flow 
### Ingress Traffic: 
Comes from outside -> **Node/Pod ports**. Examples:
- External client hits `https://my-app.example.com` 
- Traffic reaches the `IngressController Service (LoadBalancer)`.
- Routed to pod port 80 or 443. 

### Egress traffic 
Goes from **Pod -> external IP/Port**. Examples: 
- Pod wants to call `https://api.example.com:443`.
- Traffic flows out of the pod's IP (usually the **ENI secondary IP** in AWS) through the node's networking stack. 

## Kubernetes Native vs AWS EKS Differences 
### Ingress 
- **Native K8S**: Managed by your own controllers (NGINX, Traefik).
- **AWS EKS**: Can use AWS-managed ALB Ingress Controller for simplified scaling and integration with AWS services. 

### Egress 
- **Native K8S**: Pods typically use node NAT or custom routing.
- **AWS EKS**: EKS uses **VPC CNI plugi**, where pods get VPC-native IPs. Outgoing traffic can be controlled via **Security Groups, NACLs, or NAT Gateways**.

### Networking
- **Native K8S**: Can be overly network (Calico, Flannel, Weave).
- **AWS EKS**:Direct VPC-native networking with ENIs; each pod gets a **secondary IP**. Egress is more predictable in IP assignment. 

### Security Control 
- **Native K8S**: NetworkPolicies only. 
- **AWS EKS**: NetworkPolicies + AWS Security Groups + IAM roles for services accounts (IRSA) can manage permissions along with traffic control. 

**Key Differences**
- In **native K8S**, ingress/egress is mostly **logical** within the cluster network. 
- In **AWS EKS**, AWS networking (VPC, subnets, ENIs, security groups) directly influences ingress/egress. Pods have real IPs in VPC, so AWS-native firewall rules apply. 

## Port Mapping and Rules 
- **Ingress ports**: Usually 80/443 for HTTP/S, but can be any port exposed via Ingress rules.
- **Egress ports**: Can be any port your pod connects to, controlled by **Network Policies** or AWS security groups. 
- **Offset concept**: Sometimes used in Kubernetes CNI to handle **port conflicts** or **port ranges**, especially for high-density clusters. 

## Visulization of Flow 
```
[External Client] --> [AWS Load Balancer / NodePort / Ingress] --> [Pod Port 80/443]  (Ingress)
[Pod Port 8080] --> [ENI Secondary IP / VPC NAT / External API]  (Egress)
```

--- 

# VPC & Subnet & ENI & {Ingress, Egress}
## VPC and Subnet Role 
- **VPC**: Acts as the top-level network boundary. All your EKS worker node s(EC2 instances) and ENIs lives inside a VPC. It defines; 
> IP address ranges (via CIDR block)
> Routing 
> Security boundaries (via Security Groups and NACLs)

- **Subnet**: Divides the VPC into smaller ranges, usually tied to **Availability Zones**
> Worker nodes are launched into subnets. 
> ENIs for pods (via AWS VPC CNI) are assigned **secondary IPs** from the subnet CIDR. 

- **Relationship with EKS pods**
> Each pod that gets a VPC IP uses a secondary IP on an ENI.
> That means the pod's network traffic is **native to the VPC**, making ingress/egress fully trackable with standard AWS networking. 


## ENIs (Elastic 2s)
- **Worker node ENI**: Each EC2 instance has a primary ENI with a main IP.
- **Pod ENIs**: Using AWS VPC CNI, pods get **seconary IPs on t the node ENI**.
- Implication for traffic: 
> **Ingress**: External traffic enters via the VPC (Load Balancer -> subnet -> ENI -> pod).
> **Egress**: Pod traffic leaves through its assigned ENI and subnet route table could (could go direclty out to the internet if public subnet or via NAT gateway if private subnet).


## Ingress Traffic Flow (EKS)
```
[Internet] 
   ↓ (through ALB/NLB)
[VPC Subnet] 
   ↓ (security group allows traffic)
[Node ENI Primary IP]
   ↓ (secondary IP assigned to pod)
[Pod Port 80/443]
```
- **Security Groups**: Control what ingress ports are allowed. 
- **Route Table**: Ensures the subnet knows where to send incoming packets. 
- **Load Balancers**: Usually manage host/path routing to pods. 


## Egress Traffic Flow (EKS)
```
[Pod Port 8080] 
   ↓ (secondary IP on node ENI)
[Node ENI Primary IP / Subnet Route]
   ↓ (VPC NAT Gateway or IGW if public)
[Internet / External Service]
```
- Pods leave the cluster using their assigend secondary IP.
- Outbound traffic is controlled via: 
> **Security Groups (allow outbound)**
> **NACLs** at subnet level
> **Route tables** for routing to internet/NAT


## Key Relationships 
- **Ingress**: Incoming traffic to pods; associated with AWS Resources {VPC subnets, Security Groups, Load Balancers, ENIs}.
- **Egress**: Outgoing traffic from pods; associated with AWS Resources {ENIs, Subnet route tables, NAT/IGW, Security Groups}.
- **Pod IP (secondary IP from ENIs)**: Assigned from subnet via ENI; {ENI secondary IP on Node ENI}. 
- **Node IP (primary IP from ENIs)**: Receives the traffic for pods; {ENI primary IP}. 


## Summary 
- **Pods** in EKS are **first-class citizens in the VPC** because they use ENI seconary IPs.
- **Ingress** comes in through VPC -> subnet `<where attached to one or more security groups and security rules locates/works>` -> ENI -> pod. 
- **Egress** leaves from pod -> ENI -> subnet -> VPC gateway -> internet. 
- Security, routing, and IP management are all handled at the **AWS VPC level**, not just Kubernetes. 


```mermaid
flowchart LR
    Client[Client / External Traffic] -->|Ingress| VPC[VPC]
    VPC --> Subnet[Subnet]
    Subnet --> ENI1[ENI 1]
    Subnet --> ENI2[ENI 2]

    ENI1 --> SG1[SG Rules for ENI1]
    ENI2 --> SG2[SG Rules for ENI2]

    SG1 --> Node1[Node (EC2)]
    SG2 --> Node2[Node (EC2)]

    Node1 --> Pod1[Pod]
    Node2 --> Pod2[Pod]

    Pod1 -->|Egress| Node1
    Pod2 -->|Egress| Node2

    Node1 --> ENI1
    Node2 --> ENI2
```


--- 

# Ingress & Egress with Security Groups / Security Rules in AWS 
## Security Groups -> ENIs
- Security groups are **attached to ENIs**, not directly to subnets or VPCs.
- An ENI can have **multiple security groups** (many-to-one), which means one ENI can be attached to multiple security rules defined from different security groups -- but all those security groups should belong to the current VPC cross VPC security groups attaching is denied(security groups only work in one VPC across VPC is not allowed). 
- The **rules from all attached security groups** are applied to traffic going in/out that ENI (in traffic attach to pod internal is ingress, egress is out going traffic through similar security rules from security groups that attaching to the ENIs). 

## VPC & Security Groups 
- When we create a VPC, we defines **a set of security groups within that VPC**. 
- Only **security groups created in that VPC** can be attached to ENIs in that VPC. 
- So the VPC acts as a **namespace** for security groups. 

## Rules effectiveness
- Security group rules **only take effect whent he SG is actually attached to an ENI**.
- Even if you create multiple SGs in the VPC, they **don't enforce anything** until they are attached to the ENIs(ENI is subnet proxies, so SG -> attached to ENI -> SG rules work on all IPs in that IP Pool(subnet)). 

--- 

# Summary 
**Ingress** in the context of AWS/EKS/Kubernetes is about **incoming traffic to a resource{node, pod, or services}**.

- **Ingress** = **incoming traffic**
> At the **VPC/ENI** level, ingress traffic is **any traffic coming to the ENI**.
> Whether it's allowed or blocked depends on **security group rules** attached to the ENI.

- **Security groups** = **rules applied to ingress/egress**
> Security groups are **not ingress themselves**, but **they control how ingress and egress traffic is allowed**. 
> You can have mulitple security groups on an ENI; all their rules are combined. 

- **Ingress in Kubernetes**
> At the **pod/service level**, ingress can also be defined via **Kubernetes NetworkPolicies or ingress resources**.
> Kubernetes ingress controls traffic **at the applicaiton layer L7 (HTTP/S, routing, port, path)**, not just network connectivity. 


## In NutShell 
- ENI is attached to the Node. 
- An ENI has {primay IP(main IP of the Node), secondary IPs(used for Pods via AWS CNI)}.
- Security Groups (SGs) can be attached to the ENI. 
- The **rules of the SGs apply to all IPs(primay & secondary) on that ENI**, including both primary and secondary IPs. 
- Multiple ENIs can be attached to a single Node, each with its own set of SGs. 

```
Client -> VPC -> Subnet -> ENI -> Security Group rules -> Node -> Pod
```
