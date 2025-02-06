# Lab: Secure Deployment and Compliance Testing of AWS Infrastructure

## Step 1: Create the Terraform Configuration

This configuration deploys:

- An EC2 instance in the default VPC.
- A security group with ingress and egress rules.

Save this configuration as `main.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}

# Fetch the Default VPC
data "aws_vpc" "default" {
  default = true
}

# Security Group in Default VPC
resource "aws_security_group" "example" {
  name        = "example-security-group"
  description = "Allow limited traffic"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["203.0.113.0/24"] # Replace with your IP range
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"] # Allow outbound traffic
  }
}

# EC2 Instance in Default VPC
resource "aws_instance" "example" {
  ami           = "ami-12345678" # Replace with a valid AMI ID
  instance_type = "t2.micro"
  security_groups = [aws_security_group.example.name]

  tags = {
    Name        = "example-instance"
    Environment = "production" # Required tag for compliance
  }
}
```

---

## Step 2: Initialize Terraform and Apply the Configuration

### Initialize Terraform
Run the following command to initialize Terraform:

```bash
terraform init
```

### Validate and Apply the Configuration
1. Generate a plan to validate the configuration:

    ```bash
    terraform plan
    ```

2. Apply the configuration:

    ```bash
    terraform apply
    ```

### Validation Steps
1. Log in to the AWS Management Console.
2. Navigate to EC2 Instances in the `us-east-1` region.
3. Confirm that:
   - The instance is deployed into the default VPC.
   - The security group is configured as expected.

---

## Step 3: Run tfsec for Security Scanning

### Instructions:
Run tfsec to scan the configuration:

```bash
tfsec .
```

### Example Output:
If issues are detected, tfsec will flag them. For example:

```plaintext
WARNING: Security group 'example-security-group' has overly permissive egress rules.
File: main.tf:20-25
```

### What to Fix:
- Restrict egress rules to VPC CIDR.
- Ensure tags like `Environment` are added for compliance.

---

## Step 4: Run Checkov for Compliance Scanning

### Instructions:
Run Checkov to validate compliance:

```bash
checkov -f main.tf
```

### Example Output:
Checkov validates the configuration against compliance policies:

```plaintext
Check CKV_AWS_23: Ensure every security group restricts ingress to specific ports.
File: main.tf:10-15
Result: PASSED
```

---

## Step 5: Fix Identified Issues

### Update the Configuration:

#### Restrict Security Group Rules:

```hcl
resource "aws_security_group" "example" {
  name        = "example-security-group"
  description = "Allow limited traffic"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["203.0.113.0/24"] # Replace with your IP range
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["10.0.0.0/16"] # Restrict to VPC CIDR
  }
}
```

### Add Tags for Compliance:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"
  security_groups = [aws_security_group.example.name]

  tags = {
    Name        = "example-instance"
    Environment = "production"
  }
}
```

---

## Step 6: Re-Scan and Verify

### Re-Run tfsec:

```bash
tfsec .
```

Ensure there are no critical warnings.

### Re-Run Checkov:

```bash
checkov -f main.tf
```

Ensure all compliance checks pass.

---

## Step 7: Key Takeaways

### What You Learned:

- How to use tfsec to identify security misconfigurations.
- How to use Checkov to validate Terraform configurations against compliance frameworks like CIS AWS Foundations Benchmark.
- How to fix issues and re-scan to verify compliance.



