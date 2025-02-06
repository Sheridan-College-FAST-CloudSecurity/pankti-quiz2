### **Lab 1: Building Cloud Infrastructure with Terraform in CodeSpaces**

### **Overview**
In this lab, students will:
1. Set up infrastructure using Terraform in GitHub CodeSpaces.
2. Use a local Terraform module to configure a VPC with multiple subnets, IGW, NAT GW, and route tables.
3. Deploy a database in a private subnet and a bastion host in a public subnet.
4. Configure the bastion host to run nginx and MySQL client containers via user data.
5. Validate Terraform configurations with Open Policy Agent (OPA) using Conftest.
6. Perform a security scan using Checkov.
7. Push the code to a non-default branch in a new GitHub repository.
8. Configure GitHub environments for dev and prod, and set up AWS credentials.

---

### **Step-by-Step Instructions**

#### **Step 1: Set Up CodeSpaces and Repository**
1. Create a new GitHub repository for this lab.
   - Name it `terraform-cicd-lab`.
   - Initialize it with a README file.
   - Enable branch protection rules to block direct merges into the default branch.
2. Open a new CodeSpaces environment for the repository.

#### **Step 2: Initialize Terraform and Create Module**
1. In CodeSpaces, create the following directory structure:
   ```
   terraform-cicd-lab/
   ├── modules/
   │   ├── vpc/
   │   │   ├── main.tf
   │   │   ├── outputs.tf
   │   │   └── variables.tf
   ├── main.tf
   ├── outputs.tf
   └── variables.tf
   ```
2. In the `modules/vpc/main.tf` file, define the VPC and its components:
   ```hcl
   resource "aws_vpc" "main" {
     cidr_block           = var.vpc_cidr
     enable_dns_support   = true
     enable_dns_hostnames = true
     tags = {
       Name = "main-vpc"
     }
   }

   resource "aws_subnet" "public" {
     count = 2
     vpc_id            = aws_vpc.main.id
     cidr_block        = element(var.public_subnet_cidrs, count.index)
     map_public_ip_on_launch = true
     availability_zone = element(var.availability_zones, count.index)
     tags = {
       Name = "public-subnet-${count.index}"
     }
   }

   resource "aws_subnet" "private" {
     count = 2
     vpc_id            = aws_vpc.main.id
     cidr_block        = element(var.private_subnet_cidrs, count.index)
     availability_zone = element(var.availability_zones, count.index)
     tags = {
       Name = "private-subnet-${count.index}"
     }
   }

   resource "aws_internet_gateway" "igw" {
     vpc_id = aws_vpc.main.id
     tags = {
       Name = "main-igw"
     }
   }

   resource "aws_nat_gateway" "nat" {
     allocation_id = aws_eip.nat.id
     subnet_id     = aws_subnet.public[0].id
     tags = {
       Name = "main-nat-gateway"
     }
   }

   resource "aws_eip" "nat" {
     vpc = true
   }

   resource "aws_route_table" "public" {
     vpc_id = aws_vpc.main.id
     route {
       cidr_block = "0.0.0.0/0"
       gateway_id = aws_internet_gateway.igw.id
     }
   }

   resource "aws_route_table" "private" {
     vpc_id = aws_vpc.main.id
     route {
       cidr_block = "0.0.0.0/0"
       nat_gateway_id = aws_nat_gateway.nat.id
     }
   }

   resource "aws_route_table_association" "public" {
     count          = 2
     subnet_id      = aws_subnet.public[count.index].id
     route_table_id = aws_route_table.public.id
   }

   resource "aws_route_table_association" "private" {
     count          = 2
     subnet_id      = aws_subnet.private[count.index].id
     route_table_id = aws_route_table.private.id
   }
   ```
3. Create variables and outputs in `variables.tf` and `outputs.tf`:
   ```hcl
   variable "vpc_cidr" {
     default = "10.0.0.0/16"
   }

   variable "public_subnet_cidrs" {
     default = ["10.0.1.0/24", "10.0.2.0/24"]
   }

   variable "private_subnet_cidrs" {
     default = ["10.0.3.0/24", "10.0.4.0/24"]
   }

   variable "availability_zones" {
     default = ["us-east-1a", "us-east-1b"]
   }
   
   output "vpc_id" {
     value = aws_vpc.main.id
   }
   
   output "public_subnets" {
     value = aws_subnet.public[*].id
   }

   output "private_subnets" {
     value = aws_subnet.private[*].id
   }
   ```

#### **Step 3: Configure the Main Terraform Code**
1. In `main.tf`, use the `vpc` module:
   ```hcl
   module "vpc" {
     source = "./modules/vpc"

     vpc_cidr           = var.vpc_cidr
     public_subnet_cidrs = var.public_subnet_cidrs
     private_subnet_cidrs = var.private_subnet_cidrs
     availability_zones = var.availability_zones
   }
   
   resource "aws_instance" "bastion" {
     ami           = "ami-12345678"
     instance_type = "t3.micro"
     subnet_id     = module.vpc.public_subnets[0]

     user_data = <<-EOT
       #!/bin/bash
       apt-get update
       apt-get install -y docker.io
       docker run -d -p 80:80 nginx
       docker run -d mysql:latest
     EOT
   }

   resource "aws_db_instance" "mysql" {
     engine            = "mysql"
     instance_class    = "db.t3.micro"
     allocated_storage = 20
     username          = "admin"
     password          = "password123"
     db_subnet_group_name = aws_db_subnet_group.main.name
   }
   ```

#### **Step 4: Install Conftest and Checkov in CodeSpaces**
1. Open the terminal in CodeSpaces.
2. Install Conftest:
   ```bash
   curl -L https://github.com/open-policy-agent/conftest/releases/download/v0.35.0/conftest_0.35.0_Linux_x86_64.tar.gz -o conftest.tar.gz
   tar -xvf conftest.tar.gz -C /usr/local/bin
   rm conftest.tar.gz
   ```
3. Install Checkov:
   ```bash
   pip install checkov
   ```
4. Verify the installations:
   ```bash
   conftest --version
   checkov --version
   ```

#### **Step 5: Test and Validate Locally**
1. Create a policy file `policy/terraform.rego`:
   ```rego
   package main

   deny[msg] {
     input.resource_type == "aws_instance"
     not startswith(input.instance_type, "t2")
     not startswith(input.instance_type, "t3")
     msg = sprintf("Instance type %s is not allowed. Only t2 and t3 families are permitted.", [input.instance_type])
   }
   ```
2. Run Conftest:
   ```bash
   conftest test .
   ```
3. Use Checkov for security scanning:
   ```bash
   checkov -d .
   ```

---

The lab now includes detailed Terraform configurations, an updated OPA policy example, and installation instructions for Conftest and Checkov in CodeSpaces.
