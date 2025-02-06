# Terraform Lab Instructions

## Step 1: Prerequisites in CodeSpace

Before starting, ensure the following are installed and set up:

- **Terraform CLI**: Download from [Terraform.io](https://www.terraform.io/).
- **AWS CLI**: For setting up credentials locally.

## Installing AWS CLI and OpenStack CLI on Codespaces

### AWS CLI Installation

1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install the AWS CLI:
   ```bash
   sudo apt install awscli -y
   ```

3. Verify the installation:
   ```bash
   aws --version
   ```

### OpenStack CLI Installation

1. Update the package list:
   ```bash
   sudo apt update
   ```

2. Install the OpenStack client:
   ```bash
   sudo apt install python3-openstackclient -y
   ```

3. Verify the installation:
   ```bash
   openstack --version
   ```

## Step 2: Set Up Environment Variables for Credentials

### Export AWS Credentials

```bash
export AWS_ACCESS_KEY_ID="your-aws-access-key-id"
export AWS_SECRET_ACCESS_KEY="your-aws-secret-access-key"
export AWS_DEFAULT_REGION="us-east-1"
export AWS_SESSION_TOKEN="your-aws-session-token"
```

### Export OpenStack Credentials

```bash
export OS_AUTH_URL="https://your-openstack-url"
export OS_PROJECT_NAME="your-project-name"
export OS_USERNAME="your-username"
export OS_PASSWORD="your-password"
```

> **Security Concern:** Use OpenStack roles and projects to limit access to only the necessary resources.

## Step 3: Add OpenStack Provider to Terraform Configuration

Extend the Terraform configuration to include the OpenStack provider and deploy a similar web server.

### Example OpenStack Terraform Code

```hcl
provider "openstack" {
  auth_url    = var.os_auth_url
  tenant_name = var.os_project_name
  user_name   = var.os_username
  password    = var.os_password
}

resource "openstack_compute_instance_v2" "webserver" {
  name            = "OpenStack-WebServer"
  image_name      = var.os_image
  flavor_name     = var.os_flavor
  key_pair        = var.os_key_name
  security_groups = ["default"]

  network {
    name = var.os_network
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install apache2 -y",
      "sudo systemctl start apache2",
      "sudo systemctl enable apache2"
    ]
  }
}

variable "os_auth_url" {}
variable "os_project_name" {}
variable "os_username" {}
variable "os_password" {}
variable "os_image" {}
variable "os_flavor" {}
variable "os_key_name" {}
variable "os_network" {}
```

### Add Variables

Update the `variables.tf` file with the following contents:

```hcl
variable "os_auth_url" {
  description = "The OpenStack authentication URL"
  type        = string
}

variable "os_project_name" {
  description = "The OpenStack project/tenant name"
  type        = string
}

variable "os_username" {
  description = "The OpenStack username"
  type        = string
}

variable "os_password" {
  description = "The OpenStack password"
  type        = string
  sensitive   = true
}

variable "os_image" {
  description = "The name of the OpenStack image to use"
  type        = string
}

variable "os_flavor" {
  description = "The name of the OpenStack flavor to use"
  type        = string
}

variable "os_key_name" {
  description = "The OpenStack SSH key pair name"
  type        = string
}

variable "os_network" {
  description = "The OpenStack network name"
  type        = string
}
```

### Update `terraform.tfvars`

Add the following values to `terraform.tfvars`:

```hcl
os_image      = "Ubuntu 20.04"
os_flavor     = "m1.small"
os_key_name   = "my-key"
os_network    = "public"
```

## Step 4: Deploy Resources Using Terraform

### Initialize Terraform

```bash
terraform init
```

### Validate the Configuration

```bash
terraform validate
```

### Plan the Infrastructure Deployment

```bash
terraform plan
```

### Apply the Deployment

```bash
terraform apply
```

> Confirm when prompted by typing `yes`.

## Step 5: Verify Deployments

### Confirm the EC2 Instance is Running in AWS

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=WebServer"
```

### Check the OpenStack VM

```bash
openstack server list
```

### Verify the Web Server

Visit the public IP of the AWS EC2 instance and OpenStack VM in a browser to confirm Apache is running.

Example: `http://<public-ip>`

## Step 6: Clean Up Resources

> **Security Note:** Avoid leaving unnecessary resources running, which can incur costs and pose security risks.

### Destroy the Terraform-managed Infrastructure

```bash
terraform destroy
```

> Confirm by typing `yes`.

## Challenge:  add Azure provided and deploy Azure VM.


