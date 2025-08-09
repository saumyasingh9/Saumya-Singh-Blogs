---
title: "Building a Public & Private Cloud Network in AWS: VPC Setup, EC2 Launch, and Cross-Instance Connectivity"
datePublished: Sat Aug 09 2025 13:45:47 GMT+0000 (Coordinated Universal Time)
cuid: cme4b58sj000402iecu5dcd8y
slug: building-a-public-and-private-cloud-network-in-aws-vpc-setup-ec2-launch-and-cross-instance-connectivity
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1754747013845/3380ffbf-0134-4aeb-9dc5-5df6567d8457.jpeg
tags: aws, cloud-computing, aws-lambda, aws-certified-solutions-architect-associate, vpc-peering, cloud-security, vpc-basics, aws-vpc-beginners, aws-vpc-creation

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754747066885/24cf5810-bc8a-4273-a6a6-53627b093df2.png align="center")

**Introduction**

Amazon Web Services (AWS) offers robust networking capabilities through **Virtual Private Cloud (VPC)**, allowing you to design secure, isolated network environments. In this guide, we’ll:

* Create a **VPC** with both public and private subnets
    
* Launch **EC2 instances** in each subnet
    
* Connect to instances using **MobaXterm** from a Windows machine
    
* Test cross-instance connectivity by **pinging IP addresses**
    

By the end, you’ll understand how public and private networks work together in AWS.

---

## **Step-by-Step Guide**

### **1\. Create a VPC**

1. Go to the AWS Management Console → **VPC Service**
    
2. Click **Create VPC**
    
    * **Name:** My-VPC
        
    * **IPv4 CIDR block:** `10.0.0.0/16`
        
    * Tenancy: Default
        

---

### **2\. Create Public and Private Subnets**

1. **Public Subnet**
    
    * Name: `Public-Subnet`
        
    * CIDR: `10.0.1.0/24`
        
    * Availability Zone: `ap-south-1a`
        
    * Enable **Auto-assign public IP**: Yes
        
2. **Private Subnet**
    
    * Name: `Private-Subnet`
        
    * CIDR: `10.0.2.0/24`
        
    * Availability Zone: `ap-south-1a`
        
    * Auto-assign public IP: No
        

---

### **3\. Create an Internet Gateway (IGW)**

* Go to **Internet Gateways** → Create IGW → Attach it to **My-VPC**
    

---

### **4\. Create Route Tables**

1. **Public Route Table**
    
    * Associate with Public Subnet
        
    * Add route: `0.0.0.0/0` → Internet Gateway
        
2. **Private Route Table**
    
    * Associate with Private Subnet
        
    * No internet route (for private network isolation)
        

---

### **5\. Launch EC2 Instances**

1. **Public Instance**
    
    * AMI: Amazon Linux 2
        
    * Instance type: t2.micro
        
    * Subnet: Public-Subnet (auto-assign public IP)
        
    * Security Group: Allow **SSH** from your IP, Allow **ICMP** (ping)
        
2. **Private Instance**
    
    * Same AMI & type
        
    * Subnet: Private-Subnet (no public IP)
        
    * Security Group: Allow **ICMP** and **SSH** from the Public Instance’s private IP range
        

---

### **6\. Connect via MobaXterm (Windows)**

1. Download & install **MobaXterm** on your Windows machine
    
2. Connect to **Public Instance** using `.pem` key
    
3. From the public instance, **SSH into the private instance**:
    
    ```bash
    ssh -i mykey.pem ec2-user@<Private-Instance-Private-IP>
    ```
    

---

### **7\. Test Connectivity Between Instances**

1. From Public Instance:
    
    ```bash
    ping <Private-Instance-Private-IP>
    ```
    
2. From Private Instance:
    
    ```bash
    ping <Public-Instance-Private-IP>
    ```
    

If ICMP is allowed in both security groups, you should see successful replies.

---

## **Architecture Diagram**

Here’s a visual representation:

```plaintext
   +-------------------+              +-------------------+
   |   Public Subnet   |              |   Private Subnet  |
   |  EC2: Public Inst |              |  EC2: Private Inst|
   |   10.0.1.x        |              |   10.0.2.x        |
   +---------+---------+              +---------+---------+
             |                                   |
        Internet GW                          No Internet
             |
         Your PC (MobaXterm)
```

---

## **Conclusion**

By setting up a **VPC** with both public and private subnets, we achieve secure, segmented networking in AWS. The **public subnet** allows external access (via the Internet Gateway), while the **private subnet** ensures internal-only resources. Using **MobaXterm** simplifies remote access from Windows, and **ping tests** validate internal communication.

---

## **Key Learnings**

* Public subnets provide internet accessibility; private subnets keep workloads secure.
    
* Security groups and routing tables control traffic flow.
    
* MobaXterm is a powerful SSH client for Windows users.
    
* Internal networking in AWS can be tested using simple ping commands.
    

---