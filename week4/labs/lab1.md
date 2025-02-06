# **Lab: Shifting Left with Security Scanning, SAST, and SCA in CI Pipelines**

In this lab, you will enhance a basic CI pipeline by integrating **security scanning**, **Static Application Security Testing (SAST)** using **Semgrep**, and **Software Composition Analysis (SCA)** using **OWASP Dependency-Check**. These tools demonstrate the "shift left" approach by identifying vulnerabilities earlier in the software development lifecycle.

---

## **Initial Workflow**
We will start with the following CI workflow that builds and tests Python applications:

```yaml
name: CI Workflow

on:
  push:
    branches: [ main ]
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
        pytest tests/test_app.py
```

## **Step 1: Add Static Application Security Testing (SAST) with Semgrep**
### Objective:
Integrate Semgrep, a lightweight, rule-based SAST tool, to identify security vulnerabilities, insecure coding patterns, and potential bugs in the source code.

### Setup:
- Install Semgrep locally to explore its rulesets: [Semgrep Documentation](https://semgrep.dev/docs/).
- Use the `p/ci` ruleset, which includes pre-built rules for common security issues.

### Updated Workflow:
```yaml
jobs:
  sast-scan:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Install Semgrep
      run: |
        pip install semgrep

    - name: Run Semgrep Scan
      run: |
        semgrep --config p/ci --sarif --output semgrep-results.sarif

    - name: Upload SARIF Results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: semgrep-results.sarif
```
**Explanation:**
- Semgrep scans the source code for common vulnerabilities like SQL injection, hardcoded secrets, and unsafe use of libraries or functions.
- **SARIF Report**: Semgrep generates a report in SARIF format, which is uploaded to GitHub’s Security Tab for review.

## **Step 2: Add Software Composition Analysis (SCA) with OWASP Dependency-Check**
### Objective:
Introduce OWASP Dependency-Check to scan third-party libraries for known vulnerabilities and ensure compliance with security standards.

### Updated Workflow:
```yaml
jobs:
  dependency-check:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Run OWASP Dependency-Check
      uses: dependency-check/action@v3.3.0
      with:
        output-directory: ./dependency-check-report
        format: "SARIF"

    - name: Upload SARIF Results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: ./dependency-check-report/dependency-check-report.sarif
```
**Explanation:**
- OWASP Dependency-Check identifies vulnerabilities in project dependencies using databases like the National Vulnerability Database (NVD).
- **SARIF Report**: Uploads a detailed report to GitHub’s Security Tab to help developers address the identified risks.

## **Step 3: Combine All Jobs into a Unified Workflow**
The final workflow integrates Semgrep for SAST and OWASP Dependency-Check for SCA alongside the build and test jobs.

```yaml
name: CI Workflow with Security Scans

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

permissions:
  id-token: write
  contents: read

jobs:
  build-and-test:
    runs-on: ubuntu-latest
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
        pytest tests/test_app.py

  sast-scan:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Install Semgrep
      run: |
        pip install semgrep

    - name: Run Semgrep Scan
      run: |
        semgrep --config p/ci --sarif --output semgrep-results.sarif

    - name: Upload SARIF Results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: semgrep-results.sarif

  dependency-check:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Run OWASP Dependency-Check
      uses: dependency-check/action@v3.3.0
      with:
        output-directory: ./dependency-check-report
        format: "SARIF"

    - name: Upload SARIF Results
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: ./dependency-check-report/dependency-check-report.sarif
```

## **Step 4: Review Reports in GitHub’s Security Tab**
- Navigate to the Security Tab in your GitHub repository.
- View the vulnerabilities detected by Semgrep and OWASP Dependency-Check under the Code Scanning Alerts section.
- Address the flagged issues and rerun the pipeline to validate the fixes.

