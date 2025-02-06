# DevSecOps Cloud Security Lab Guide

## Lab Overview

1. Test the Flask application locally using **GitHub Codespaces**.
2. Create a virtual machine (VM) on **AWS** using the AWS Management Console.
3. Connect to the VM using **EC2 Instance Connect**.
4. Clone the Flask application repository.
5. Set up a Python virtual environment and install dependencies.
6. Start the application and test using `curl`.
7. Automate the process using a Bash script.

---

## **Part 1: Testing the Application Locally**

### Step 1: Open GitHub Codespaces

1. In your Labs repository, click the **Code** button and select **Codespaces**.
2. Create a new Codespace if you don’t already have one.

### Step 2: Clone the repository with Flask Application code

1. Once inside Codespaces, run the following command to clone the repository:
   ```bash
   git clone https://github.com/Sheridan-College-FAST-CloudSecurity/devops-week1-lab1.git
   cd devops-week1-lab1
   ```

### Step 3: Create a Virtual Environment named "venv"

1. Create a Python virtual environment:
   ```bash
   python3 -m venv venv
   ```
2. Activate the virtual environment:
   ```bash
   source venv/bin/activate
   ```

### Step 4: Install Dependencies

1. Install the required dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Step 5: Start the Flask Application

1. Start the Flask application:
   ```bash
   cd hello
   flask run
   ```
2. Once the application runs, note the local URL ( `http://127.0.0.1:5000`).
3. GitHub CodeSpaces will prompt you to open the application in the browser. Open the application and confirm you can see "Hello, World" message


---

## **Part 2: Deploying the Application on AWS EC2**

### Step 1: Create a VM on AWS

1. Log in to the **AWS Management Console**.
2. Navigate to the **EC2 Dashboard**.
3. Click **Launch Instance** and configure the following:
   - **AMI**: Select the default AMI
   - **Instance Type**: Choose an appropriate instance type (e.g., t2.micro).
   - **Key Pair**: Select an existing key pair.
   - **Security Group**: Allow inbound traffic on port 22 (SSH) and 5000 (Flask application).
4. Launch the instance and wait for it to initialize.

### Step 2: Connect to the VM

1. In the EC2 dashboard, select your instance and click **Connect**.
2. Choose **EC2 Instance Connect** and click **Connect**.

### Step 3: Clone the Repository

1. Once connected to the instance, run the following commands:
   ```bash
   sudo yum update -y
   sudo yum install git -y  
   git clone https://github.com/Sheridan-College-FAST-CloudSecurity/devops-week1-lab1.git
   cd devops-week1-lab1
   ```

### Step 4: Set Up a Virtual Environment

1. Install Python 3 and `pip` (if not already installed):
   ```bash
   sudo yum install python3-pip -y   # For Amazon Linux 2

   ```
2. Create and activate a virtual environment:
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

### Step 5: Install Dependencies

1. Install the application dependencies:
   ```bash
   pip install -r requirements.txt
   ```

### Step 6: Start the Flask Application

1. Start the application:
   ```bash
   cd hello
   flask run
   ```
2. Note the application using `curl localhost`:
   ```bash
   curl localhost:5000
   ```
3. Note the public IP address of the instance and test the application in the browser:
   ```bash
   # Stop the running application with CTRL+C
   # Modify the Flask run command to listen on all available IP addresses by using 0.0.0.0
   flask run --host=0.0.0.0
   curl localhost:5000
   curl http://<public-ip>:5000
   ```
---

## **Part 3: Wrapping Up in a Bash Script**

### Step 1: Create the Bash Script

1. Create a new file called `deploy_flask.sh`:
   ```bash
   nano deploy_flask.sh
   ```
2. Add the following content to the script 
   ```bash
   #!/bin/bash

   # Update and install necessary packages
   sudo yum update -y  
   sudo yum install python3-pip -y  

   # Clone the repository
   git clone https://github.com/Sheridan-College-FAST-CloudSecurity/devops-week1-lab1.git
   cd devops-week1-lab1

   # Set up virtual environment
   python3 -m venv venv
   source venv/bin/activate

   # Install dependencies
   pip install -r requirements.txt

   # Start the application
   python app.py
   ```
3. Save and close the file.

### Step 2: Make the Script Executable

1. Run the following command to make the script executable:
   ```bash
   chmod +x deploy_flask.sh
   ```

### Step 3: Execute the Script

1. Execute the script to deploy the application:
   ```bash
   ./deploy_flask.sh
   ```
### Step 4: It does not work! ***What is the problem?***:

1. Update the script
   ```bash
   #!/bin/bash

   # Update and install necessary packages
   sudo yum update -y  
   sudo yum install python3-pip -y  

   # Clone the repository
   git clone https://github.com/Sheridan-College-FAST-CloudSecurity/devops-week1-lab1.git
   cd devops-week1-lab1

   # Set up virtual environment
   python3 -m venv venv
   source venv/bin/activate

   # Install dependencies
   pip install -r requirements.txt

   # UPDATE the start command 
   cd hello
   flask run  --host=0.0.0.0
   ```
---
