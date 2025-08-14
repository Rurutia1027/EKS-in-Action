# EKS DaemonSet Overview 

## EKS DaemonSet Detail Info 

```bash 
# command describe the aws-node DaemonSet in kube-system namespace
# shows which pods are running on which nodes and their status, environments, volumes, etc. 

$ kubectl describe ds -n kube-system aws-node

Name: aws-node
Selector: k8s-app=aws-node
Node-Selector: <none>
labels: 
  app.kubernates.io/instance=aws-vpc-cni # identifiers the CNI DaemonSet instance 
  app.kubernetes.io/managed-by=Hel       # indicates it was deployed by Helm 
  app.kubernetes.io/name=aws-node
  app.kubernetes.io/version=v1.16.2
Annotations:
  helm.sh/chart=aws-vpc-cni-1.16.2
  k8s-app=aws-node 
  eksctl.io/restartedAt: 2024-02-04T21:52:14-08:00
Desired Nodes Scheduled: 2
Current Nodes Scheduled: 2
Nodes with Up-to-date Pods: 2
Nodes with Available Pods: 2
Nodes Misscheduled: 0
Pods Status: 2 Running / 0 Waiting / 0 Succeeded / 0 Failed 

# -> The above shows that all nodes expected to run aws-node pods are actually running them 

Pod Template: 
  Labels: 
    app.kubernetes.io/instance=aws-vpc-cni
    app.kubernetes.io/name=aws-node
    k8s-app=aws-node
  Service Account: aws-node # IAM permission for accessing ENIs and EC2 metadata 

Init Containers:
  aws-vpc-cni-init:
    Image: 602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon-k8s-cni-init:v1.16.2-eksbuild.1
    # Prepares CNI binaries on the host before the main container runs 
    Requests:
      cpu: 25m
    Environment:
      DISABLE_TCP_EARLY_DEMUX: false 
      ENABLE_IPV6: false 
    Mounts:
      /host/opt/cni/bin from cni-bin-dir (rw) # puts CNI binaries on the host for kubelet to use 

Containers:
  aws-node: 
    Image: 602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon-k8s-cni:v1.16.2-eksbuild.1
    # Main CNI plugin container that assigns IPs, sets up routes, NodePort, ENIs
    Port: 61678/TCP
    Host Port: 0/TCP
    Requests:
      cpu: 25m
    Liveness & Readiness Probes:
      # Ensures container is healthy and responsive 
      exec [/app/grpc-health-probe -addr=:50051 -connect-timeout=5s -rpc-timeout=10s period=10s]
    Environment:
      ADDITIONAL_ENI_TAGS: {}           # Extra tags for ENIs
      ANNOTATE_POD_IP: false            # Whether to annotate Pod IPs in metadata 
      AWS_VPC_CNI_NODE_PORT_SUPPORT: true       # Enables NodePort networking
      AWS_VPC_ENI_MTU: 9001 
      ENABLE_IPV4: true 
      ENABLE_POD_ENI: false             # Disable per-Pod ENIs
      VPC_ID: vpc-068d84bd223c3afd6
      WARM_ENI_TARGET: 1                # Pre-allocate one ENI per node for performance 
      MY_NODE_NAME: (v1: spec.nodeName)
      MY_POD_NAME: (v1: metadata.name)

    Mountes:
      /host/etc/cni/net.d from cni-net-dir (rw)
      /host/opt/cni/bin from cni-bin-dir (rw)
      /host/var/log/aws-routed-eni from log-dir (rw)
      /run/xtables.lock from xtables-lock (rw)
      /var/run/aws-node from run-dir (rw) 

  aws-eks-nodeagent:
    Image:  602401143452.dkr.ecr.us-west-2.amazonaws.com/amazon/aws-network-policy-agent:v1.0.7-eksbuild.1
    # Optional network policy enforcement for EKS 
    Args:
      --enable-ipv6=false 
      --enable-network-policy=true 
      --enable-cloudwatch-logs=false 
      -metrics-ind-addr=:8162
      --health-probe-bind-addr=:8163
    Requests:
      cpu: 25m
    Mounts:
      /sys/fs/bpf from bpf-pin-path (rw)
      /var/log/aws-routed-eni from log-dir (rw)
      /var/run/aws-node from run-dir (rw)

Volumes:
  bpf-pin-path:
    Path: /sys/fs/bpf
    # Used by eBPF programs for networking
  cni-bin-dir:
    Path: /opt/cni/bin
  cni-net-dir:
    Path: /etc/cni/net.d
  log-dir:
    Path: /var/log/aws-routed-eni
  run-dir:
    Path: /var/run/aws-node
  xtables-lock:
    Path: /run/xtables.lock
    # Prevents multiple processes from modifying iptables at the same time
```

## Concepts & Explanation
### DaemonSet (aws-node)
- Ensures one copy of the aws-node pod runs on each worker node. 
- Provides the VPC CNI networking functionality for pods. 

### Node Scheduling & Status
- Desired Nodes Scheduled: 2 -> two worker nodes in the cluster
- Current Nodes Scheduled: 2 -> each node ahs the aws-node pod running
- Pods Status: 2 Running -> all networking pods are healthy.

### Pod Template 
Container labels, annotations, service account, and pod spec. 
##### Init Container (**aws-vpc-cni-init**) prepares the node for pod networking. 
- Sets up IP rules and routes
- Mounts `/host/opt/cni/bin` for CNI binaries
- Minimal CPU requests (25m)

##### Main container (**aws-node**) handles: 
- Allocating secondary IPs from ENIs to pods. 
- Configuring VPC-native routing. 


##### Secondary container (**aws-eks-nodeagent**):
- Handles **network policies**, metrics, and connection tracking cleanup.
- Supports optional IPv6, CloudWatch logging, and event logging. 


### ENI & IP Mapping 
- Each worker node (EC2 instance) has one or more ENIs. 
- Pods get **secondary IPs from these ENIs**, which come from the subnet's IP pool.
- ENIs act as a **proxy for subnet IPs**: multiple pods share the ENI, but each pod gets a unique IP.

### Health Checks 
- `aws-node`: uses **gRPC probes** for liveness/readiness.
- `aws-eks-nodeagent`: has **HTTP probes** for metrics and health. 

### Networking Flow 
```
Worker Node (EC2 instance)
   │
   └─> aws-node pod (DaemonSet)
          │
          ├─> Init container sets up networking
          │
          └─> Main container allocates pod IPs via ENI
                   │
                   └─> Pod traffic flows directly into VPC
```

### Cluster Integration 
- The `aws-node` pods are critical **EKS pod networking**
- They ensure pods can communicate using **VPC-native IPs**
- Scaling nodes or pods triggers **ENI allocation automatically**


--- 

## `aws-vpc-cni` Container Logs
- EKS `aws-vpc-cni-init` container logs 

```bash 
# View logs for the aws-vpc-cni-init container 
kubectl logs -n kube-system aws-node-z7llk -c aws-vpc-cni-init 


time="2024-02-05T05:55:00Z" level=info msg="Copying CNI plugin binaries ..."

# Copies the AWS VPC CNI plugin binaries into the host's CNI directory
# These binaries are executed by kubelet to assign Pod IPs and configure network routes. 


time="2024-02-05T05:55:00Z" level=info msg="Copied all CNI plugin binaries to /host/opt/cni/bin"
# Successfully placed CNI binaries into /opt/cni/bin on the host.
# The host's kubelet will call these plugins when creating Pods

time="2024-02-05T05:55:00Z" level=info msg="Found primaryMAC 06:cb:7e:3b:e4:d3"
# Deleted the MAC address of the Node's primary network interface (Primary ENI).
# This is the main network card assigned to the EC2 instance.

time="2024-02-05T05:55:00Z" level=info msg="Found primaryIF eth0"
# Confirmed that the primary ENI is mapped to the eth0 interface
# By default, AWS maps the primary ENI to eth0

time="2024-02-05T05:55:00Z" level=info msg="Updated net/ipv4/conf/eth0/rp_filter to 2"
# Adjusted Reverse Path Filtering (rp_filter) to value 2 (loose mode).
# Loose mode allows asymmetric routing so cross-sunet and VPC Pod traffic won't be dropped. 

time="2024-02-05T05:55:00Z" level=info msg="Updated net/ipv4/tcp_early_demux to 1"
# Enabled TCP Early Demux, which speeds up TCP packet handling by routing packets faster to the correct sockets.

time="2024-02-05T05:55:00Z" level=info msg="CNI init container done"
# The init container finished its setup tasks.
# The main aws-node container now takes over IP address management and ongoing network configuration.
```

