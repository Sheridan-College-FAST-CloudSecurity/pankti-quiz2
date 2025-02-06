# Assignment Overview

This assignment is designed to assess your proficiency across three key areas: Python development, Containerization, and DevOps/DevSecOps practices. You will develop a Python application, containerize it using Docker, and implement continuous integration (CI) workflows with GitHub Actions. The assignment will be managed through GitHub Classroom and will utilize autograding where applicable.

## Assignment Overview

### Python Development

- Develop a Python application featuring a RESTful GET API to retrieve items from a file (e.g., a list of books).
- Implement logging and unit tests for the application.

### Containerization

- Create a Dockerfile to containerize the Python application.
- Build and test the Docker image locally.

### DevOps and DevSecOps Practices

- Set up GitHub Actions workflows triggered on each pull request.
- Integrate unit tests, static code analysis, and Static Application Security Testing (SAST).
- Build and scan the Docker image, and publish it to Amazon Elastic Container Registry (ECR).
- Protect the main branch, enable code scanning and Dependabot, and remediate identified vulnerabilities.

## Detailed Steps

### 1. Python Development

**Goal**: Develop a Python application with a RESTful GET API, implement logging, and create unit tests.

**Steps**:

#### Set Up the Development Environment

- Initialize a new Python project.
- Create a virtual environment and install necessary packages (e.g., Flask for the API).

**Image Placeholder**: Diagram showing the development environment setup, including Python installation, virtual environment, and Flask package.

#### Develop the RESTful API

- Create an API endpoint that reads items from a file (e.g., books.txt) and returns them as a JSON response.

**Image Placeholder**: Screenshot of the code snippet showing the API endpoint, including the route definition and response structure.

#### Implement Logging

- Set up logging to record API requests and responses, including timestamps and status codes.

**Image Placeholder**: Example log output showing API requests, timestamps, and status codes, illustrating the logging format.

#### Write Unit Tests

- Develop unit tests to verify the functionality of the API using a testing framework like unittest or pytest.

**Image Placeholder**: Screenshot showing unit tests written for the API, including a successful test run output.

**Expected Outcome**: A functional Python application with a RESTful GET API, comprehensive logging, and passing unit tests.

### 2. Containerization

**Goal**: Containerize the Python application using Docker and test it locally.

**Steps**:

#### Create a Dockerfile

- Write a Dockerfile that sets up the environment, installs dependencies, and runs the application.

**Image Placeholder**: Screenshot of the Dockerfile, highlighting the key instructions such as base image, dependencies, and entry point.

#### Build the Docker Image

- Use Docker to build the image locally.

**Image Placeholder**: Terminal output showing successful Docker image build, including the image ID and build steps.

#### Test the Docker Container

- Run the container locally and verify that the API functions as expected.

**Image Placeholder**: Screenshot of the Docker container running, with a successful response from the API endpoint.

**Expected Outcome**: A Docker image that encapsulates the Python application, successfully running and serving the API locally.

### 3. DevOps and DevSecOps Practices

**Goal**: Implement CI workflows with GitHub Actions, integrate security testing, and manage the Docker image lifecycle.

**Steps**:

#### Set Up GitHub Actions Workflows

- Create workflows triggered on pull requests to run unit tests and perform static code analysis.

**Image Placeholder**: Screenshot of GitHub Actions workflow YAML file, showing the configuration for triggering on pull requests.

#### Integrate Static Code Analysis and SAST

- Use Pylint for static code analysis.
- Use Bandit for SAST.
- Configure these tools in the CI pipeline, ensuring results are posted to the GitHub Security tab.

**Image Placeholder**: Screenshot of the GitHub Actions pipeline run, highlighting static code analysis and SAST results.

#### Build and Scan the Docker Image

- Build the Docker image within the CI pipeline.
- Use Trivy to scan the Docker image for vulnerabilities.
- Publish scan results to the GitHub Security tab.

**Image Placeholder**: Screenshot of Trivy scan results, showing identified vulnerabilities and their severity levels.

#### Deploy to Amazon ECR

- If the container passes all tests, push the image to Amazon ECR.

**Image Placeholder**: Screenshot of the AWS console, showing the Docker image successfully pushed to Amazon ECR.

#### Implement Branch Protection and Security Features

- Protect the main branch to require pull request reviews before merging.
- Enable GitHub's code scanning and Dependabot alerts.

**Image Placeholder**: Screenshot of GitHub branch protection settings, showing enabled features like required reviews.

#### Remediate Vulnerabilities

- Review identified vulnerabilities, select one, and update the code or dependencies to address it.
- Rebuild and redeploy the Docker image, ensuring the issue is resolved.

**Image Placeholder**: Screenshot of the updated code, with comments explaining the vulnerability fix, and a successful rebuild in the CI pipeline.

**Expected Outcome**: A robust CI pipeline that tests, analyzes, and secures the application, with a Docker image deployed to Amazon ECR and proactive vulnerability management.

## Submission Requirements

- **Word Document**: Submit a Word document that includes screenshots demonstrating each of the key steps. Ensure the screenshots are clear and include annotations where necessary to explain what is being shown.
- **GitHub Repository**: Provide a link to your private GitHub repository in GitHub Classroom, containing all the implementation code. Ensure that the repository is accessible to the instructor for grading purposes.

## Grading Rubric

| Criteria                               | Points |
|----------------------------------------|--------|
| **Python Development (30 Points)**     |        |
| RESTful API Functionality              | 10     |
| Logging Implementation                 | 10     |
| Unit Tests Coverage and Passing        | 10     |
| **Containerization (20 Points)**       |        |
| Dockerfile Creation                    | 10     |
| Successful Local Testing               | 10     |
| **DevOps and DevSecOps Practices (50 Points)** | |
| CI Workflow Setup                      | 10     |
| Static Code Analysis Integration       | 10     |
| SAST Integration                       | 10     |
| Docker Image Scanning and Publishing   | 10     |
| Branch Protection and Security Features | 10    |

**Total Points: 100**

