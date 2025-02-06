# Lab: Deploying Infrastructure with Remote States and Local Modules

This lab builds on the previous lab and introduces the use of **remote state** for managing `dev` and `prod` environments and a **local module** to deploy VPCs. Follow the folder structure as shown in the provided diagram.

## Step 1: Set Up the Folder Structure

> **Explanation**: To organize the Terraform code, set up the folder structure as per the provided diagram. This structure separates environments (`dev` and `prod`) and modules.

Create the following folder structure:

```bash
terraform-project/
├── envs/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf
│   ├── prod/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   ├── terraform.tfvars
│   │   ├── backend.tf
├── modules/
│   ├── vpc/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
├── provider.tf
├── README.md
```

---

## Step 2: Create the Local VPC Module

> **Explanation**: The local module encapsulates the logic for creating a VPC and subnets. This makes the VPC configuration reusable for both environments.

### `modules/vpc/main.tf`

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags       = merge(var.tags, { Name = "${var.environment}-vpc" })
}

resource "aws_subnet" "public" {
  for_each = toset(var.public_subnet_cidrs)
  vpc_id   = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = true
  tags = merge(var.tags, { Name = "${var.environment}-public-subnet-${each.key}" })
}

resource "aws_subnet" "private" {
  for_each = toset(var.private_subnet_cidrs)
  vpc_id   = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = false
  tags = merge(var.tags, { Name = "${var.environment}-private-subnet-${each.key}" })
}
```

### `modules/vpc/variables.tf`

```hcl
variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
}

variable "public_subnet_cidrs" {
  description = "CIDR blocks for public subnets"
  type        = list(string)
}

variable "private_subnet_cidrs" {
  description = "CIDR blocks for private subnets"
  type        = list(string)
}

variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
}

variable "environment" {
  description = "Environment (dev or prod)"
  type        = string
}
```

### `modules/vpc/outputs.tf`

```hcl
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

---

## Step 3: Configure Remote States

> **Explanation**: Remote state ensures that each environment stores its Terraform state file in a separate S3 location.

### `envs/dev/backend.tf`

```hcl
terraform {
  backend "s3" {
    bucket         = "your-s3-bucket-name"
    key            = ""
    region         = "us-east-1"
  }
}
```

### `envs/prod/backend.tf`

```hcl
terraform {
  backend "s3" {
    bucket         = "your-s3-bucket-name"
    key            = ""
    region         = "us-east-1"
  }
}
```

---

## Step 4: Use the VPC Module in `dev` and `prod`

### `envs/dev/main.tf`

```hcl
module "vpc" {
  source              = "../../modules/vpc"
  vpc_cidr            = "10.1.0.0/16"
  public_subnet_cidrs = ["10.1.1.0/24", "10.1.2.0/24"]
  private_subnet_cidrs = ["10.1.3.0/24", "10.1.4.0/24"]
  tags                = {
    Project     = "CloudSecurityLab"
    Environment = "dev"
    Owner       = "Student"
  }
  environment         = "dev"
}
```

### `envs/prod/main.tf`

```hcl
module "vpc" {
  source              = "../../modules/vpc"
  vpc_cidr            = "10.0.0.0/16"
  public_subnet_cidrs = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnet_cidrs = ["10.0.3.0/24", "10.0.4.0/24"]
  tags                = {
    Project     = "CloudSecurityLab"
    Environment = "prod"
    Owner       = "Student"
  }
  environment         = "prod"
}
```

---

## Step 5: Initialize and Apply the Configurations

> **Explanation**: Initialize and apply the configurations for both `dev` and `prod` environments.

### For `dev`

```bash
cd terraform-project/envs/dev
terraform init -backend-config="key=dev/terraform.tfstate"
terraform apply
```

### For `prod`

```bash
cd terraform-project/envs/prod
terraform init -backend-config="key=prod/terraform.tfstate"
terraform apply
```

---

## Step 6: Verify the Outputs

> **Explanation**: Check the outputs to verify that the VPC and subnets are created correctly for both environments.

Run the following commands in each environment:

```bash
terraform output
```

---

## Step 7: Destroy the `dev` and `prod` Environments

> **Explanation**: Clean up resources by destroying the `dev` and `prod` deployments. This step ensures no leftover resources remain after testing.

### For `dev`

```bash
cd terraform-project/envs/dev
terraform destroy
```

### For `prod`

```bash
cd terraform-project/envs/prod
terraform destroy
```




