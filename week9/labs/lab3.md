## Extended Lab: Using Existing Key Pair, Subnet, and AMI with Terraform Data Sources 🛠️

### Objective

This lab extension demonstrates how to use Terraform data sources to fetch an existing key pair, a default subnet, and the latest Amazon Linux 2 AMI dynamically. The EC2 instance will use these data sources for configuration, showcasing Terraform's ability to query and integrate existing AWS resources.

---

### Step-by-Step Instructions

#### Step 1: Update the Provider Configuration

Ensure the AWS provider is configured for the us-east-1 region. Add this to `main.tf` if not already present:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

#### Step 2: Add Data Sources for Existing Key Pair, Subnet, and AMI

Add the following data sources to dynamically fetch the required AWS resources:

```hcl
# Fetch the existing key pair named 'voclabs'
data "aws_key_pair" "existing_key" {
  key_name = "voclabs"
}

# Fetch the default subnet for the availability zone 'us-east-1a'
data "aws_subnet" "default_subnet" {
  filter {
    name   = "default-for-az"
    values = ["true"]
  }

  filter {
    name   = "availability-zone"
    values = ["us-east-1a"]
  }
}

# Fetch the latest Amazon Linux 2 AMI
data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*x86_64-gp2"]
  }
}
```

#### Step 3: Verify the Key Pair

Ensure the key pair `voclabs` exists in the `us-east-1` region. Run the following AWS CLI command to confirm:

```bash
aws ec2 describe-key-pairs --query 'KeyPairs[*].KeyName' --region us-east-1
```

Look for `voclabs` in the output. If it doesn’t exist, create it using the AWS CLI or Management Console.

#### Step 4: Modify the EC2 Instance Resource

Update the EC2 instance configuration to use the fetched data sources for the AMI, subnet, and key pair:

```hcl
resource "aws_instance" "web_server" {
  ami           = data.aws_ami.amazon_linux.id
  instance_type = "t2.micro"
  subnet_id     = data.aws_subnet.default_subnet.id
  security_groups = [
    aws_security_group.web_sg.name
  ]

  # Use the existing key pair fetched from the data source
  key_name = data.aws_key_pair.existing_key.key_name

  user_data = <<-EOF
              #!/bin/bash
              yum update -y
              yum install -y httpd
              systemctl start httpd
              systemctl enable httpd
              echo "Hello, Terraform with dynamic data sources!" > /var/www/html/index.html
            EOF

  tags = {
    Name = "Terraform-Web-Server"
  }
}
```

#### Step 5: Update the Output Variables

Update the output variable to include the public IP of the EC2 instance for easy SSH access:

```hcl
output "instance_public_ip" {
  description = "Public IP of the web server"
  value       = aws_instance.web_server.public_ip
}
```

---

### Execution Steps

#### Step 1: Initialize Terraform

Run the following command to initialize Terraform:

```bash
terraform init
```

#### Step 2: Plan the Changes

Preview the changes Terraform will make:

```bash
terraform plan
```

Ensure that:

- The `ami` is fetched dynamically from the `aws_ami` data source.
- The `subnet_id` is fetched dynamically from the `aws_subnet` data source.
- The `key_name` is fetched dynamically from the `aws_key_pair` data source.

#### Step 3: Apply the Configuration

Deploy the resources:

```bash
terraform apply
```

Enter `yes` when prompted to confirm.

#### Step 4: Access the EC2 Instance

After the resources are deployed:

- Retrieve the instance's public IP from the output.
- SSH into the instance using the private key associated with the `voclabs` key pair:

```bash
ssh -i /path/to/voclabs.pem ec2-user@<public_ip>
```

---

### Testing the Web Server

Open a web browser and navigate to `http://<public_ip>`.

Verify that the page displays:

```plaintext
Hello, Terraform with dynamic data sources!
```

---

### Cleanup

To avoid incurring charges, destroy the resources after completing the lab:

```bash
terraform destroy
```

Enter `yes` when prompted.

