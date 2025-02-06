# **Lab: Integrating SAST and SCA in Jenkins for Python Code**

This lab guides you through configuring Jenkins to scan Python code for vulnerabilities using Static Application Security Testing (SAST) and Software Composition Analysis (SCA) tools. Students will set up Jenkins pipelines, automate scans using GitHub webhooks, and publish results to GitHub pull requests. The lab will use **Bandit** for SAST and **OWASP Dependency-Check** for SCA.

---

## **Potential Improvement**

To streamline the lab experience, consider providing **pre-configured starter templates**:

1. **Starter Jenkinsfile:**  
   - Provide a template with placeholders for repository URLs, credentials, and tools, allowing students to focus on customizing specific pipeline steps.

2. **Sample GitHub Repository:**  
   - Provide a private repository with a Python application, `requirements.txt`, and example vulnerabilities to scan. This helps students focus on the process rather than creating or finding a test repository.

These improvements save setup time, allowing students to concentrate on key learning objectives.

---

## **Step 1: Generate a GitHub Personal Access Token (PAT)**

To enable Jenkins to authenticate with a private GitHub repository, you’ll need a **Personal Access Token (PAT)**.

### **Instructions:**

1. **Log in to GitHub:**  
   Open [GitHub](https://github.com/) and log in to your account.

2. **Navigate to Developer Settings:**  
   - Click your profile picture (top-right corner) and go to **Settings**.
   - Scroll down to **Developer settings** and click it.
   - Select **Personal access tokens > Tokens (classic)**.

3. **Generate a New Token:**  
   - Click **Generate new token (classic)**.
   - Add a note (e.g., `Jenkins Access`) and set an expiration (e.g., 30 days).
   - Under **Scopes**, select:
     - **`repo`**: Full access to private repositories.
     - **`read:org`**: (Optional) Access for organization repositories.

4. **Copy the Token:**  
   - Click **Generate Token** and copy it immediately (you won’t see it again).
   - Store it securely; you’ll use it in Jenkins configuration.

![Screenshot of GitHub Developer Settings](placeholder_github_developer_settings.png)

---

## **Step 2: Configure Jenkins Credentials**

### **Instructions:**

1. **Access Jenkins:**  
   Open Jenkins in your browser (e.g., `http://<your-jenkins-server>:8080`) and log in.

2. **Add GitHub Credentials:**  
   - Go to **Manage Jenkins > Credentials > System > Global credentials (unrestricted)**.
   - Click **Add Credentials** and configure:
     - **Kind:** Username with password or Secret text.
     - **Scope:** Global.
     - **Username:** Your GitHub username.
     - **Password/Secret Text:** Paste your PAT.
     - **ID:** Use a recognizable ID like `github-credentials`.

3. **Save Credentials:**  
   Confirm the credentials are saved and accessible for use in pipelines.

![Screenshot of Jenkins Credentials Section](placeholder_jenkins_credentials_section.png)

---

## **Step 3: Creating and Testing the Jenkins Pipeline**

### **Step 3.1: Jenkinsfile to Clone the Repository**

#### **Pipeline Code:**

```groovy
pipeline {
    agent any
    environment {
        GIT_CREDENTIALS = credentials('github-credentials') // Replace with your credentials ID
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: "${GIT_CREDENTIALS}",
                    url: 'https://github.com/<your-repo>.git' // Replace with your private repo URL
            }
        }
    }
}
```

### **Instructions:**

1. **Create a new Pipeline job in Jenkins and paste the above Jenkinsfile.**
2. **Run the pipeline by clicking Build Now.**
3. **Verify the Console Output to confirm the repository was cloned successfully.**

![Screenshot of Jenkins Pipeline Configuration](placeholder_jenkins_pipeline_configuration.png)

---

### **Step 3.2: Jenkinsfile to Run Unit Tests**

#### **Pipeline Code:**

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/<your-repo>.git' // Replace with your private repo URL
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'python3 -m venv venv' // Create a virtual environment
                sh '. venv/bin/activate && pip install -r requirements.txt' // Install dependencies
                sh '. venv/bin/activate && pytest > pytest-results.txt' // Run tests and save results
                archiveArtifacts artifacts: 'pytest-results.txt', fingerprint: true // Archive test results
            }
        }
    }
}
```

### **Instructions:**

1. **Update the Jenkinsfile to include the new "Run Unit Tests" stage.**
2. **Run the pipeline again and check:**
   - **Dependency installation success in the logs.**
   - **pytest-results.txt under Archived Artifacts.**

![Screenshot of Jenkins Build Running Unit Tests](placeholder_jenkins_build_running_unit_tests.png)

---

### **Step 3.3: Jenkinsfile to Request User Approval**

#### **Pipeline Code:**

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/<your-repo>.git'
            }
        }
        stage('Run Unit Tests') {
            steps {
                sh 'python3 -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
                sh '. venv/bin/activate && pytest > pytest-results.txt'
                archiveArtifacts artifacts: 'pytest-results.txt', fingerprint: true
            }
        }
        stage('Approval to Continue') {
            steps {
                input message: "Tests passed! Do you approve the next step?", submitter: 'admin'
            }
        }
    }
}
```

### **Instructions:**

1. **Modify the Jenkinsfile to add the "Approval to Continue" stage.**
2. **Run the pipeline and wait for the approval prompt. Log in as admin to approve.**

![Screenshot of Jenkins Approval Request Step](placeholder_jenkins_approval_request_step.png)

---

## **Step 4: Configure GitHub Webhook**

### **Set Up a Webhook:**

1. **In your GitHub repository, go to Settings > Webhooks.**
2. **Add a webhook with:**
   - **Payload URL:** `http://<your-jenkins-server>:8080/github-webhook/`
   - **Content type:** `application/json`
   - **Trigger events:** Select **Pull requests**.

### **Enable Webhook in Jenkins:**

1. **In Jenkins, go to your job’s configuration and enable GitHub hook trigger for GITScm polling under Build Triggers.**

---

## **Step 5: Publish Results to GitHub PR**

### **Update Jenkinsfile to Publish Results:** Modify the "Publish Reports" stage to include PR checks:

```groovy
stage('Publish Reports') {
    steps {
        publishIssues issues: [
            scanForIssues tool: bandit(pattern: 'bandit-report.xml'),
            scanForIssues tool: dependencyCheck(pattern: 'dependency-check-report.xml')
        ]
        githubCheckStep(name: 'Security Scans', description: 'Results published to GitHub', context: 'sast-sca-check', conclusion: 'success')
    }
}
```

### **Test the Integration:**

1. **Open a pull request to the main branch in GitHub.**
2. **Verify that Jenkins runs the pipeline and publishes results to the PR as a status check.**

---

## **Visual Aid Placeholders**

- **Figure 1:** Diagram showing Jenkins integration with GitHub (webhooks triggering the pipeline).
- **Figure 2:** Example of Bandit and OWASP Dependency-Check results visualized in Jenkins.
- **Figure 3:** GitHub PR with Jenkins status checks ("Security Scans Passed").

