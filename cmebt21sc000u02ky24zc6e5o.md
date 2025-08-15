---
title: "Mastering AWS VPC NAT Gateways"
datePublished: Thu Aug 14 2025 19:41:35 GMT+0000 (Coordinated Universal Time)
cuid: cmebt21sc000u02ky24zc6e5o
slug: mastering-aws-vpc-nat-gateways
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755200426725/319d560c-3540-465a-bba1-7d26d2d78f6b.jpeg
tags: cloud, aws, cloud-computing, vpc, aws-certified-solutions-architect-associate, cloud-security, nat-gateway, vpc-basics

---

## Why you need a NAT Gateway (in one minute)

In a well‑designed VPC, application servers often live in **private subnets** with **no public IPs**. They still need to **reach the internet** for package updates, pulling container images, or talking to third‑party APIs. A **NAT Gateway** lets instances in private subnets **initiate** outbound connections to the public internet **without being directly reachable** from it.

> TL;DR: NAT Gateway = outbound‑only internet access for private subnets (IPv4). For IPv6, use an **egress‑only internet gateway** instead.

---

## NAT Gateway vs Alternatives

* **NAT Gateway (managed, zonal):** Highly available within one AZ, scales automatically, minimal ops. Billed per hour + per GB processed. Deploy **one per AZ** for HA and to avoid cross‑AZ data charges.
    
* **NAT Instance (self‑managed):** Cheaper in tiny labs, but you patch, scale, secure, and failover it yourself.
    
* **VPC Endpoints (Gateway/Interface):** Bypass the internet entirely for AWS services like S3/DynamoDB and often reduce NAT data costs.
    
* **Egress‑Only Internet Gateway:** For **IPv6** outbound‑only traffic.
    

---

## What you’ll build

A VPC with two AZs, each containing a public and a private subnet. Each public subnet hosts a **NAT Gateway**; each private subnet routes default traffic to the NAT Gateway **in the same AZ**.

### Architecture Diagram (Mermaid)

```mermaid
flowchart TB
  Internet((Internet))
  IGW[Internet Gateway]
  subgraph VPC
    direction TB
    subgraph AZ-a
      direction TB
      PubA[Public Subnet (AZ-a)]
      NATa[NAT Gateway (AZ-a)]
      PrivA[Private Subnet (AZ-a)]
    end
    subgraph AZ-b
      direction TB
      PubB[Public Subnet (AZ-b)]
      NATb[NAT Gateway (AZ-b)]
      PrivB[Private Subnet (AZ-b)]
    end
  end

  Internet --> IGW
  IGW --> PubA
  IGW --> PubB
  PubA --> NATa
  PubB --> NATb
  PrivA --> NATa
  PrivB --> NATb
```

### Quick ASCII Diagram (fallback)

```plaintext
Internet
   |
 [IGW]
  /   \
[PubA] [PubB]
  |       |
[NATa]  [NATb]
  |       |
[PrivA] [PrivB]  -> Instances have no public IPs; outbound goes via NAT in same AZ
```

---

## Prerequisites

* A VPC with at least **two AZs** (for HA).
    
* **Two public subnets** (auto‑assign public IPv4 on), **two private subnets** (no public IPs).
    
* An **Internet Gateway (IGW)** attached to the VPC.
    
* Public route table(s) with default route `0.0.0.0/0 → IGW`.
    

> Not strictly required but recommended: enable **DNS hostnames** and **DNS resolution** on the VPC for package mirrors and domain lookups.

---

## Step‑by‑Step: Create NAT Gateways (AWS Console)

1. **Allocate Elastic IPs (EIPs)**
    
    * VPC Console → **Elastic IP addresses** → **Allocate Elastic IP address**.
        
    * Allocate **one EIP per NAT Gateway** (typically one per AZ).
        
2. **Create NAT Gateways**
    
    * VPC Console → **NAT Gateways** → **Create NAT gateway**.
        
    * **Subnet:** choose the **public subnet** in AZ‑a; attach **EIP‑1**; **Connectivity type:** *Public*.
        
    * Repeat for AZ‑b (public subnet in AZ‑b with **EIP‑2**).
        
3. **Private Route Tables → Default Routes**
    
    * VPC Console → **Route tables** → create (or edit) a route table for **Private‑AZ‑a**.
        
        * Add route: `0.0.0.0/0 → NAT Gateway (AZ‑a)`.
            
    * Do the same for **Private‑AZ‑b** with its **AZ‑b NAT Gateway**.
        
    * (Optional IPv6) Add `::/0 → Egress‑only Internet Gateway` if you use IPv6.
        
4. **Associate Route Tables to Private Subnets**
    
    * Associate **Private‑AZ‑a** route table to **Private Subnet in AZ‑a**.
        
    * Associate **Private‑AZ‑b** route table to **Private Subnet in AZ‑b**.
        
5. **Security Groups & NACLs**
    
    * **Security Groups:** For instances in private subnets, ensure **egress** allows needed destinations (`0.0.0.0/0` is common; restrict if you can). Inbound is not required for outbound‑only.
        
    * **Network ACLs:** Keep default *stateless* allows, or ensure ephemeral return ports (1024‑65535) are permitted for outbound flows.
        
6. **Test Connectivity**
    
    * Launch an **EC2** in a private subnet **without public IP**.
        
    * Connect via **SSM Session Manager** (preferred) or via a **bastion** in the public subnet.
        
    * Run `curl` [`https://ifconfig.me`](https://ifconfig.me) (should show the **EIP** of the NAT in that AZ), or `sudo yum -y update` / `apt update` to confirm outbound.
        
7. **High Availability Notes**
    
    * A NAT Gateway is an **AZ‑scoped** resource. For HA and to avoid **cross‑AZ data charges**, place **one NAT per AZ** and ensure private subnets in that AZ route to the **local** NAT.
        
8. **Cost & Optimization Tips**
    
    * Charged **per hour** and **per GB processed**. Use **VPC Endpoints** (S3, DynamoDB, ECR, CloudWatch) to lower NAT traffic.
        
    * Turn off labs you’re not using. One NAT per AZ can be significant in non‑prod.
        

---

## Optional: Terraform snippet

Use this as a starting template; adapt IDs and CIDRs to your environment.

```plaintext
# Elastic IPs (one per NAT)
resource "aws_eip" "nat_a" { domain = "vpc" }
resource "aws_eip" "nat_b" { domain = "vpc" }

# NAT Gateways in public subnets
resource "aws_nat_gateway" "a" {
  allocation_id = aws_eip.nat_a.id
  subnet_id     = aws_subnet.public_a.id
  tags = { Name = "nat-a" }
}
resource "aws_nat_gateway" "b" {
  allocation_id = aws_eip.nat_b.id
  subnet_id     = aws_subnet.public_b.id
  tags = { Name = "nat-b" }
}

# Private route tables
resource "aws_route_table" "priv_a" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.a.id
  }
  tags = { Name = "rt-private-a" }
}
resource "aws_route_table" "priv_b" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.b.id
  }
  tags = { Name = "rt-private-b" }
}

# Associate route tables with private subnets
resource "aws_route_table_association" "priv_a" {
  subnet_id      = aws_subnet.private_a.id
  route_table_id = aws_route_table.priv_a.id
}
resource "aws_route_table_association" "priv_b" {
  subnet_id      = aws_subnet.private_b.id
  route_table_id = aws_route_table.priv_b.id
}
```

---

## Troubleshooting checklist

* **No internet from private instances?**
    
    * Does the private route table have `0.0.0.0/0 → NAT`?
        
    * Is the private subnet **associated** with that route table?
        
    * Is the NAT in a **public subnet** with a route to the IGW and an **Elastic IP** attached?
        
    * Security group **egress** allows outbound? NACLs allow ephemeral return ports?
        
    * Using IPv6? Remember NAT Gateway is IPv4‑only → use **egress‑only IGW**.
        
* **High data charges?** Add **VPC Endpoints** for S3, DynamoDB, ECR/CloudWatch, or consider keeping NAT per AZ to avoid cross‑AZ charges.
    

---

## Clean‑up (avoid charges)

1. Terminate test EC2 instances.
    
2. Delete **route entries** pointing to NAT, then delete **route tables** if dedicated.
    
3. Delete **NAT Gateways**.
    
4. Release **Elastic IPs**.
    
5. (If lab only) Detach & delete the **IGW**, subnets, and the VPC.
    

---

## Summary

A NAT Gateway is the simplest, production‑ready way to provide **outbound internet** to private subnets. Keep it **one per AZ**, push heavy AWS‑service traffic to **VPC Endpoints**, and verify routes/associations carefully. With the steps above and the diagram, you can go from zero to a secure, internet‑enabled private tier in minutes.

---