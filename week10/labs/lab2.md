# Lab: Managing Terraform State in S3 for Dev and Prod Environments

## Target Audience: Cloud Security Program Students

### Objectives:
- Understand the importance of managing Terraform state remotely.
- Learn how to configure remote state in S3 for separate environments (dev and prod).
- Migrate existing local state to remote state in S3.

### Prerequisites:
- AWS account with access to S3.
- AWS CLI installed and configured.
- Terraform installed.
- Basic understanding of Terraform and AWS services.

---

## Step-by-Step Guide

### Step 1: Create an S3 Bucket for Terraform State

1. **Open your terminal** and create an S3 bucket to store the Terraform state for both environments (`dev` and `prod`).

    ```sh
    aws s3api create-bucket --bucket my-terraform-state --region us-east-1 --create-bucket-configuration LocationConstraint=us-east-1
    ```

2. **Verify** that the bucket has been created.

    ```sh
    aws s3 ls
    ```

### Step 2: Configure Remote State for a New Project

1. **Create a new Terraform configuration** file, `main.tf`, in a new directory.

    ```hcl
    provider "aws" {
      region = "us-east-1"
    }

    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"
    }
    ```

2. **Create a `backend.tf` file** to configure the backend for both environments.

    ```hcl
    terraform {
      backend "s3" {
        bucket = "my-terraform-state"
        region = "us-east-1"
      }
    }
    ```

3. **Create a `variables.tf` file** to define the environment variable.

    ```hcl
    variable "env" {
      description = "The environment for this deployment (dev or prod)"
      type        = string
    }
    ```

4. **Create a `terraform.tfvars` file** to specify the environment.

    ```hcl
    env = "dev"
    ```

5. **Initialize Terraform** to configure the backend using `backend-config`.

    ```sh
    terraform init -backend-config="key=${var.env}/terraform.tfstate"
    ```

6. **Apply the Terraform configuration** to the `dev` environment.

    ```sh
    terraform apply
    ```

7. **Check the S3 bucket** to verify that the state file has been created.

    ```sh
    aws s3 ls s3://my-terraform-state/dev/
    ```

8. **Repeat steps 4-7** for the `prod` environment by updating the `terraform.tfvars` file.

    ```hcl
    env = "prod"
    ```

    Then reinitialize and apply:

    ```sh
    terraform init -backend-config="key=${var.env}/terraform.tfstate"
    terraform apply
    ```

    Verify:

    ```sh
    aws s3 ls s3://my-terraform-state/prod/
    ```

### Step 3: Migrate an Existing Local State to Remote State

1. **Create an existing Terraform configuration** file, `main.tf`, in a new directory.

    ```hcl
    provider "aws" {
      region = "us-east-1"
    }

    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"
    }
    ```

2. **Create a `backend.tf` file** to configure the backend for both environments.

    ```hcl
    terraform {
      backend "s3" {
        bucket = "my-terraform-state"
        region = "us-east-1"
      }
    }
    ```

3. **Create a `variables.tf` file** to define the environment variable.

    ```hcl
    variable "env" {
      description = "The environment for this deployment (dev or prod)"
      type        = string
    }
    ```

4. **Create a `terraform.tfvars` file** to specify the environment.

    ```hcl
    env = "dev"
    ```

5. **Initialize Terraform** to generate the local state file.

    ```sh
    terraform init
    terraform apply
    ```

6. **Reinitialize Terraform** to migrate the local state to the remote state using `backend-config`.

    ```sh
    terraform init -migrate-state -backend-config="key=${var.env}/terraform.tfstate"
    ```

7. **Verify the migration** by checking the S3 bucket.

    ```sh
    aws s3 ls s3://my-terraform-state/dev/
    ```

8. **Repeat steps 4-7** for the `prod` environment by updating the `terraform.tfvars` file.

    ```hcl
    env = "prod"
    ```

    Then reinitialize and apply:

    ```sh
    terraform init -migrate-state -backend-config="key=${var.env}/terraform.tfstate"
    terraform apply
    ```

    Verify:

    ```sh
    aws s3 ls s3://my-terraform-state/prod/
    ```

---

### Additional Resources:
- [Terraform State](https://www.terraform.io/docs/state/index.html)
- [Terraform Backends](https://www.terraform.io/docs/backends/index.html)
- [AWS S3](https://aws.amazon.com/s3/)

