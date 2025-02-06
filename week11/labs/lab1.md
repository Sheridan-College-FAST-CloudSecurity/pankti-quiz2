# Lab: Secure MySQL RDS Deployment Using Terraform

This hands-on lab guides students through deploying a **MySQL RDS database** in an AWS environment using Terraform. The lab covers key **DevSecOps practices**, including **Terraform functions**, **state management best practices**, and **sensitive data handling**.

---

## Lab Objectives

1. Use Terraform functions (`join`, `map`, `lookup`) for dynamic and modular configurations.
2. Implement **Terraform state best practices** by storing the state in an encrypted S3 bucket with state locking using DynamoDB.
3. Handle sensitive data securely by storing the MySQL database password in AWS Secrets Manager and referencing it in Terraform.

---

## Pre-requisites

1. **AWS Resources to Create Before the Lab**:
   - An **S3 bucket** for storing Terraform state (with versioning and encryption enabled).
   - A **DynamoDB table** for state locking.
   - A **KMS key** to encrypt the S3 bucket.
   - IAM roles with required permissions for S3, DynamoDB, and Secrets Manager.

2. **Terraform Installed**: Ensure you have the latest version of Terraform installed on your system.
3. **AWS CLI Configured**: Your AWS CLI should be authenticated and configured to access your AWS account.

---

## Step 1: Setup Terraform Backend for Remote State

### Objective: Configure Terraform to store its state file securely in S3 with encryption and locking.

1. Create a `backend.tf` file to configure the remote backend:

```hcl
terraform {
  backend "s3" {
    bucket         = "my-secure-terraform-state"    # Replace with your bucket name
    key            = "rds/mysql/terraform.tfstate"  # Specify the state file path
    region         = "us-east-1"
    encrypt        = true
    kms_key_id     = "alias/my-kms-key"             # Use your pre-created KMS key alias
    dynamodb_table = "terraform-state-lock"         # Replace with your DynamoDB table name
  }
}
```

Ensure the S3 bucket has:

- Server-Side Encryption (using the specified KMS key).
- Bucket Versioning enabled.
- Proper access controls (restrict access to the Terraform IAM role).

Initialize Terraform to use the backend:

```bash
terraform init
```

---

## Step 2: Configure Subnet Group and MySQL Database

### Subnet Group Configuration

Define a subnet group in Terraform to use the default VPC subnets:

```hcl
resource "aws_db_subnet_group" "default" {
  name       = "default-subnet-group"
  description = "Subnet group for RDS using default VPC subnets"

  subnet_ids = data.aws_subnets.default.ids

  tags = {
    Name = "default-subnet-group"
  }
}

# Data source to fetch default subnets
data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

# Data source to fetch the default VPC
data "aws_vpc" "default" {
  default = true
}
```

### MySQL Database Instance Configuration

Reference the subnet group in your `aws_db_instance` configuration:

```hcl
resource "aws_db_instance" "mysql" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = lookup(var.instance_sizes, var.environment)  # Use lookup function
  name                 = "mydb"
  username             = "admin"
  password             = data.aws_secretsmanager_secret_version.db_password.secret_string  # Reference secret
  publicly_accessible  = false
  db_subnet_group_name = aws_db_subnet_group.default.name  # Reference the subnet group

  tags = {
    Environment = var.environment
    Name        = join("-", ["mysql", var.environment])  # Use join function for tag value
  }
}
```

---

## Step 3: Handle Sensitive Data Securely

### Objective: Store the MySQL password securely in AWS Secrets Manager and reference it in Terraform.

1. Manually create the secret in AWS Secrets Manager:

```bash
aws secretsmanager create-secret \
  --name "mysql-rds-password" \
  --secret-string "SuperSecurePassword123!"  # Replace with your secure password
```

2. Add a data block in Terraform to retrieve the secret:

```hcl
data "aws_secretsmanager_secret_version" "db_password" {
  secret_id = "mysql-rds-password"  # Replace with your secret name
}
```

3. Update the RDS resource to reference the secret for the password field:

```hcl
resource "aws_db_instance" "mysql" {
  # Other configuration remains the same...
  password = data.aws_secretsmanager_secret_version.db_password.secret_string
}
```

---

## Step 4: Deploy the Infrastructure

1. Initialize the Terraform workspace:

```bash
terraform init
```

2. Validate the configuration:

```bash
terraform validate
```

3. Generate an execution plan:

```bash
terraform plan -var="environment=dev"
```

4. Apply the configuration to deploy the RDS instance:

```bash
terraform apply -var="environment=dev"
```

---

## Step 5: Verify the Deployment

1. Log in to the AWS Console:
   - Check the RDS instance in the region specified.
   - Ensure the tags are applied correctly (Environment=dev, Name=mysql-dev).

2. Verify the Terraform state:
   - Check the S3 bucket to ensure the state file is stored securely and encrypted.
   - Check the DynamoDB table for state locking entries.

3. Test the secret:
   - Verify the password in AWS Secrets Manager matches the password used in the RDS deployment.

---

## Cleanup

1. Destroy the infrastructure to avoid unnecessary costs:

```bash
terraform destroy -var="environment=dev"
```

2. Delete the secret from AWS Secrets Manager:

```bash
aws secretsmanager delete-secret --secret-id "mysql-rds-password"
```

3. Review and clean up S3 and DynamoDB resources if not required.



