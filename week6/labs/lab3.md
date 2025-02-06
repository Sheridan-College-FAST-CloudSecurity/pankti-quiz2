# **Lab: Automating Docker Image Deployment with Jenkins**

## **Step 1: Configure Docker Credentials in Jenkins**

1. Log in to your Jenkins instance.
2. Navigate to **Manage Jenkins > Manage Credentials**.
3. Click on **(global)** under "Stores scoped to Jenkins".
4. Click **Add Credentials**.
5. Choose **Username with password** as the kind.
6. Enter your DockerHub username and password.
7. Set the ID as `dockerhub_credentials`.
8. Click **OK** to save the credentials.

---

## **Step 2: Create a Jenkinsfile**

1. Create a file named `Jenkinsfile` in your project repository with the following content:

```groovy
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_credentials')
    }

    stages {
        stage('Build') {
            steps {
                sh 'docker build -t your-dockerhub-username/your-image-name:$BUILD_NUMBER .'
            }
        }

        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Push') {
            steps {
                sh 'docker push your-dockerhub-username/your-image-name:$BUILD_NUMBER'
            }
        }
    }

    post {
        always {
            sh 'docker logout'
        }
    }
}
```

2. Replace `your-dockerhub-username` and `your-image-name` with your actual DockerHub username and desired image name.

---

## **Step 3: Create a Dockerfile**

1. In your project root, create a `Dockerfile` that defines your container image.
2. Ensure the `Dockerfile` has the appropriate instructions for building your application.

---

## **Step 4: Create a Jenkins Pipeline**

1. In Jenkins, click **New Item**.
2. Enter a name for your pipeline and select **Pipeline**.
3. In the Pipeline section, choose **Pipeline script from SCM**.
4. Select your SCM (e.g., Git) and enter your repository URL.
5. Specify the branch to build (e.g., `*/main`).
6. Set the Script Path to `Jenkinsfile`.
7. Click **Save**.

---

## **Step 5: Run the Pipeline**

1. In Jenkins, navigate to your pipeline.
2. Click **Build Now** to manually trigger the pipeline.
3. The pipeline will:
   - Build your Docker image.
   - Log in to DockerHub.
   - Push the image to DockerHub.
   - Log out of DockerHub.

---

## **Step 6: Verify the Results**

1. Check the Jenkins console output for any errors.
2. Log in to your DockerHub account and verify that the new image has been pushed successfully.

---

### **Security Note**

- Always keep your DockerHub credentials secure.
- Avoid exposing credentials in your code or version control system.

