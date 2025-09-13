---
title: "Connect Terraform to WSL2 and Set Up AWS — A Practical Step‑by‑Step Guide"
datePublished: Sat Sep 13 2025 21:10:43 GMT+0000 (Coordinated Universal Time)
cuid: cmfirg8fd000002jperlj4grf
slug: connect-terraform-to-wsl2-and-set-up-aws-a-practical-stepbystep-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1757797829301/5c7a09a9-9d1f-4d1e-8512-5c2ff2e86276.png

---

## Why this setup?

WSL2 gives you a lightweight Linux environment inside Windows, which is perfect for running Terraform and the AWS CLI as if you were on a Linux server. This guide walks you from zero → working Terraform project using AWS as a provider.

**Prerequisites**

* Windows 10 (build 19041+) or Windows 11 with WSL2 support. (If you already use WSL2, you’re ahead of the game.)
    
* A user account with admin rights to run PowerShell commands.
    
* An AWS account and permission to create an IAM user (or use SSO).
    
* Basic familiarity with terminal commands.
    

---

## TL;DR — Commands you’ll run

> (Run these at the right step below.)

```bash
# On Windows PowerShell (Admin) - install WSL and Ubuntu
wsl --install -d ubuntu
wsl --set-default-version 2

# Inside WSL (Ubuntu)
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl gnupg lsb-release unzip

# Install Terraform (using HashiCorp repo)
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y terraform
terraform -v

# Install AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version

# Configure AWS credentials
aws configure --profile myprofile

# Example Terraform flow (in project folder)
terraform init
terraform plan -out=tfplan
terraform apply "tfplan"
```

---

## Step 1 — Install and check WSL2 (Windows side)

1. Open PowerShell **as Administrator**.
    
2. Install WSL with a modern Ubuntu distribution:
    

```powershell
wsl --install -d ubuntu
```

3. (Optional) Ensure default WSL version is 2:
    

```powershell
wsl --set-default-version 2
wsl -l -v   # lists installed distros and versions
```

4. Launch Ubuntu (from Start menu) and complete first-time user setup.
    

---

## Step 2 — Prepare your WSL Ubuntu environment

Inside your new Ubuntu prompt run:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl gnupg lsb-release unzip
```

These packages are needed to fetch repositories, unzip installers, and verify signatures.

Tip: if you use VS Code, install the **Remote - WSL** extension in Windows so you can edit files directly in the WSL filesystem (`\wsl$\`), and use `code .` from your WSL shell.

---

## Step 3 — Install Terraform in WSL2 (recommended: HashiCorp APT repo)

Add HashiCorp’s official APT repository and install Terraform:

```bash
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y terraform

# Verify
terraform -v
```

If `terraform -v` prints a version, you’re good.

---

## Step 4 — Install and configure AWS CLI v2

Download and install AWS CLI v2 inside WSL:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws --version
```

Now configure credentials for an AWS IAM user (or SSO):

```bash
aws configure --profile myprofile
# Follow prompts: AWS Access Key ID, Secret Access Key, region (e.g. ap-south-1), output (json)
```

This writes `~/.aws/credentials` and `~/.aws/config` in WSL.

**If you store credentials in Windows and want to reuse them in WSL:**

```bash
# replace 'YourWindowsUser' with your actual Windows username
ln -s /mnt/c/Users/YourWindowsUser/.aws ~/.aws
```

That symlink reuses the same credentials files so you don’t duplicate secrets.

Security note: never commit `~/.aws/credentials` to git. Add it to `~/.gitignore` in projects.

---

## Step 5 — Create a simple Terraform project (test)

Create a folder for the tutorial:

```bash
mkdir ~/tf-wsl-aws-demo && cd ~/tf-wsl-aws-demo
```

Create a [`main.tf`](http://main.tf) file with this minimal example that creates an S3 bucket (it uses the `random` provider to add a stable random suffix so bucket names stay unique):

```plaintext
terraform {
  required_providers {
    aws = { source = "hashicorp/aws" }
    random = { source = "hashicorp/random" }
  }
}

provider "aws" {
  region  = "ap-south-1"    # change to your region
  profile = "myprofile"     # matches the aws configure profile you set earlier
}

resource "random_pet" "suffix" {
  length = 2
}

resource "aws_s3_bucket" "demo" {
  bucket = "tf-wsl2-demo-${random_pet.suffix.id}"
  acl    = "private"

  tags = {
    Name = "TF WSL2 Demo"
    Env  = "dev"
  }
}
```

**Run Terraform:**

```bash
terraform init
terraform plan -out=tfplan
terraform apply "tfplan"
```

If everything is configured correctly, Terraform will create the S3 bucket in your AWS account.

To destroy the resources later:

```bash
terraform destroy -auto-approve
```

---

## Step 6 — Environment variables & profiles (alternatives)

Terraform will discover AWS credentials in this order (common paths):

* Environment variables `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` (if present)
    
* `~/.aws/credentials` for the profile named in the `provider` block
    
* EC2/ECS instance role metadata (when running on AWS)
    

To use environment variables in your shell session:

```bash
export AWS_PROFILE=myprofile
export AWS_REGION=ap-south-1
```

Using `AWS_PROFILE` is convenient when switching between accounts.

---

## Troubleshooting (common issues)

`terraform: command not found`

* Make sure `/usr/bin/terraform` is installed and your PATH includes `/usr/bin`.
    

`aws: command not found`

* Re-run the AWS CLI installer (`sudo ./aws/install`) and ensure `/usr/local/bin` is in PATH.
    

**Credentials errors (AccessDenied / InvalidClientTokenId)**

* Confirm keys are correct: `aws sts get-caller-identity --profile myprofile`
    
* If you use SSO or temporary tokens, ensure the session is valid.
    

**Bucket name collision or invalid bucket name**

* S3 bucket names are global. Use random suffixes or unique prefixes.
    

---

## Good practices & next steps

* **Don’t commit secrets** — add `~/.aws` and any files containing keys to `.gitignore`.
    
* **Use IAM least privilege** — create specific policies for Terraform-managed resources.
    
* **Use remote state** — store Terraform state in an S3 backend with state locking via DynamoDB.
    
* **Use** `aws-vault` or SSO for safer credential handling in local development.
    
* **Automate with CI/CD** — once local dev is stable, run `terraform plan` and `apply` from a pipeline using secure credentials (e.g. GitHub Actions with OIDC).
    

---

## Wrap up

You now have:

* a working WSL2 (Ubuntu) environment
    
* Terraform installed and verified in WSL
    
* AWS CLI v2 installed and configured
    
* a sample Terraform project that creates an AWS resource