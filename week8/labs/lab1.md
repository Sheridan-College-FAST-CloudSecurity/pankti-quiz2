# Lab: Using Terraform in GitHub CodeSpaces to Create an Amazon EC2 Instance

This lab guides you through installing Terraform in GitHub CodeSpaces, configuring AWS credentials, and creating an Amazon EC2 instance using Terraform. 
Each step explains the commands and concepts in detail.

---

## **Step 1: Launch GitHub CodeSpaces**

1. Open your GitHub repository.
2. Click on the **Code** dropdown menu and select **Open with CodeSpaces**.
3. If no CodeSpace exists, create a new one. This launches an online development environment.

---

## **Step 2: Install Terraform in CodeSpaces**

1. Open the terminal in CodeSpaces.
2. Run the following commands to install Terraform:
   ```bash
   sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
   curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
   sudo apt-get update && sudo apt-get install terraform
   ```
3. Verify the installation:
   ```bash
   terraform --version
   ```

---

## **Step 3: Configure AWS Credentials**

1. Install the AWS CLI:
   ```bash
   sudo apt-get install awscli
   ```
2. Configure AWS credentials using environment variables:
   ```bash
   export AWS_ACCESS_KEY_ID="<your-access-key-id>"
   export AWS_SECRET_ACCESS_KEY="<your-secret-access-key>"
   export AWS_DEFAULT_REGION="us-east-1"
   export AWS_SESSION_TOKEN="<your-aws-session-token-if-required>"
   ```
   - Replace `<your-access-key-id>`, `<your-secret-access-key>`, and `<your-aws-session-token-if-required>` with the appropriate values for your temporary or permanent credentials.
   - Ensure `AWS_SESSION_TOKEN` is included if you are using temporary credentials (e.g., from AWS STS).

Test the credentials:

```bash
aws s3 ls
```

This command lists your AWS S3 buckets, confirming that your credentials are correct.

---

## **Step 4: Create the Terraform Configuration File**

1. Create a file named `main.tf`:
   ```bash
   touch main.tf
   ```
2. Open the file and add the following HCL configuration:
   ```hcl
   # Provider block
   provider "aws" {
     region = "us-east-1" # AWS region to deploy resources
   }

   # Resource block to create an EC2 instance
   resource "aws_instance" "example_instance" {
     ami           = "ami-0c55b159cbfafe1f0" # Amazon Machine Image for Ubuntu
     instance_type = "t2.micro"             # Free-tier eligible instance type

     # Tags to identify the EC2 instance
     tags = {
       Name = "MyTerraformInstance"        # Name tag for the instance
     }
   }
   ```
3. **Explanation of HCL Components**:
   - **Provider Block**:
     - **Purpose**: Specifies the cloud provider (AWS) and the region for deployment.
     - **Key Argument**: `region` defines the AWS region (e.g., `us-east-1`).
   - **Resource Block**:
     - **Purpose**: Defines the resource (an EC2 instance) and its properties.
     - **Key Arguments**:
       - `ami`: Specifies the Amazon Machine Image (e.g., Ubuntu).
       - `instance_type`: Sets the instance type (e.g., `t2.micro`).
       - `tags`: Adds metadata to identify the resource (e.g., a `Name` tag).

---

## **Step 5: Initialize and Apply Terraform**

1. **Format the Configuration File**:
   ```bash
   terraform fmt
   ```
   - Ensures consistent formatting and readability of the `main.tf` file.
2. **Initialize Terraform**:
   ```bash
   terraform init
   ```
   - Downloads necessary provider plugins and initializes the working directory.
3. **Validate the Configuration**:
   ```bash
   terraform validate
   ```
   - Checks the syntax and correctness of the configuration file.
4. **Apply the Configuration**:
   ```bash
   terraform apply
   ```
   - Terraform displays an execution plan.
   - Type `yes` to confirm and provision the resources.

---

## **Step 6: Verify the EC2 Instance**

1. Log in to the AWS Management Console.
2. Navigate to the EC2 Dashboard.
3. Confirm that the instance is running with the correct configuration and tags.

---

## **Step 7: Understand the Terraform State File**

### Explanation:
- Terraform creates a state file (`terraform.tfstate`) in the working directory.
- This file tracks the current state of your infrastructure and is crucial for:
  - **Incremental Changes**: Ensures only necessary updates are made to match the desired state.
  - **Dependency Management**: Tracks relationships between resources.

### Actions:
1. Open the state file to inspect its contents:
   ```bash
   cat terraform.tfstate
   ```
2. **Key Contents**:
   - Resource attributes (e.g., instance ID, tags).
   - Provider details (e.g., AWS region).

### Note:
- Never share your state file publicly as it may contain sensitive information.
- Use remote state backends (e.g., AWS S3) for collaboration.

### Customizing Instance Properties

To provide more flexibility and depth, you can customize the EC2 instance properties in the `main.tf` file. Below are a few examples:

1. **Changing Instance Type:**
   - Update the `instance_type` to meet your workload requirements:
     ```hcl
     instance_type = "t2.large"
     ```
   - Larger instance types provide more memory and CPUs.

2. **Adding Additional Tags:**
   - Use tags to add metadata to resources for better organization:
     ```hcl
     tags = {
       Name        = "MyTerraformInstance"
       Environment = "Production"
       Owner       = "DevOps Team"
     }
     ```

3. **Enabling Detailed Monitoring:**
   - Add the `monitoring` argument to enable CloudWatch monitoring:
     ```hcl
     monitoring = true
     ```

4. **Assigning an IAM Role:**
   - Attach an IAM role to the instance for granting permissions:
     ```hcl
     iam_instance_profile = "my-instance-profile"
     ```

Make these changes in the `main.tf` file before running `terraform apply` to reflect the updated configurations.

---

## **Step 8: Clean Up**

1. Destroy the EC2 instance and remove associated resources:
   ```bash
   terraform destroy
   ```
2. Type `yes` to confirm.

---

## **Lab Summary**

- **Key Achievements**:
  - Installed Terraform in GitHub CodeSpaces.
  - Configured AWS CLI for authentication.
  - Created an EC2 instance using Terraform.
  - Focused on understanding provider and resource arguments:
    - **Provider**: Configured AWS with a region.
    - **Resource**: Defined EC2 instance properties such as AMI, instance type, and tags.
  - Understood the importance of Terraform state for managing infrastructure.

