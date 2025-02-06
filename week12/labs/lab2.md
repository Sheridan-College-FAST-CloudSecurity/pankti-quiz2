# **Lab 2: Implementing CI/CD with GitHub Actions**

## **Overview**

In this lab, students will:

1. Create a GitHub Action to perform OPA policy checks and run Checkov scans.
2. Configure pull request (PR) checks to validate compliance with OPA policies and Checkov results.
3. Set up a workflow to run the checks on every branch for every commit.
4. Create another GitHub workflow that triggers on commits to the default branch to deploy a development environment.
5. Reuse the Terraform workflow from Lab 1 and add steps to validate the application running on the bastion host.
6. Configure a `dev` environment in GitHub for deployment.
7. Configure a "prod" environment in GitHub for deployment.
8. Update the workflow to deploy to "dev" and to "prod"
9. Add a manual approval action from the GitHub marketplace to require review before deployment to `prod`
10. Use native manual approvals support of GitHub Environments available with GitHub Pro.

---

## **Step-by-Step Instructions**

### **Step 1: Create GitHub Actions for OPA and Checkov**

1. In your repository, create a new directory `.github/workflows/` if it doesn’t already exist.
2. Create a new workflow file `.github/workflows/opa-checkov.yml`:

```yaml
name: OPA and Checkov Validation

on:
  push:
    branches:
      - "*"
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4

      - name: Install Checkov
        run: pip install checkov

      - name: Install Conftest
        run: |
          curl -L https://github.com/open-policy-agent/conftest/releases/download/v0.35.0/conftest_0.35.0_Linux_x86_64.tar.gz -o conftest.tar.gz
          tar -xvf conftest.tar.gz -C /usr/local/bin
          rm conftest.tar.gz

      - name: Run OPA policy checks
        run: conftest test .

      - name: Run Checkov scan
        run: checkov -d .
```

3. Commit and push the file to your repository.

### **Step 2: Configure Pull Request Checks**

1. In your repository, go to **Settings** > **Branches** > **Branch Protection Rules**.
2. Add a rule for the default branch and ensure:
   - PRs require the `OPA and Checkov Validation` workflow to pass before merging.
   - Direct commits to the branch are blocked.

### **Step 3: Create Deployment Workflow for the Default Branch**

1. Create another workflow `.github/workflows/deploy.yml`:

```yaml
name: Deployment Workflow

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed"```
```

### **Step 4: Configure Dev Environment And Deploy to Dev**

1. In your GitHub repository, go to **Settings** > **Environments**.
2. Create a new environment called `dev`.
3. Add the necessary AWS credentials as secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_SESSION_TOKEN`
4. Specify the `dev` environment in the `Deployment Workflow` workflow.
```yaml
name: Deployment Workflow

on:
  push:
    branches:
      - main

jobs:
  deploy-to-dev:
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""
    environment: dev  
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve -var "environment=dev" tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed in Dev environment"
```
### **Step 5: Add Deployment to Prod**

1. In your GitHub repository, go to **Settings** > **Environments**.
2. Create a new environment called `prod`.
3. Add the necessary AWS credentials as secrets:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - `AWS_SESSION_TOKEN`
4. Add deployment to `prod` job in the `Deployment Workflow`.
```yaml
name: Deployment Workflow

on:
  push:
    branches:
      - main

jobs:
  deploy-to-dev:
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""
    environment: dev  
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve -var "environment=dev" tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed in Dev"

  deploy-to-prod:
    needs: [deploy-to-dev]
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""
    environment: prod
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve -var "environment=prod" tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed in Prod"
```
### **Step 6: Add Manual Approval Step before Deploying to Prod**

1. (Manual approval with GitHub Actions)[!https://github.com/marketplace/actions/manual-workflow-approval].
2. (Manual approval with Environments)[!https://docs.github.com/en/actions/managing-workflow-runs-and-deployments/managing-deployments/reviewing-deployments]
3. This action implements a free alternative for the Environments approvals available in GitHub Enterprise only for private repos.
   
#### **Manual Approval Action Workflow**
This action introduces a manual approval process in GitHub Actions workflows.

#### **How It Works**

1. The workflow reaches the `manual-approval` action.
2. The `manual-approval` action creates an issue in the repository where the workflow is running.
3. The issue is assigned to the specified approvers.
4. The approvers review the issue and respond with an approval or denial keyword.
   - If **all approvers** respond with an **approval keyword**, the workflow continues.
   - If **any approver** responds with a **denial keyword**, the workflow exits with a failed status.

**Approval Keywords**
- `approve`, `approved`, `lgtm`, `yes`

**Denial Keywords**
- `deny`, `denied`, `no`

```yaml
name: Deploy to Dev Environment

on:
  push:
    branches:
      - main

jobs:
  deploy-to-dev:
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""
    environment: dev  
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve -var "environment=dev" tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed"

      - uses: trstringer/manual-approval@v1
        with:
          secret: ${{ github.TOKEN }}
          approvers: user1 # Replace with your GitHub user
          minimum-approvals: 1
          issue-title: "Deploying to prod from dev"
          issue-body: "Please approve or deny the deployment."
          exclude-workflow-initiator-as-approver: false
          additional-approved-words: ''
          additional-denied-words: ''

  deploy-to-prod:
    needs: [approve, deploy-to-dev]
    runs-on: ubuntu-latest
    env:
      BASTION_PUBLIC_IP: ""
    environment: prod
    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set AWS Credentials
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        run: echo "AWS credentials configured"

      - name: Set up Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: 1.5.0

      - name: Initialize Terraform
        run: terraform init

      - name: Plan Terraform
        run: terraform plan -out=tfplan

      - name: Apply Terraform
        run: terraform apply -auto-approve -var "environment=dev" tfplan

      - name: Output Bastion Public IP
        id: bastion_ip
        run: |
          echo "::set-output name=bastion_public_ip::$(terraform output -raw bastion_public_ip)"
        env:
          BASTION_PUBLIC_IP: ${{ steps.bastion_ip.outputs.bastion_public_ip }}

      - name: Validate Application on Bastion
        run: |
          APP_RESPONSE=$(curl -s http://$BASTION_PUBLIC_IP:80)
          if [[ "$APP_RESPONSE" != *"nginx"* ]]; then
            echo "Application is not running as expected"
            exit 1
          fi
          echo "Application validation passed"```
```
### **Step 7: Setting Prod Environment to Require Reviewers

1. **Navigate to your repository settings:**
   - Go to the main page of your repository on GitHub.
   - Click on the `Settings` tab.

2. **Access Environments:**
   - In the left sidebar, click on `Environments` under the `Security` section.

3. **Create or select the prod environment:**
   - Click on edit to modify the`prod` environment.

4. **Configure required reviewers:**
   - Scroll down to the `Environment protection rules` section.
   - Click on `Add required reviewers`.
   - In the `Required reviewers` field, add the GitHub usernames or team names of the required reviewers.
   - Click `Save protection rules` to apply the settings.
5. **Confirm the setup:**
   - Ensure that the `prod` environment now lists the required reviewers under its protection rules.
     
### Step 8: Approve Deployment to Prod Environment

Remove the `approve` step from the workflow and trigger the workflow by committing new code to the main branch.
This will start the `Deployment Workflow`. The workflow will stop and wait for approval before deployment to the `prod` environment.
To approve deployment to the `prod` environment configured with required approvers, follow these steps:

1. **Navigate to the GitHub Actions tab:**
   - Go to the main page of your repository on GitHub.
   - Click on the `Actions` tab.

2. **Find the Workflow Run:**
   - In the Actions tab, find the workflow run that is waiting for approval.
   - Click on the workflow run to view the details.

3. **Review Deployment:**
   - Locate the job waiting for approval under the `prod` environment.
   - Review the deployment details provided in the workflow logs.

4. **Approve the Deployment:**
   - Click the `Review deployments` button.
   - Provide any necessary comments and click `Approve and deploy` to approve the deployment to the `prod` environment.

By completing these steps, you ensure the deployment to the `prod` environment is properly reviewed and approved.


