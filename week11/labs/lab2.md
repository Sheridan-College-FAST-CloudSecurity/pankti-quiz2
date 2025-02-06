# **Lab: Evaluating Terraform Configurations Using OPA and Conftest in Codespaces**

In this lab, students will learn how to:

1. **Set up a Codespaces environment** with Terraform and Conftest pre-installed.
2. **Write a simple Rego policy** to enforce region constraints for Terraform configurations.
3. **Test Terraform configurations** within Codespaces using Conftest.
4. **Understand the execution flow of OPA and Rego.**

---

## **1. Prerequisites**

- Access to a **GitHub Codespaces** environment.
- Basic knowledge of Terraform configuration files.
- A sample Terraform configuration file for deploying an RDS MySQL database into the default VPC (provided below).
- Codespaces pre-configured with **Terraform**, **Conftest**, and **AWS CLI**.

---

## **2. Set Up Codespaces Environment**

### Launch a New Codespaces Instance

1. Navigate to your repository in GitHub.
2. Click on the **Code** button, then select **Open with Codespaces**.
3. If no Codespaces configuration exists, GitHub will create a default environment.

Ensure the environment has Terraform and Conftest pre-installed. You can verify this with:

```bash
terraform --version
conftest --version
```

If any tools are missing, you can install them using the following commands in the Codespaces terminal:

### Install Terraform
```bash
sudo apt-get update && sudo apt-get install -y terraform
```

### Install Conftest
```bash
curl -L https://github.com/open-policy-agent/conftest/releases/download/v0.37.0/conftest_0.37.0_Linux_x86_64.tar.gz -o conftest.tar.gz
tar -xzf conftest.tar.gz
sudo mv conftest /usr/local/bin
```

---

## **3. Understanding OPA, Conftest, and Rego**

### **What is OPA (Open Policy Agent)?**
- OPA is a policy engine that decouples policy decision-making from your application logic.
- Policies are written in a declarative language called Rego.

### **What is Conftest?**
- Conftest is a CLI tool that integrates with OPA to test configuration files.
- It runs OPA policies against configuration files like Terraform, Kubernetes YAML, and more.

### **What is Rego?**
- Rego is the language used to write policies for OPA.
- It is declarative, meaning you define what should happen rather than how it happens.

---

## **4. Write a Simple OPA Policy in Rego**

### **Policy Objective**
Ensure that only `us-east-1` or `us-west-2` regions are used for deploying resources.

### **Policy Code**
Create a directory for the policy in the Codespaces terminal:
```bash
mkdir policies
cd policies
```

Create a new file named `region.rego`:
```rego
# Define a policy package for Terraform
package terraform.regions

# Default rule: deny deployment unless conditions are met
default deny = false

# Deny message if the region is not allowed
deny[msg] {
    input.resource_changes[_].change.after.region != "us-east-1"
    input.resource_changes[_].change.after.region != "us-west-2"
    msg = sprintf("Region '%s' is not allowed. Use 'us-east-1' or 'us-west-2'.", [input.resource_changes[_].change.after.region])
}
```

---

## **5. Create a Sample Terraform Configuration**

Here’s a sample Terraform configuration that deploys an RDS MySQL database in the default VPC. Save this as `main.tf` in Codespaces:

```hcl
provider "aws" {
  region = "eu-west-1"  # Change this to an invalid region to test the policy
}

resource "aws_db_instance" "example" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "db.t3.micro"
  name                 = "mydb"
  username             = "admin"
  password             = "securepassword"
  publicly_accessible  = false
}
```

---

## **6. Evaluate the Terraform Configuration in Codespaces**

### Step 1: Convert Terraform Plan to JSON

#### Initialize Terraform:
```bash
terraform init
```

#### Generate a plan and save it as a JSON file:
```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
```

### Step 2: Test the Configuration Using Conftest

#### Run Conftest to evaluate the plan against the policy:
```bash
conftest test tfplan.json
```

#### Output:
If the region is invalid (e.g., `eu-west-1`):
```plaintext
FAIL - tfplan.json - Region 'eu-west-1' is not allowed. Use 'us-east-1' or 'us-west-2'.
```

If the region is valid (e.g., `us-east-1`), no failures are reported.

---

## **7. Modify and Re-Test in Codespaces**

### Test with a Valid Configuration

#### Edit the `main.tf` file to use a valid region:
```hcl
provider "aws" {
  region = "us-east-1"
}
```

#### Re-generate the plan:
```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
```

#### Run Conftest again:
```bash
conftest test tfplan.json
```

#### Expected Output:
```plaintext
PASS - tfplan.json
```
## **8. Enforce Instance Type Constraints**
### **Policy 2: Enforce Instance Type Constraints**

#### **Objective**
Ensure that only instance types starting with `t` (e.g., `t3.micro`, `t2.small`) are used for deploying resources.

#### **Policy Code**
Create a new file named `instance_type.rego`:
```rego
# Define a policy package for Terraform
package terraform.instance_types

# Default rule: deny deployment unless conditions are met
default deny = false

# Deny message if the instance type is not allowed
deny[msg] {
    not startswith(input.resource_changes[_].change.after.instance_type, "t")
    msg = sprintf("Instance type '%s' is not allowed. Use types starting with 't'.", [input.resource_changes[_].change.after.instance_type])
}
```

---

## **9. Create a Sample Terraform Configuration**

Here’s a sample Terraform configuration that deploys an RDS MySQL database in the default VPC. Save this as `main.tf` in Codespaces:

```hcl
provider "aws" {
  region = "eu-west-1"  # Change this to an invalid region to test the policy
}

resource "aws_db_instance" "example" {
  allocated_storage    = 20
  engine               = "mysql"
  engine_version       = "8.0"
  instance_class       = "m5.large"  # Change this to a valid type like 't3.micro' to test the policy
  name                 = "mydb"
  username             = "admin"
  password             = "securepassword"
  publicly_accessible  = false
}
```

---

## **10. Evaluate the Terraform Configuration in Codespaces**

### Step 1: Convert Terraform Plan to JSON

#### Initialize Terraform:
```bash
terraform init
```

#### Generate a plan and save it as a JSON file:
```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
```

### Step 11: Test the Configuration Using Conftest

#### Run Conftest to evaluate the plan against both policies:
```bash
conftest test tfplan.json
```

#### Output:
If the region is invalid (e.g., `eu-west-1`):
```plaintext
FAIL - tfplan.json - Region 'eu-west-1' is not allowed. Use 'us-east-1' or 'us-west-2'.
```

If the instance type is invalid (e.g., `m5.large`):
```plaintext
FAIL - tfplan.json - Instance type 'm5.large' is not allowed. Use types starting with 't'.
```

If both constraints are met, no failures are reported.

---

## **12. Modify and Re-Test in Codespaces**

### Test with Valid Configurations

#### Edit the `main.tf` file to use a valid region and instance type:
```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_db_instance" "example" {
  instance_class = "t3.micro"
}
```

#### Re-generate the plan:
```bash
terraform plan -out=tfplan.binary
terraform show -json tfplan.binary > tfplan.json
```

#### Run Conftest again:
```bash
conftest test tfplan.json
```

#### Expected Output:
```plaintext
PASS - tfplan.json
```





