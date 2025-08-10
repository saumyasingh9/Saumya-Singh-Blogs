---
title: "Creating and Connecting AWS VPCs with Transit Gateway — Access via MobaXterm on Windows"
datePublished: Sun Aug 10 2025 12:30:54 GMT+0000 (Coordinated Universal Time)
cuid: cme5nwsgd000i02l81yod940y
slug: creating-and-connecting-aws-vpcs-with-transit-gateway-access-via-mobaxterm-on-windows
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1754828674909/e8bf7544-025b-4fd1-ad03-f81844f21e45.jpeg
tags: aws, cloud-computing, devops, aws-lambda, aws-vpc, aws-certified-solutions-architect-associate, aws-cloud-practitioner, cloud-security, aws-vpc-introduction, aws-vpc-beginners, aws-vpc-creation

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754828853854/d1c6fa42-c036-4c31-b9bf-6d7f99c8b4a7.png align="center")

**Introduction**

In modern cloud architectures, it's common to have multiple Virtual Private Clouds (VPCs) for isolation, security, and scaling purposes. But what if your workloads in different VPCs need to communicate securely and efficiently?  
That’s where **AWS Transit Gateway** comes in — acting as a central hub to connect your VPCs, on-premises networks, and more.

In this tutorial, we’ll:

* Create multiple VPCs in AWS.
    
* Connect them using a **Transit Gateway**.
    
* Access EC2 instances from Windows using **MobaXterm** for SSH connectivity.
    

---

## **Architecture Diagram**

Below is the high-level architecture:

```plaintext
         +-------------------+             +-------------------+
         |    VPC 1 (Public) |             |    VPC 2 (Private) |
         |  EC2 Instance A   |             |  EC2 Instance B   |
         +--------+----------+             +--------+----------+
                  \                              /
                   \                            /
                    +-------- Transit ---------+
                    |        Gateway           |
                    +--------------------------+
                             |
                             |
                        AWS Backbone
```

---

## **Step 1: Create VPCs**

1. **Login to AWS Management Console** → Go to **VPC** service.
    
2. **Create VPC 1** (Public subnet)
    
    * CIDR Block: `10.0.0.0/16`
        
    * Create a **Public Subnet**: `10.0.1.0/24`
        
    * Enable **Auto-assign Public IP**.
        
3. **Create VPC 2** (Private subnet)
    
    * CIDR Block: `10.1.0.0/16`
        
    * Create a **Private Subnet**: `10.1.1.0/24`
        

---

## **Step 2: Launch EC2 Instances**

* **In VPC 1 (Public)** → Launch an **Amazon Linux** or **Ubuntu** instance. Assign a public IP.
    
* **In VPC 2 (Private)** → Launch an instance without a public IP (we’ll reach it via VPC connectivity).
    

---

## **Step 3: Create a Transit Gateway**

1. Navigate to **Transit Gateway** in AWS Console.
    
2. Click **Create Transit Gateway**:
    
    * Name: `TGW-Demo`
        
    * Amazon ASN: Keep default (e.g., 64512).
        
3. Once created, note the **Transit Gateway ID**.
    

---

## **Step 4: Attach VPCs to Transit Gateway**

* Go to **Transit Gateway Attachments** → **Create attachment**.
    
* Select **VPC 1** and attach it to TGW.
    
* Repeat for **VPC 2**.
    

---

## **Step 5: Update Route Tables**

* **For VPC 1 Route Table** → Add a route for `10.1.0.0/16` pointing to **Transit Gateway**.
    
* **For VPC 2 Route Table** → Add a route for `10.0.0.0/16` pointing to **Transit Gateway**.
    

---

## **Step 6: Access EC2 Instances from Windows (MobaXterm)**

**Install MobaXterm**:

* Download from [https://mobaxterm.mobatek.net/](https://mobaxterm.mobatek.net/) and install.
    

**Connect to VPC 1 EC2 Instance**:

1. Open MobaXterm → Click **Session** → SSH.
    
2. Enter **Public IP** of VPC 1 EC2.
    
3. Browse and select your `.pem` key in **Advanced SSH settings**.
    

**Access VPC 2 EC2 via VPC 1**:

* First SSH into **VPC 1 EC2**.
    
* From there, run:
    
    ```bash
    ssh -i "key.pem" ec2-user@<Private-IP-of-VPC2-EC2>
    ```
    

---

## **Conclusion**

With **AWS Transit Gateway**, we’ve connected two VPCs to enable secure communication without complex peering configurations. By using **MobaXterm**, we easily accessed the EC2 instances from our Windows machine — even those in private subnets.

---

## **Learning Outcomes**

* Understand how Transit Gateway acts as a central networking hub.
    
* Configure route tables for inter-VPC communication.
    
* Use MobaXterm for SSH access to EC2 instances across VPCs.
    

---