## Hands-on Lab: Deploying a Web Server with Terraform

### **Step 1: Add and Commit `main.tf` to Git**

Create a new directory for your project and initialize Git:

```bash
mkdir terraform-webserver-lab
cd terraform-webserver-lab
git init
```

Create the `main.tf` file and add the EC2 resource:

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0" # Amazon Linux 2 AMI
  instance_type = "t2.micro"

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  tags = {
    Name = "terraform-example"
  }
}
```

Add and commit the file to Git:

```bash
git add main.tf
git commit -m "Add main.tf with EC2 resource"
```

### **Step 2: Add `.gitignore` to Ignore Terraform State Files**

Create a `.gitignore` file:

```bash
touch .gitignore
```

Add the following contents to ignore Terraform state files:

```plaintext
*.tfstate
*.tfstate.backup
.terraform/
terraform.tfvars
```

Add and commit the `.gitignore` file:

```bash
git add .gitignore
git commit -m "Add .gitignore to ignore Terraform state files"
```

### **Step 4: Add Security Group Resource to Allow Ingress on Port 8080**

Update `main.tf` to include a security group resource:

```hcl
resource "aws_security_group" "web_sg" {
  name        = "webserver_sg"
  description = "Allow HTTP traffic on port 8080"

  ingress {
    description = "HTTP traffic"
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    description = "Allow all outbound traffic"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### **Step 5: Add the Security Group to the EC2 Instance**

Update the EC2 resource in `main.tf` to attach the security group:

```hcl
resource "aws_instance" "example" {
  ami                    = "ami-0c55b159cbfafe1f0"
  instance_type          = "t2.micro"
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  user_data = <<-EOF
              #!/bin/bash
              echo "Hello, World" > index.html
              nohup busybox httpd -f -p 8080 &
              EOF

  tags = {
    Name = "terraform-example"
  }
}
```

### **Step 6: Run `terraform graph` to See Dependencies**

Initialize Terraform:

```bash
terraform init
```

Run `terraform graph` to generate the dependency graph:

```bash
terraform graph > graph.dot
```

Use [Graphviz Online](https://dreampuf.github.io/GraphvizOnline/) to visualize the graph:

- Copy the contents of `graph.dot` into the online tool.
- Observe the dependencies.

### **Step 7: Plan and Deploy Infrastructure**

Run `terraform plan` to review changes:

```bash
terraform plan
```

Apply the changes:

```bash
terraform apply
```

Confirm with `yes`.

### **Step 8: Test the Web Server**

Find the public IP of the instance:

```bash
terraform output
```

Test the web server using `curl` from CodeSpaces:

```bash
curl http://<public-ip>:8080
```

Replace `<public-ip>` with the actual public IP of the EC2 instance.

Output: You should see `Hello, World`.

### **Step 9: Update `user_data` to Use Port Variable**

Update `main.tf` to use a variable for the port in `user_data`:

```hcl
user_data = <<-EOF
            #!/bin/bash
            echo "Hello, World" > index.html
            nohup busybox httpd -f -p ${var.server_port} &
            EOF
```

Update the security group ingress rule to use the variable `server_port`:

```hcl
ingress {
  description = "HTTP traffic"
  from_port   = var.server_port
  to_port     = var.server_port
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}
```

### **Step 10: Add `variables.tf` and Use Environment Variables**

Create a `variables.tf` file:

```bash
touch variables.tf
```

Define a variable for the port number:

```hcl
variable "server_port" {
  description = "The port the web server will use"
  default     = 8080
}
```

Override the port using an environment variable:

```bash
export TF_VAR_server_port=9090
```

Plan and apply the changes:

```bash
terraform plan
terraform apply
```

### **Step 11: Verify the Updated Port 9090**

Check the updated port by curling the web server:

```bash
curl http://<public-ip>:9090
```

Replace `<public-ip>` with the public IP of the EC2 instance.

Output: You should see `Hello, World`.

### **Step 12: Add Outputs to Retrieve Public IP**

Create an `output.tf` file:

```bash
touch output.tf
```

Add an output to display the EC2 public IP:

```hcl
output "public_ip" {
  description = "The public IP of the web server"
  value       = aws_instance.example.public_ip
}
```

Plan and deploy the changes:

```bash
terraform plan
terraform apply
```

Retrieve the output:

```bash
terraform output
```

### **Step 13: Destroy Infrastructure**

Destroy the deployed infrastructure:

```bash
terraform destroy
```

Confirm with `yes`.

### **Step 14: Commit Code to Git**

Add all changes:

```bash
git add .
```

Commit the code:

```bash
git commit -m "Add dynamic port, outputs, and updated user_data"
git push origin main
```

