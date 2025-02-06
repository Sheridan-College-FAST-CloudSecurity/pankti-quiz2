# **Lab: Advanced Continuous Integration and Deployment with GitHub Actions**

In this lab, you will build a powerful, secure, and maintainable GitHub Actions CI/CD pipeline. The pipeline will incorporate **dependent jobs**, **matrix strategy**, **Docker containerization**, and **deployment to Amazon Elastic Container Registry (ECR)**. You will also explore **reusable workflows**, enabling modularity and consistency in automation.

---

## **Lab Objectives**
By the end of this lab, you will:

1. Create workflows with **dependent jobs** to enforce execution order.
2. Use a **matrix strategy** to test code across multiple Linux distributions.
3. Build and test a **Docker container** for your application.
4. Deploy the containerized application to **Amazon ECR** securely.
5. Create and use **reusable workflows** for modularity.
6. Implement **GitHub Actions Security Best Practices**.

---

## **Step 1: Create a Workflow with a Matrix Strategy**
You will modify your existing workflow to include a **matrix strategy**, enabling testing across multiple Linux distributions.

### **Workflow Configuration**
```yaml
name: CI NEW Workflow

on:
  push:
    branches: 
      - '**' # Match all branches
  pull_request:
    branches: [ main ]

permissions:
  id-token: write
  contents: read

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    # Add a matrix strategy for multiple Linux distributions
    strategy:
      matrix:
        os: [ubuntu-latest, ubuntu-22.04, ubuntu-20.04]

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.8'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run Tests
      run: |
        pytest test_app.py
```
**What You Learn**:
- The matrix strategy runs the same workflow across multiple Linux distributions (ubuntu-latest, ubuntu-22.04, and ubuntu-20.04).
- This ensures your application works consistently across different environments.

**Reference**:
- [Matrix builds in GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstrategy)

---

## **Step 2: Add Dependent Jobs**
We are going to add a new job that depends on the successful completion of the `build-and-test` job.
The new job builds a docker image.
Use the `needs` keyword to make the Docker image build dependent on the successful completion of tests.
Add the Dockerfile below to the root of your repository.

```
# Use the official Python 3.12 image from the Docker Hub
FROM python:3.12-slim

# Set the working directory
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the dependencies
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install pytest

# Copy the rest of the application code into the container
COPY *.py .

# Expose the port the app runs on
EXPOSE 5000

# Command to run the application, making port 5000 externally accessible
CMD ["flask", "run", "--host=0.0.0.0", "--port=5000"]
```
### Build the Docker image and create the container

```
docker build -t my-app:latest .
docker run --rm my-app:latest pytest test_app.py
```

### **Updated Workflow**
```yaml
docker-build:
  needs: build-and-test  # Runs only after tests pass
  runs-on: ubuntu-latest

  steps:
  - name: Checkout repository
    uses: actions/checkout@v2

  - name: Build Docker Image
    run: |
      docker build -t my-app:latest .
```
**What You Learn**:
- **Dependent Jobs**: The `docker-build` job only runs if `build-and-test` passes, enforcing proper execution order and improving pipeline reliability.

**Reference**:
- [Dependent jobs in GitHub Actions](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)

---

## **Step 3: Build and Test a Docker Container**
Next, extend the pipeline to build and test a Docker container for the application.

### **Updated Workflow**
```yaml
  - name: Run Container Tests
    run: |
      docker run --rm my-app:latest pytest test_app.py
```
**What You Learn**:
- **Container Testing**: Ensure your application works as expected in its containerized environment.
- Run tests inside the Docker container to validate container behavior.

**Reference**:
- [Docker container testing](https://docs.docker.com/develop/dev-best-practices/)

---

## **Step 4: Deploy to Amazon ECR**
Securely log in to Amazon ECR and push the Docker image to the registry using GitHub Secrets.

### **Updated Workflow**
```yaml
  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v3
    with:
      role-to-assume: arn:aws:iam::<AWS_ACCOUNT_ID>:role/<ROLE_NAME>
      aws-region: us-east-1

  - name: Log in to Amazon ECR
    run: |
      aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

  - name: Push Docker Image to Amazon ECR
    run: |
      docker tag my-app:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
      docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```
**What You Learn**:
- Use OIDC authentication to securely log in to AWS without storing static credentials.
- Push your Docker image to Amazon ECR for deployment.

**Reference**:
- [Amazon ECR Authentication](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)

---

## **Step 5: Create and Use Reusable Workflows**
Reusable workflows simplify shared tasks across repositories.

### **Create a Reusable Workflow**
Create a new workflow file (`.github/workflows/build-and-test.yml`) in your repository.

```yaml
name: Reusable Build and Test Workflow

on:
  workflow_call:
    inputs:
      python-version:
        required: true
        type: string
      os-matrix:
        required: true
        type: array

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        os: ${{ inputs.os-matrix }}

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: ${{ inputs.python-version }}

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: Run Tests
      run: |
        pytest tests/test_app.py
```
### **Use the Reusable Workflow**
Call the reusable workflow in your main CI pipeline:

```yaml
jobs:
  reuse-build-and-test:
    uses: my-org/my-repo/.github/workflows/build-and-test.yml
    with:
      python-version: '3.8'
      os-matrix: [ubuntu-latest, ubuntu-22.04]
```
**What You Learn**:
- **Reusable workflows** improve maintainability and reduce duplication.
- You can call reusable workflows with different input parameters to adapt to diverse projects.

**Reference**:
- [Reusable workflows in GitHub Actions](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

---

## **Step 6: Implement GitHub Actions Security Best Practices**
In this step, you will enhance the security of your GitHub Actions workflows by following best practices for permission management, credential handling, and action usage. These practices help minimize risks and protect sensitive information during CI/CD execution.

### **Instructions**

### **1. Use Minimal Permissions for `GITHUB_TOKEN`**
The `GITHUB_TOKEN` automatically generated for your workflows should follow the principle of least privilege. Update your workflow to explicitly specify the required permissions for each job.

#### Update Workflow Permissions:
```yaml
permissions:
  contents: read         # Minimum access to repository content
  id-token: write        # Required for OIDC authentication with AWS
  packages: write        # Needed for pushing Docker images to ECR
```

**What You Learn**:
- Minimize permissions granted to the `GITHUB_TOKEN` to enhance security.
- Specify only the necessary permissions, reducing the attack surface during CI/CD execution.

**Reference**:
- [GitHub Actions Security Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

---

## **Final Workflow**
Your final pipeline includes:

1. A matrix strategy for multi-environment testing.
2. Dependent jobs to enforce proper execution order.
3. Building and testing Docker containers.
4. Deploying the Docker image to Amazon ECR.
5. Using reusable workflows for modularity.
6. Implementing security best practices.

This lab teaches advanced GitHub Actions techniques for real-world CI/CD scenarios, ensuring security and maintainability throughout. 🎓
