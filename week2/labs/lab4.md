## Part 1: Configuring Branch Protection Rules

### Step 1: Access Your Repository

1. Log in to your GitHub account.
2. Navigate to the repository where you want to set up the branch protection rules.

### Step 2: Access Repository Settings

1. Click on the "Settings" tab in the upper right corner of the repository page.
2. In the left sidebar, select "Branches."

### Step 3: Add Branch Protection Rules

1. In the "Branch protection rules" section, click on "Add rule."

### Step 4: Require Pull Request Reviews

1. In the Branch name pattern field, enter the branch you want to protect (e.g., main).
2. Check the box for "Require pull request reviews before merging."
3. Optionally, specify the number of required approvals by adjusting the settings below.

### Step 5: Require Status Checks 

1. Below the pull request reviews option, check the box for "Require status checks to pass before merging."
2. Select the specific status checks you want to enforce from the dropdown menu. (Note: Skip this step for now. We will get back to status checks next week.)
   - Example: Select checks like "build" or "tests."

### Step 6: Prevent Bypassing Settings

1. Check the box for "Do not allow bypassing the above settings."
   - This ensures that even users with admin access cannot merge without reviews and passing checks.

### Step 7: Restrict Push Access

1. Check the box for "Restrict who can push to matching branches."
2. Specify which users or teams can push directly to this branch.
   - You can add specific GitHub users or teams that have permission to push.

### Step 8: Save Changes

1. Click on the "Create" button to save your branch protection rule.

## Part 2: Defining Rulesets

### Step 1: Access Organization Settings (if applicable)

1. Navigate to the "Organizations" page on GitHub (if your repository is under an organization).
2. Select your organization and click on "Settings."

### Step 2: Go to Rulesets

1. In the organization settings sidebar, select "Rulesets."

### Step 3: Create a New Ruleset

1. Click on "New ruleset."
2. Enter a descriptive name for your ruleset (e.g., "Repository Security Ruleset").

### Step 4: Set Permissions for Access, Edit, or Push Changes

1. In the ruleset configuration, define who can access, edit, or push changes to the repository.
2. Specify permissions for different teams or users using GitHub teams/groups (e.g., Developers, Reviewers).

### Step 5: Specify Commit Standards

1. Look for options to specify commit message standards.
2. You might use a regular expression to define the required format (e.g., `^\[JIRA-\d+\] .*` for ticket-based commit messages).

### Step 6: Restrict Specific Actions

1. In the same ruleset, find settings to restrict actions such as:
   - **Force pushes**: Check to prevent users from force pushing to the branches.
   - **Deleting branches**: Specify if branch deletions should be restricted.

### Step 7: Review and Save the Ruleset

1. Review the rules you have defined.
2. Click on "Create" or "Save" to activate the ruleset.

