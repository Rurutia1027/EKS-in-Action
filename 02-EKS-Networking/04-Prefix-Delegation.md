# AWS EKS: Prefix Delegation 

## Context Overview 
- **Prefix Delegation** allows ENIs attached to EC2 nodes to pre-allocate blocks of IP addresses for pods before they are actually needed. 
- **Warm ENIs** are ENIs with pre-attached IPs, ready for pods, reducing scheduling latency. 
- **VPC Subnets** provide the IP address ranges thatn ENIs and prefix delegation blocks draw from. 


## How IPs are Allocated 
- The ENI requests a prefix delegation block from the VPC subnet. 
- The VPC subnet has a route table that determines the available IP ranges. 
- The IPs are allocated in **continuous blocks**, often sized as /28 slices (16 IPs each).
- Each prefix block is then pre-occupied for future pod use, even if pods are not yet scheduled.

## Relationships: VPC -> Route Table -> ENI -> Pods

- VPC Subnet: Source of all IP address
- Route Table: Determines IP routing and allocation slices
- ENI: Attaches to EC2 node; can have prefix-delegated IP blocks. 
- Prefix Delegation: Pre-allocates IP blocks for pods, reducing attach latency. 
- Pod: Receives an IP from the delegated block when scheduled. 

## ENI Configuration for Prefix Delegation 
- Parameter: `ENABLE_PREFIX_DELEGATION` (Boolean toggle)
- Enable/true; ENI can receive pre-allocated prefix blocks from the VPC sunet. 
- Disabled/false: ENI uses only standard single IP allocations from the subnet. 

### AWS CLI Example
```bash 
aws ec2 create-network-interface \
  --subnet-id subnet-xxxxxxxx \
  --description "ENI with prefix delegation" \
  --interface-type interface \
  --enable-prefix-delegation
```

### AWS Terraform 
```hcl 
provider "aws" {
  region = "us-east-1"
}

resource "aws_network_interface" "eni_with_prefix_delegation" {
  subnet_id       = "subnet-xxxxxxxx"
  description     = "ENI with prefix delegation"
  interface_type  = "interface"
  enable_prefix_delegation = true

  # Optional: assign security groups
  security_groups = ["sg-xxxxxxxx"]

  tags = {
    Name = "prefix-delegated-eni"
  }
}

# Optionally attach to an EC2 instance
resource "aws_instance" "example" {
  ami           = "ami-xxxxxxxx"
  instance_type = "t3.medium"

  network_interface {
    network_interface_id = aws_network_interface.eni_with_prefix_delegation.id
    device_index         = 0
  }

  tags = {
    Name = "example-instance"
  }
}
```

