# **Lab: Deploy and Automate Flask App with Jenkins and Amazon EC2**

This lab demonstrates how to deploy a Flask application on an Amazon Linux EC2 instance and automate its deployment using Jenkins pipelines. 
It reviews the pros and cons of the approach and provides steps to improve security.

---

## **Step 1: Deploy Amazon Linux EC2 and Set Up the Environment**

### **Instructions:**

1. **Launch an Amazon Linux EC2 instance.**

2. **SSH into the instance:**
   ```bash
   ssh -i your-key.pem ec2-user@your-instance-ip
   ```

3. **Update the system and install required packages:**
   ```bash
   sudo yum update -y
   sudo yum install -y python3 python3-pip
   ```

4. **Create a directory for the application:**
   ```bash
   mkdir -p /home/ec2-user/app
   ```

5. **Set up a virtual environment:**
   ```bash
   cd /home/ec2-user/app
   python3 -m venv venv
   ```

6. **Create a service file for the application:**
   ```bash
   sudo nano /etc/systemd/system/flaskapp.service
   ```
   - Add the following content:
     ```ini
     [Unit]
     Description=Flask App
     After=network.target

     [Service]
     User=ec2-user
     WorkingDirectory=/home/ec2-user/app
     ExecStart=/home/ec2-user/app/venv/bin/python /home/ec2-user/app/app.py
     Restart=always

     [Install]
     WantedBy=multi-user.target
     ```

---

## **Step 2: Enable and Start the Service**

1. **Enable the service:**
   ```bash
   sudo systemctl enable flaskapp.service
   ```

2. **Reload the systemd configuration:**
   ```bash
   sudo systemctl daemon-reload
   ```

3. **Start the service:**
   ```bash
   sudo systemctl start flaskapp.service
   ```

---

## **Step 3: Create Jenkinsfile**

### **Content of Jenkinsfile:**

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    ssh ec2-user@your-instance-ip "
                        cd /home/ec2-user/app
                        source venv/bin/activate
                        pip install -r requirements.txt
                    "
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    scp -r * ec2-user@your-instance-ip:/home/ec2-user/app/
                    ssh ec2-user@your-instance-ip "
                        sudo systemctl restart flaskapp.service
                    "
                '''
            }
        }
    }
}
```

### **Explanation:**

- **Checkout:** Retrieves the code from the repository.
- **Install Dependencies:** Installs required Python packages in the virtual environment.
- **Deploy:** Copies the application files to the EC2 instance and restarts the service.

---

## **Step 4: Create a New Pipeline in Jenkins**

1. **Open Jenkins and click "New Item".**

2. **Enter a name for your pipeline and select "Pipeline".**

3. **In the Pipeline section:**
   - Choose "Pipeline script from SCM".
   - Select your SCM (e.g., Git) and enter your repository URL.
   - Specify the branch to build (e.g., `*/main`).

4. **Save the pipeline configuration.**

---

## **Step 5: Run the Pipeline Manually**

1. **Navigate to your pipeline in Jenkins.**

2. **Click "Build Now" to manually trigger the pipeline.**

---

## **Step 6: Run the Pipeline Upon Code Changes**

1. **In your GitHub repository:**
   - Go to **Settings > Webhooks**.
   - Add a new webhook with the Payload URL:
     ```
     http://your-jenkins-url/github-webhook/
     ```
   - Choose "Just the push event" for the trigger.

2. **In Jenkins pipeline configuration:**
   - Enable "GitHub hook trigger for GITScm polling" under **Build Triggers**.

---

## **Step 7: Security Pros and Cons**

### **Pros:**

1. Automated deployment reduces human error.
2. Version control integration ensures traceability.
3. Virtual environment isolates dependencies.

### **Cons:**

1. Storing credentials in Jenkins poses a security risk.
2. Direct SSH access to the EC2 instance from Jenkins is a potential vulnerability.
3. The EC2 instance must be accessible from Jenkins, potentially exposing it to the internet.

### **Security Improvements:**

1. Use AWS IAM roles for EC2 instead of hardcoded credentials.
2. Implement a bastion host or VPN for accessing the EC2 instance.
3. Use secrets management tools like AWS Secrets Manager or HashiCorp Vault.
4. Regularly update and patch all systems, including Jenkins and the EC2 instance.

---

This lab equips students with practical experience in deploying and automating applications using Jenkins and EC2 while highlighting security best practices.

