# Step-by-Step Lab Instructions

## Step 1: Create an Amazon EC2 Instance in a Public Subnet and Store SSH Key in SSM Parameter Store

### Instructions:

#### Context:
In this step, you will set up an EC2 instance that will serve as your Jenkins server. This instance needs to be accessible via SSH and HTTP, and the SSH key will be securely stored in AWS Systems Manager Parameter Store for easier access.

#### Expected Outcome:
By the end of this step, you should have a running EC2 instance, configured correctly, with the SSH key stored securely in Parameter Store.

1. **Log in to the AWS Management Console**.

2. **Launch an EC2 Instance**:
   - Navigate to the EC2 Dashboard and click **Launch Instance**.
   - Enter a name for the instance (e.g., Jenkins-Server).
   - Select the Amazon Linux 2 AMI or Ubuntu (latest LTS version).
   - Choose the **t2.micro** instance type (free-tier eligible).

   ![Screenshot: Launch EC2 Instance](placeholder_ec2_instance_launch.png)

3. **Configure Network Settings**:
   - Ensure the instance is launched in the default VPC.
   - Select a public subnet to ensure internet access.
   - Enable **Auto-assign Public IP**.

   ![Screenshot: Network Settings](placeholder_network_settings.png)

4. **Create a New Key Pair**:
   - In the Key Pair section, choose **Create New Key Pair**.
   - Download the `.pem` file and save it securely.

   ![Screenshot: Create Key Pair](placeholder_create_key_pair.png)

5. **Store the Key in SSM Parameter Store**:
   - Navigate to **AWS Systems Manager > Parameter Store**.
   - Click **Create Parameter**.
   - Enter the following details:
     - **Name**: `/ec2/jenkins-key`
     - **Type**: SecureString
     - **Value**: Copy the contents of the `.pem` file and paste it here.
   - Save the parameter.

   ![Screenshot: Store SSH Key in Parameter Store](placeholder_store_ssh_key.png)

6. **Security Group Configuration**:
   - Ensure the security group allows **SSH (port 22)** and **HTTP (port 8080)** traffic.

   ![Screenshot: Security Group Settings](placeholder_security_group_settings.png)

7. **Launch the Instance**:
   - Click **Launch Instance** and wait for the instance to initialize.

## Step 2: Set Up Single-Node Jenkins on the Amazon EC2 Instance

### Instructions:

#### Context:
In this step, you will install Jenkins on the EC2 instance that you set up in Step 1. This involves updating the system, installing necessary dependencies, and configuring Jenkins to be accessible via the web.

#### Expected Outcome:
By the end of this step, Jenkins should be installed and running on your EC2 instance, accessible through the public IP address on port 8080.

1. **Connect to the EC2 Instance**:
   - Use AWS CLI or terminal to connect to the instance. Run:
     ```bash
     ssh -i /path/to/jenkins-key.pem ec2-user@<public-ip>
     ```
   - Alternatively, retrieve the key from SSM Parameter Store:
     ```bash
     aws ssm get-parameter --name /ec2/jenkins-key --with-decryption --query Parameter.Value --output text > jenkins-key.pem
     chmod 400 jenkins-key.pem
     ssh -i jenkins-key.pem ec2-user@<public-ip>
     ```

2. **Install Jenkins**:
   - Update the instance:
     ```bash
     sudo yum update -y
     sudo yum install -y java-11-openjdk
     ```
   - Add Jenkins repository and import its key:
     ```bash
     sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
     sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
     ```
   - Install Jenkins:
     ```bash
     sudo yum install -y jenkins
     sudo systemctl enable jenkins
     sudo systemctl start jenkins
     ```

3. **Verify Jenkins Is Running**:
   - Check running components using:
     ```bash
     ps aux | grep jenkins
     netstat -tuln | grep 8080
     ```
   - Other methods to list Jenkins components:
     - Use `systemctl status jenkins`.
     - Check logs in `/var/log/jenkins/jenkins.log`.

4. **Access Jenkins in Browser**:
   - Open your browser and navigate to `http://<public-ip>:8080`.
   - Retrieve the initial admin password:
     ```bash
     sudo cat /var/lib/jenkins/secrets/initialAdminPassword
     ```
   - Follow the setup wizard to configure Jenkins.

## Step 3: Configure a New Jenkins Job Triggered by a Webhook

### Instructions:

#### Context:
In this step, you will create a Jenkins job that will be triggered automatically whenever there is a push event in your GitHub repository. This setup is crucial for implementing CI/CD practices.

#### Expected Outcome:
By the end of this step, Jenkins should be configured to automatically start a pipeline job whenever code is pushed to the GitHub repository.

1. **Create a GitHub Repository**:
   - Create a new public repository named `week5-lab2`.

2. **Set Up a Webhook in GitHub**:
   - Go to the **Settings** of the repository > **Webhooks** > **Add Webhook**.
   - Enter the following details:
     - **Payload URL**: `http://<jenkins-public-ip>:8080/github-webhook/`
     - **Content type**: `application/json`.
     - **Select events**: Just the **push** event.

3. **Create a Jenkins Job**:
   - In Jenkins, click **New Item** > **Freestyle Project** > Name it `week5-lab2`.
   - Configure the following:
     - Under **Source Code Management**, select **Git** and enter the repository URL.
     - Under **Build Triggers**, check **GitHub hook trigger for GITScm polling**.
     - Under **Build**, select **Execute Shell** and enter:
       ```bash
       echo "Hello Cloud Security students!"
       ```
   - Save the job.

## Step 4: Start the Pipeline Manually

#### Context:
In this step, you will manually start the Jenkins pipeline to verify that it is configured correctly and that it produces the expected output.

#### Expected Outcome:
By the end of this step, you should be able to see the output "Hello Cloud Security students!" in the Jenkins console log.

1. From the Jenkins dashboard, go to the `week5-lab2` job.
2. Click **Build Now**.
3. **Verify the output**:
   - Click the build number (e.g., `#1`) > **Console Output**.
   - Ensure "Hello Cloud Security students!" is printed.

## Step 5: Commit Changes to the Main Branch and Verify Trigger

#### Context:
In this step, you will test the GitHub webhook by making changes to the repository and observing Jenkins automatically triggering a new build.

#### Expected Outcome:
By the end of this step, any commit made to the main branch should automatically trigger a Jenkins pipeline run.

1. **Make a commit to the repository**:
   - Update the `README.md` file or add a new file to the repository.
2. **Verify Jenkins automatically triggers a new build**:
   - Check the Jenkins dashboard for the new build.
   - Ensure the build outputs "Hello Cloud Security students!".

## Step 6: Configure a PR Check for the Pipeline

### Instructions:

#### Context:
In this step, you will configure Jenkins to automatically perform checks for pull requests (PRs) before they can be merged into the main branch. This helps ensure code quality and stability.

#### Expected Outcome:
By the end of this step, Jenkins should automatically run the pipeline for every new pull request, ensuring that code passes checks before merging.


1. **Install GitHub Plugin**:
   - In Jenkins, go to **Manage Jenkins > Plugin Manager**.
   - Install the **GitHub Integration Plugin**.

2. **Configure PR Check**:
   - Go to the `week5-lab2` job > **Configure**.
   - Under **Build Triggers**, enable:
     - **GitHub Pull Request Builder**.

3. **Update Webhook for PR Events**:
   - Go to the repository’s **Settings > Webhooks**.
   - Ensure **Pull Request** events are selected in addition to push.

4. **Test the PR Check**:
   - Create a new branch and make a pull request to the main branch.
   - Ensure the pipeline runs and completes successfully.

### Expected Outcome:
- Jenkins runs a pipeline on every push or PR event.
- The pipeline prints "Hello Cloud Security students!" for successful builds.
- PR checks are automatically performed for code quality verification.

## Step 7: Configure Branch Protection Rules to Trigger the Webhook

### Instructions:

#### Context:
In this step, you will set up branch protection rules to enforce Jenkins status checks before merging any pull requests into the main branch. This ensures that only thoroughly tested and approved code makes it into the main branch.

#### Expected Outcome:
By the end of this step, Jenkins status checks will be required for all pull requests, and unapproved PRs will be blocked from merging until they pass all necessary checks.

1. **Go to the GitHub Repository Settings**:
   - Open your `week5-lab2` repository on GitHub.
   - Click on the **Settings** tab.

2. **Navigate to Branch Protection Rules**:
   - Scroll down to the **Code and Automation** section on the left menu.
   - Click on **Branches** and then select **Add Rule** under the **Branch Protection Rules** section.

3. **Set Up the Branch Protection Rule for the main Branch**:
   - Under **Branch name pattern**, enter `main` to apply the rule to the main branch.

4. **Enable Status Checks for Webhooks**:
   - Scroll down to the **Require status checks to pass before merging** section.
   - Enable the checkbox for **Require status checks to pass before merging**.
   - Select the Jenkins webhook status check (this will appear automatically if the webhook and Jenkins are integrated correctly).

5. **Enable Additional Protections (Optional)**:
   - Enable **Require pull request reviews before merging** to ensure that pull requests are reviewed by at least one team member before merging.
   - Enable **Include administrators** if you want branch protection rules to apply to administrators as well.

6. **Save the Rule**:
   - Scroll down and click **Save Changes**.

### Testing the Branch Protection Rules

1. **Push Changes to the main Branch**:
   - Push any code changes directly to the main branch.
   - Verify that the Jenkins pipeline is triggered by the webhook.

2. **Verify Status Checks**:
   - Open the **Pull Requests** tab in your repository.
   - Create a new pull request against the main branch.
   - Ensure that the Jenkins pipeline runs and displays a "Pending" or "Completed" status check under the pull request.

3. **Check Merging Behavior**:
   - Attempt to merge the pull request without passing the Jenkins status check.
   - GitHub should block the merge until the pipeline completes successfully and the status check is marked as passed.

### Expected Outcome:
- Branch protection rules enforce status checks for Jenkins pipeline executions before merging into the main branch.
- The Jenkins pipeline is triggered through the webhook for each commit and pull request.
- Pull requests cannot be merged unless the pipeline passes, ensuring stable and tested code in the main branch.

