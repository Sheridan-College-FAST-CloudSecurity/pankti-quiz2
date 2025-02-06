# **Lab Instructions: Configure Branch Protection and Check Runs in a Private GitHub Repository**

In this lab, you will configure branch protection rules and check runs in your private repository. Additionally, you will set up email notifications to alert you when workflows fail. These configurations will enforce secure, high-quality contributions to the `main` branch.

---

## **Objective**
By the end of this lab, you will have configured the following: 

1. Direct commits to the `main` branch are blocked.
2. Code changes can only be merged via pull requests (PRs).
3. Security and build workflows must pass before merging.
4. At least one approver must review and approve PRs before merging.
5. Success/failure of check runs will be displayed in pull requests.
6. Email notifications will alert you when workflows fail.
7. Prevent incomplete or insecure changes from being merged.

---

## **Step-by-Step Instructions**

### **Step 1: Access Branch Protection Settings**
1. Open your **GitHub repository**.
2. Click on the **Settings** tab in the repository menu.
3. In the left-hand menu, under **Code and automation**, click on **Branches**.
4. Locate the **Branch protection rules** section and click **Add rule**.

---

### **Step 2: Block Direct Commits to the Main Branch**
1. In the **Branch name pattern** field, type `main`.
   - This ensures the rule applies to the `main` branch.
2. Check the box for **Require a pull request before merging**.
   - This blocks direct commits and ensures all changes go through pull requests.

---

### **Step 3: Require Passing Status Checks**
1. Check the box for **Require status checks to pass before merging**.
2. Under "Status checks that are required," click **Edit** and add the following:
   - `Security Scan Workflow` (e.g., a workflow for security scans).
   - `Build Workflow` (e.g., a workflow for building and testing the code).
3. Check the box for **Require branches to be up to date before merging**.
   - This ensures the PR branch is synchronized with the latest changes from `main`.

---

### **Step 4: Require Pull Request Approvals**
1. Check the box for **Require approvals** under "Pull request reviews".
2. Set the **Number of required reviewers** to **1**.
   - This ensures at least one team member reviews and approves the PR before merging.

---

### **Step 5: Enable Success/Failure Reporting for Check Runs**
1. Confirm that **Require status checks to pass before merging** is enabled (as configured in Step 3).
2. GitHub will now display the pass/fail status of the workflows in the PR details.

---

### **Step 6: Prevent Incomplete or Insecure Changes**
1. Under "Require pull request reviews," check **Dismiss stale pull request approvals when new commits are pushed**.
   - This ensures that new commits automatically invalidate previous approvals.
2. Check the box for **Require signed commits** under "Commits".
   - This requires all commits to be signed, verifying their authenticity.
3. Check the box for **Prevent merge commits**.
   - This enforces a clean commit history by disallowing merge commits.

---

### **Step 7: Set Up Email Notifications for Workflow Failures**
1. Open the **.github/workflows** directory in your repository (create it if it doesn’t exist).
2. Create a new workflow file named `email-notification.yml` with the following content:

```yaml
name: Notify on Workflow Failure

on:
  workflow_run:
    workflows: 
      - "Security Scan Workflow"
      - "Build Workflow"
    types:
      - completed

jobs:
  notify-failure:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    runs-on: ubuntu-latest
    steps:
    - name: Send Email Notification
      uses: dawidd6/action-send-mail@v3
      with:
        server_address: smtp.gmail.com
        server_port: 465
        username: ${{ secrets.SMTP_USERNAME }}
        password: ${{ secrets.SMTP_PASSWORD }}
        subject: "Workflow Failure Alert: ${{ github.workflow }}"
        to: student_email@example.com
        from: "GitHub Actions <your_email@example.com>"
        body: |
          The workflow ${{ github.workflow }} failed for the following branch:
          Repository: ${{ github.repository }}
          Branch: ${{ github.ref_name }}
          URL: ${{ github.event.workflow_run.html_url }}
```

### **Configure Secrets**
Navigate to your repository’s **Settings > Secrets and variables > Actions**.
Add the following secrets:
- **SMTP_USERNAME**: Your email username (e.g., Gmail address).
- **SMTP_PASSWORD**: Your email password or an app-specific password if using Gmail.

### **Verify Notifications**
Trigger a failure in one of your workflows and verify that you receive an email notification.

---

## **Verification Checklist**

### **Direct Commit Blocking**
- Try pushing a commit directly to the `main` branch.
- Verify that the push is rejected with a message requiring a pull request.

### **Required Status Checks**
- Open a new PR and verify that the `Security Scan` and `Build Workflow` checks are listed under required checks.
- Confirm that the PR cannot be merged until both workflows pass.

### **Pull Request Approval**
- Create a PR and verify that it cannot be merged without approval.
- Ensure an approver reviews and approves the PR.

### **Stale Approval Test**
- Add a new commit to an approved PR and confirm that the approval is dismissed automatically.

### **Email Notification Test**
- Trigger a failure in one of the workflows and verify that an email notification is sent with the correct details.

---

## **Expected Outcomes**
- **Direct Commits Blocked**: All changes must go through pull requests.
- **Required Status Checks**: Security and build workflows must pass before merging.
- **Pull Request Approval**: At least one approver is required for PR merges.
- **Email Notifications**: Workflow failures trigger automated email alerts.
- **Stale Approvals**: New commits invalidate existing approvals.

