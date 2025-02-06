# Step-by-Step Instructions

## Step 1: Create a New GitHub Repository

1. Go to GitHub and log in.
2. Click on **New** to create a new repository.
3. Name the repository (e.g., `flask-ci-workflow`) and set it to **Public**.
4. Check **Add a README file** and click **Create repository**.

## Step 2: Initialize a Simple Flask Application

1. Clone the repository to your local machine:
   ```bash
   git clone git@github.com:Sheridan-College-FAST-CloudSecurity/week4-1-<YOUR USERNAME>.git
   cd week4-1-<YOUR USERNAME>
   ```

2. Create a new file named `app.py` and add the following Flask application code:
   ```python
   from flask import Flask

   app = Flask(__name__)

   @app.route('/')
   def hello():
       return b"Hello, DevSecOps!"

   if __name__ == "__main__":
       app.run()
   ```

3. Create a `requirements.txt` file to define the Flask dependency:
   ```text
   Flask==2.0.3
   Werkzeug==2.2.2
   ```

## Step 3: Add Unit Tests for the Flask Application

1. Create a file `test_app.py` with the following content:
   ```python
   from app import app

   def test_hello():
       response = app.test_client().get('/')
       assert response.status_code == 200
       assert response.data == b"Hello, DevSecOps!"
   ```

2. Install dependencies and run the test locally to verify it works:
   ```bash
   pip install -r requirements.txt
   pytest test_app.py
   ```

## Step 4: Create a GitHub Actions Workflow for CI

1. In your repository, create a `.github/workflows` directory.
2. Inside the `workflows` directory, create a file named `ci.yml`.
3. Open `ci.yml` and add the following YAML content:
   ```yaml
   name: CI Workflow

   on:
     push:
       branches: [ main ]
     pull_request:
       branches: [ main ]

   jobs:
     build-and-test:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout repository
         uses: actions/checkout@v2

       - name: Set up Python
         uses: actions/setup-python@v2
         with:
           python-version: '3.12'

       - name: Install dependencies
         run: |
           python -m pip install --upgrade pip
           pip install -r requirements.txt
           pip install pytest

       - name: Run Tests
         run: |
           pytest test_app.py
   ```

## Step 5: Commit and Push the Workflow

1. Add, commit, and push the changes to GitHub:
   ```bash
   git add .
   git commit -m "Add CI workflow for Flask application"
   git push origin main
   ```

2. Once pushed, go to the **Actions** tab in your GitHub repository. You should see the CI workflow running automatically on the `main` branch.

## Step 6: Check for Test Failures

1. To simulate a test failure, modify `app.py` temporarily:
   ```python
   # Modify the return statement to introduce a failure
   def hello():
       return "Hello, DevSecOps - modified!"
   ```

2. Commit and push the change to the `main` branch.
3. Go to the **Actions** tab and observe the workflow failing due to the test mismatch.

## Step 7: Configure Automatic Reruns for Failed Jobs

1. In GitHub Actions, navigate to **Settings > Actions > General** for your repository.
2. Under **Workflow permissions**, enable **Read and write permissions** if not already enabled.
3. Use a retry feature by rerunning the failed workflow manually from the **Actions** tab. (GitHub does not currently support automatic reruns, but workflows can be retried manually or with additional automation plugins.)

## Step 8: Fix the Code and Rerun the Workflow

1. Revert `app.py` to its original state:
   ```python
   # Correct the return statement
   def hello():
       return "Hello, DevSecOps!"
   ```

2. Commit and push the change:
   ```bash
   git add .
   git commit -m "Fix test failure in hello endpoint"
   git push origin main
   ```

3. Observe the **Actions** tab to confirm that the workflow passes successfully after the fix.
