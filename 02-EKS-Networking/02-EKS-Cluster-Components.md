# EKS Cluster Components 

## Control Plane (Fully Managed by AWS)
### Fully Managed and Abstracted 
- You cannot see or manage EC2 instances for the control plane.
- AWS automatically handles high availability, scaling, patching, and security.
- You interact with it only via **Kubernetes API**, **kubectl**, or other supported management tools. 
- Users can configure cluster-level behavior safely (e.g., scheduling policies, RBAC, namespaces,quotas) without direct OS-level access. 

### Differences from EC2 Instances 
- Runs the Kubernetes API server, scheduler, controller mmanager. 
- Fully managed and highly available across multiple AZs. 
- Control Plane != EC2 instance - you cannot stop, start, restart, or teminate it. 
- No direct ENIs or IPs to assign; networking is abstracted. 
- Ensures the cluster functions and orchestrates pods on worker nodes. 
- **Billing**: charged per cluster, independent of worker nodes. 

## Worker Nodes (EC2 Instances You Applied Constructed of)

### One Node = One EC2 Instance 
- Each worker node corresponds to **exactly one EC2 instance**
- Responsible for running pods.
- Multiple worker nodes can exist in the same cluster. 
- **Billing**: standard EC2 pricing applies (instance type, storage, networking).

### ENIs and Subnets 
Node -> ENIs = 1 : N : each (Work Node | EC2 Instance) node can attache one or more ENIs.
- Primary ENI -> main IP used by the node itself. 
- Secondary IPs on ENIs -> assigned to pods. 
- If one ENI's IP pool is insufficient, ** additional ENIs can be attached**. 

### CNI (AWS VPC CNI)
- CNI plugin manages ENIs and IP allocation for pods. 
- Pods get secondary IPs from ENIs for VPC-native networking. 
- Enables pods to communicate directly in the VPC without NAT. 
- Scaling IPs: additional ENIs allow nodes to support more pods than a single ENI's CIDR permits. 


## Networking Summary 
- **Control Plane**: abstracted, managed, no direct ENIs or pod IPs.
- **Worker Nodes**: real EC2 instances, attach multiple ENIs, primary + secondary IPs. 
- **Pods**: get IPs from secondary IPs on ENIs, allowing direct VPC connectivity. 
- **ENIs**: fundamental unit for networking; each ENI belongs to **one subnet**, IPs allocated from that subnet (CIDR). 
