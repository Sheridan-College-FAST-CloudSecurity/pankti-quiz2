# Step-by-Step Lab: Multi-Environment Terraform Configuration

## Step 1: Project Setup

> **Explanation**: This step sets up the project directory and ensures you have the necessary files to structure your Terraform configuration.

Create a project folder:

```bash
mkdir terraform-multi-env
cd terraform-multi-env
```

Create the following Terraform configuration files:

- `main.tf`
- `variables.tf`
- `output.tf`

---

## Step 2: Define Variables for `prod` VPC (`variables.tf`)

> **Explanation**: This step defines the necessary variables for the `prod` VPC and subnets. These variables will be used to configure resources in the following steps.

Add the following code to define variables specific to the `prod` environment:

```hcl
variable "vpc_cidr" {
  description = "VPC CIDR range for the prod environment"
  type        = string
  default     = "10.0.0.0/16"
}

variable "public_subnet_cidrs" {
  description = "Public subnet CIDR ranges for the prod environment"
  type        = list(string)
  default     = ["10.0.1.0/24", "10.0.2.0/24"]
}

variable "private_subnet_cidrs" {
  description = "Private subnet CIDR ranges for the prod environment"
  type        = list(string)
  default     = ["10.0.3.0/24", "10.0.4.0/24"]
}
```

---

## Step 3: Build the `prod` Environment Without Loops

> **Explanation**: This step creates the `prod` environment without using loops. Resources are defined explicitly for the `prod` VPC and its subnets.

Add the following code to `main.tf` to create the `prod` environment manually:

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidr
  tags = {
    Name = "prod-vpc"
  }
}

resource "aws_subnet" "public_subnet_1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.public_subnet_cidrs[0]
  map_public_ip_on_launch = true
  tags = {
    Name = "prod-public-subnet-1"
  }
}

resource "aws_subnet" "public_subnet_2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.public_subnet_cidrs[1]
  map_public_ip_on_launch = true
  tags = {
    Name = "prod-public-subnet-2"
  }
}

resource "aws_subnet" "private_subnet_1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.private_subnet_cidrs[0]
  map_public_ip_on_launch = false
  tags = {
    Name = "prod-private-subnet-1"
  }
}

resource "aws_subnet" "private_subnet_2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = var.private_subnet_cidrs[1]
  map_public_ip_on_launch = false
  tags = {
    Name = "prod-private-subnet-2"
  }
}
```

---

## Step 4: Refactor to Use Loops

> **Explanation**: This step uses loops to simplify resource creation. 'for_each' and 'count' are both used to generate multiple resources, but they work differently. 'for_each' allows explicit mapping of keys to values, making it ideal for named resources or more complex configurations. On the other hand, 'count' creates a simple list-based index, which is straightforward but less flexible when resource names or keys are significant.

Refactor the subnet creation in `main.tf` to use loops:

```hcl
resource "aws_subnet" "public" {
  for_each = toset(var.public_subnet_cidrs)
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = true
  tags = {
    Name = "prod-public-subnet-${each.key}"
  }
}

resource "aws_subnet" "private" {
  for_each = toset(var.private_subnet_cidrs)
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = false
  tags = {
    Name = "prod-private-subnet-${each.key}"
  }
}
```

---

## Step 5: Update `variables.tf` for Multi-Environment Support

> **Explanation**: This step updates the variables file to handle both `prod` and `dev` environments using an object structure. By leveraging variables of type `object`, we can efficiently store complex, structured data for both environments. This approach allows for clear separation of configuration details, reduces duplication, and simplifies dynamic resource creation based on the chosen environment.

Modify `variables.tf` to support both `prod` and `dev` environments:

```hcl
variable "environment" {
  description = "Name of the environment (prod or dev)"
  type        = string
  default     = "prod"
}

variable "vpc_cidrs" {
  description = "VPC CIDR ranges for prod and dev"
  type        = object({
    prod = string
    dev  = string
  })
  default = {
    prod = "10.0.0.0/16"
    dev  = "10.1.0.0/16"
  }
}

variable "subnet_cidrs" {
  description = "Subnet CIDR ranges for prod and dev environments"
  type        = object({
    prod = object({
      public  = ["10.0.1.0/24", "10.0.2.0/24"]
      private = ["10.0.3.0/24", "10.0.4.0/24"]
    })
    dev = object({
      public  = ["10.1.1.0/24", "10.1.2.0/24"]
      private = []
    })
  })
  default = {
    prod = {
      public  = ["10.0.1.0/24", "10.0.2.0/24"]
      private = ["10.0.3.0/24", "10.0.4.0/24"]
    }
    dev = {
      public  = ["10.1.1.0/24", "10.1.2.0/24"]
      private = []
    }
  }
}
```

---

## Step 6: Add Validation to Variables

> **Explanation**: Adding validation rules ensures that the variables provided are correct and prevent misconfigurations.

Update the `variables.tf` file to include validation rules:

```hcl
variable "environment" {
  description = "Name of the environment (prod or dev)"
  type        = string
  default     = "prod"

  validation {
    condition     = contains(["prod", "dev"], var.environment)
    error_message = "The environment must be either 'prod' or 'dev'."
  }
}

variable "vpc_cidrs" {
  description = "VPC CIDR ranges for prod and dev"
  type        = object({
    prod = string
    dev  = string
  })
  default = {
    prod = "10.0.0.0/16"
    dev  = "10.1.0.0/16"
  }

  validation {
    condition     = can(regex("^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}/\d{1,2}$", var.vpc_cidrs.prod))
    error_message = "CIDR block for prod must be a valid CIDR."
  }
}
```

---

## Step 7: Add Common Tags to Resources

> **Explanation**: Adding common tags to all resources helps in resource identification and organization. This step ensures all resources are tagged with `Environment`, `StudentName`, and `Project` details.

Update `variables.tf` to include a variable for common tags:

```hcl
variable "tags" {
  description = "Tags to apply to all resources"
  type        = map(string)
  default = {
    Project     = "CloudSecurityLab"
    Environment = "prod"
    Owner       = "Student"
  }
}
```

Modify each resource in `main.tf` to include the tags using `merge`:

```hcl
resource "aws_vpc" "main" {
  cidr_block = var.vpc_cidrs[var.environment]
  tags       = merge(var.tags, { Name = "${var.environment}-vpc" })
}

resource "aws_subnet" "public" {
  for_each = toset(var.subnet_cidrs[var.environment].public)
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = true
  tags       = merge(var.tags, { Name = "${var.environment}-public-subnet-${each.key}" })
}

resource "aws_subnet" "private" {
  for_each = toset(var.subnet_cidrs[var.environment].private)
  vpc_id     = aws_vpc.main.id
  cidr_block = each.value
  map_public_ip_on_launch = false
  tags       = merge(var.tags, { Name = "${var.environment}-private-subnet-${each.key}" })
}
```

---

## Step 8: Outputs

> **Explanation**: Outputs help you verify the created resources and make their IDs accessible for further use.

Add the following to `output.tf` to verify resources:

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
}
```

Apply the configuration again and verify the outputs:

```bash
terraform apply
```

---

## Step 9: Add Security Groups

> **Explanation**: Security groups provide fine-grained access control to resources in the VPC. This step adds a security group configuration that dynamically adapts based on the environment.

Update `variables.tf` to define allowed CIDR blocks and ports:

```hcl
variable "allowed_ip" {
  description = "IP range allowed to access resources"
  type        = string
  default     = "0.0.0.0/0"
}

variable "environment_ports" {
  description = "Ports to allow based on environment"
  type        = map(object({
    ingress = list(number)
    egress  = list(number)
  }))
  default = {
    prod = { ingress = [80, 443], egress = [0] }
    dev  = { ingress = [8080], egress = [0] }
  }
}
```

Add a security group resource to `main.tf`:

```hcl
resource "aws_security_group" "main" {
  name        = "${var.environment}-sg"
  vpc_id      = aws_vpc.main.id
  description = "Security group for ${var.environment} environment"

  ingress {
    from_port   = var.environment_ports[var.environment].ingress[0]
    to_port     = var.environment_ports[var.environment].ingress[-1]
    protocol    = "tcp"
    cidr_blocks = [var.allowed_ip]
  }

  egress {
    from_port   = var.environment_ports[var.environment].egress[0]
    to_port     = var.environment_ports[var.environment].egress[-1]
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Add an output to `output.tf`:

```hcl
output "security_group_id" {
  value = aws_security_group.main.id
}
```

> **Verification**: Apply the configuration and verify the security group is created with the correct ingress and egress rules based on the environment.

```bash
terraform apply
```

---

## Step 10: Cleanup

> **Explanation**: Use this step to clean up all resources created during the lab.

Add `terraform destroy` to clean up resources:

```bash
terraform destroy
```

