# Jenkins Pipeline Lab - Step-by-Step Instructions

This lab guides you through creating and testing a Jenkins pipeline step by step. The pipeline will clone a private GitHub repository, install dependencies for a Python application, run unit tests using `pytest`, and request user approval before proceeding to the next step.

---

## Summary of Variables and Credentials Used

| Variable/Credential                               | Description                                                                                                                   |
| ------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **GitHub Personal Access Token (PAT)**            | Token used to authenticate with GitHub for accessing private repositories. Generated in Step 1.                               |
| **Jenkins Credentials ID** (`github-credentials`) | ID for the credentials added to Jenkins for GitHub access. Used to securely store and retrieve GitHub PAT in the Jenkinsfile. |
| **Environment Variables** (`GIT_CREDENTIALS`)     | Environment variable in Jenkins to access the GitHub credentials securely.                                                    |

---

## Step 1: Generate a GitHub Personal Access Token (PAT)

![Screenshot of GitHub Developer Settings]\(placeholder\_github\_developer\_settings.png)

To enable Jenkins to authenticate with a private GitHub repository, you’ll need a **Personal Access Token (PAT)**.

### Instructions:

1. **Log in to GitHub**:

   - Open [GitHub](https://github.com/) and log in to your account.

2. **Navigate to Developer Settings**:

   - Click your profile picture (top-right corner) and go to **Settings**.
   - Scroll down to the **Developer settings** section and click on it.
   - Select **Personal access tokens > Tokens (classic)** from the left-hand menu.

3. **Generate a New Token**:

   - Click **Generate new token (classic)**.
   - Set a **Note** (e.g., `Jenkins Access`) to identify the token.
   - Set an **Expiration** (e.g., 30 days) for security purposes.
   - Under **Scopes**, select:
     - **`repo`**: Full access to private repositories.
     - **`read:org`**: (Optional) Read access for organization repositories.

4. **Copy the Token**:

   - Click **Generate Token** and copy it immediately (you won’t be able to see it again).
   - Store the token securely; you will need it in the next step.

---

## Step 2: Configure Jenkins Credentials



![Screenshot of Jenkins Credentials Section]\(placeholder\_jenkins\_credentials\_section.png)

### Instructions:

1. **Access Jenkins**:

   - Open Jenkins in your browser (e.g., `http://<your-jenkins-server>:8080`) and log in.

2. **Navigate to the Credentials Section**:

   - Go to **Manage Jenkins > Credentials > System > Global credentials (unrestricted)**.
   - Click **Add Credentials**.

3. **Add GitHub Credentials**:

   - Set the following:
     - **Kind**: Select **Username with password** or **Secret text**.
     - **Scope**: Global.
     - **Username**: Your GitHub username.
     - **Password/Secret Text**: Paste your GitHub personal access token.
     - **ID**: Set a recognizable ID (e.g., `github-credentials`).
   - Click **OK** to save the credentials.

---

## Step 3: Create and Test the Jenkins Pipeline

This step is divided into three parts to incrementally build and test the Jenkins pipeline.

### Step 3.1: Jenkinsfile That Clones the Repository



![Screenshot of Jenkins Pipeline Configuration]\(placeholder\_jenkins\_pipeline\_configuration.png)

Create a basic pipeline to clone your private GitHub repository.

#### Jenkinsfile Code:

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
                    url: 'https://github.com/your-private-repo.git' // Replace with your private repo URL
            }
        }
    }
}
```

### Instructions:

1. **Configure the Pipeline**:

   - Go to the Jenkins dashboard and create a new Pipeline job.
   - In the Pipeline section, paste the above Jenkinsfile code.

2. **Run the Build**:

   - Click **Build Now** to execute the pipeline.

3. **Verify Output**:

   - Check the **Console Output** to confirm the repository was cloned successfully:
     ```
     Cloning repository https://github.com/your-private-repo.git
     Checking out branch main
     ```

---

### Step 3.2: Jenkinsfile That Runs Unit Tests



![Screenshot of Jenkins Build Running Unit Tests]\(placeholder\_jenkins\_build\_running\_unit\_tests.png)

Extend the pipeline to install dependencies and run unit tests using `pytest`.

#### Jenkinsfile Code:

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/your-private-repo.git' // Replace with your private repo URL
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

### Instructions:

1. **Update the Pipeline Code**:

   - Replace the Jenkinsfile code from Step 3.1 with the new code above.

2. **Run the Build**:

   - Click **Build Now** to execute the updated pipeline.

3. **Verify Output**:

   - Ensure dependencies are installed, and `pytest` runs successfully.
   - Check the `pytest-results.txt` file under **Archived Artifacts**.

---

### Step 3.3: Jenkinsfile That Requests User Approval



![Screenshot of Jenkins Approval Request Step]\(placeholder\_jenkins\_approval\_request\_step.png)

Add a stage that pauses the pipeline and requests user approval before proceeding.

#### Jenkinsfile Code:

```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/your-private-repo.git' // Replace with your private repo URL
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
        stage('Approval to Continue') {
            steps {
                input message: "Tests passed! Do you approve the next step?", submitter: 'admin'
            }
        }
    }
}
```

### Instructions:

1. **Update the Pipeline Code**:

   - Replace the Jenkinsfile code with the new version above.

2. **Run the Build**:

   - Click **Build Now** to execute the updated pipeline.

3. **Verify Output**:

   - Jenkins should pause at the **Approval to Continue** stage.
   - Approve or reject the request as the admin user.

---

## Step 4: Test and Validate the Full Pipeline

### Instructions:

1. **Run the pipeline multiple times**:

   - Test with changes to the repository (e.g., modifying `requirements.txt` or adding a failing test).
   - Observe how the pipeline behaves in different scenarios.

2. **Verify all stages**:

   - Ensure the repository is cloned, tests run successfully, and the approval step pauses the pipeline as expected.

