## **Lab Pre-requisites**

1. **GitHub Account**: Ensure you have access to GitHub CodeSpaces.
2. **GitHub Repository**: Create a new repository or use an existing one for this lab.
3. **AWS Account**: Set up your IAM credentials with an optional session token.

---

## **Lab Steps**

---

### **Step 1: Set Up Terraform in GitHub CodeSpaces**

1. Open your GitHub repository.
2. Start a new CodeSpace:
   - Click **Code > CodeSpaces > Create CodeSpace on main**.
3. Install Terraform in CodeSpaces:

```bash
sudo apt update
sudo apt install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform
```

Verify the installation:

```bash
terraform version
```

### **Step 2: Configure AWS Credentials Using Environment Variables**

Add your AWS credentials and session token as environment variables:

```bash
export AWS_ACCESS_KEY_ID=<your-access-key-id>
export AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
export AWS_SESSION_TOKEN=<your-session-token>
export AWS_REGION=us-east-1
```

Replace `<your-access-key-id>`, `<your-secret-access-key>`, and `<your-session-token>` with your IAM user's credentials and session token.

Note: The session token is required if your AWS credentials were generated with temporary access (e.g., from AWS SSO or MFA).

Confirm the environment variables are set:

```bash
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN
echo $AWS_REGION
```

### **Step 3: Set Up Terraform Project**

Create a new Terraform configuration file:

```bash
mkdir terraform-aws-lab
cd terraform-aws-lab
touch main.tf
```

Write the following configuration into `main.tf`:

```hcl
# Provider Configuration
provider "aws" {
  region = var.aws_region
}

# Define variables
variable "aws_region" {
  default = "us-east-1"
}

# Resource Configuration: EC2 Instance
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformLabInstance"
  }
}
```

### **Step 4: Initialize Terraform in CodeSpaces**

Initialize Terraform to download AWS provider plugins:

```bash
terraform init
```

Confirm successful initialization.

### **Step 5: Deploy EC2 Instance with Tags**

Validate the configuration:

```bash
terraform plan
```

Verify the output includes the `Name` tag for the EC2 instance.

Deploy the instance:

```bash
terraform apply
```

Confirm with `yes`.

Verify the deployment:

- Log in to the AWS Console.
- Navigate to EC2 > Instances and confirm the instance is created with the tag `Name: TerraformLabInstance`.

### **Step 6: Examine the Terraform State File**

Locate the `terraform.tfstate` file in your project directory:

```bash
ls -l
```

Output: Ensure `terraform.tfstate` exists.

View the state file contents using a text editor or `cat`:

```bash
cat terraform.tfstate
```

Key elements to examine:

- **Resource Details**: The `resources` section lists the managed resources, their IDs, and properties.
- **Outputs**: Any output values specified in the configuration.

### **Step 7: Clean Up Resources**

Destroy the created infrastructure to avoid unnecessary costs:

```bash
terraform destroy
```

Confirm with `yes`.

Verify the infrastructure is destroyed:

- Log in to the AWS Console.
- Check that the EC2 instance is no longer listed under EC2 > Instances.

### **Step 8: Verify Terraform State File is Destroyed**

After all resources are destroyed, Terraform removes details about the managed infrastructure from the state file.

Inspect the `terraform.tfstate` file:

```bash
cat terraform.tfstate
```

Output: The state file will still exist, but it will no longer contain resource details. Instead, it will show an empty state like this:

```json
{
  "version": 4,
  "terraform_version": "X.XX.X",
  "resources": []
}
```

Confirm that all resources are removed from the state.

If you want to completely remove the state file:

```bash
rm terraform.tfstate
```

