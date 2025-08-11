---
title: "Mastering AWS IAM: Creating Users in Groups, Assigning EC2 & S3 Permissions, and Setting Custom Passwords"
datePublished: Mon Aug 11 2025 07:27:36 GMT+0000 (Coordinated Universal Time)
cuid: cme6sil77000o02jm19oncu8w
slug: mastering-aws-iam-creating-users-in-groups-assigning-ec2-and-s3-permissions-and-setting-custom-passwords
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1754897004168/c60b9425-d70e-4b69-b734-05a7f967ac21.jpeg
tags: aws, cloud-computing, aws-lambda, aws-certified-solutions-architect-associate, aws-iam, aws-iam-policies, iam-role-in-aws, aws-iam-user

---

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1754897146025/77e6a010-ea21-4ee1-90d2-d99e367234d0.png align="center")

## **Introduction**

Security and access control are critical in the cloud, and **AWS Identity and Access Management (IAM)** plays a key role in managing who can do what in your AWS environment. In this article, we will create an IAM user under a group, assign permissions for **Amazon EC2** and **Amazon S3**, and set a custom password for login.

This structured approach keeps user access **organized, secure, and easy to manage** — especially for teams.

---

## **What is AWS IAM?**

**AWS Identity and Access Management (IAM)** allows you to:

* Create and manage **users**, **groups**, and **roles**
    
* Assign **permissions** using **policies**
    
* Control access to **AWS services and resources**
    
* Enable **secure sign-in** for different team members
    

---

## **Step-by-Step Guide**

### **Step 1: Sign in to AWS Management Console**

* Log in with your **root account** or an existing **IAM admin user**.
    
* Go to **IAM** service from the search bar.
    

---

### **Step 2: Create a Group**

1. In the **IAM dashboard**, click **User groups → Create group**.
    
2. Enter a **Group name** (e.g., `Developers_Group`).
    
3. Attach policies:
    
    * `AmazonEC2FullAccess`
        
    * `AmazonS3FullAccess`
        
4. Click **Create group**.
    

✅ **Why groups?** — They help manage permissions for multiple users at once, saving time.

---

### **Step 3: Create a User and Add to Group**

1. Go to **Users → Add users**.
    
2. Enter **User name** (e.g., `john.doe`).
    
3. Select **Password - AWS Management Console access**.
    
4. Set a **custom password** (e.g., `Cloud@1234`) and uncheck *Require password reset* if you want them to keep it.
    
5. Click **Next: Permissions**.
    
6. Choose **Add user to group** and select `Developers_Group`.
    
7. Click **Create user**.
    

---

### **Step 4: Test User Access**

* Log out from your admin account.
    
* Sign in using the **IAM user link**, username, and password you created.
    
* Verify if the user can:
    
    * Launch EC2 instances
        
    * Upload files to S3 buckets
        

---

## **IAM Diagram**

Here’s a simple diagram showing how it works:

```plaintext
   [Admin User]
        |
        v
   Create Group ---- Attach EC2 & S3 Policies
        |
        v
   Create User ----> Add to Group
        |
        v
   Custom Password ----> User Logs In
```

---

## **Best Practices**

* Always use **groups** for permission assignment instead of giving policies to users directly.
    
* Enable **MFA** (Multi-Factor Authentication) for added security.
    
* Assign the **least privilege** necessary — don’t give full access unless needed.
    

---

## **Conclusion**

By using **AWS IAM** to create users under groups and assign specific permissions, you keep your cloud environment secure and manageable. Adding a **custom password** makes onboarding easier while keeping login credentials under control.

With this setup, your team can work efficiently on EC2 and S3 without compromising security.

---