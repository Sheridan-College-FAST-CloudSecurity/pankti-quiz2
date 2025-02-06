# Lab1: Basic Git Flow

## Step 1: Create a New Repository in a Folder

```bash
mkdir my_project  # Create a new directory for your project
cd my_project     # Navigate into the project folder
git init          # Initialize a new Git repository in this folder
```

**Explanation**: The `git init` command initializes a new Git repository in the specified folder, allowing you to start tracking changes in your project files.

## Step 2: Configure Git Username and Email

```bash
git config --global user.name "Your Name"       # Set the global Git username
git config --global user.email "your@email.com"  # Set the global Git email address
```

**Explanation**: These commands configure your Git identity, which will be associated with all your commits. This helps track changes and identify who made specific modifications in a collaborative environment.

## Step 3: Add the First Commit

```bash
touch README.md                # Create a new README file
git add README.md              # Stage the README file for commit
git commit -m "Initial commit" # Commit the staged file with a message
```

**Explanation**: The `touch` command creates a new README file. The `git add` command stages the file, making it ready to be committed. Finally, `git commit` permanently records the changes in the repository with a descriptive message.

## Step 4: Modify the File and Commit the Changes

```bash
echo 'Project description' >> README.md  # Modify the README file by adding a project description
git add README.md                        # Stage the modified file for commit
git commit -m "Update README with project description"  # Commit the changes with a descriptive message
```

**Explanation**: The `echo` command appends text to the README file, modifying it. The `git add` command stages the changes for the next commit, and `git commit` records the new changes in the repository.

## Step 5: Roll Back Changes

```bash
git log                         # View the commit history to identify the commit hash
git revert <commit-hash>        # Revert the specified commit, creating a new commit that undoes the changes
git reset --hard <commit-hash>  # Alternatively, reset to a previous commit (warning: can lead to data loss)
```

**Explanation**: The `git log` command helps you view the commit history and identify the commit you want to revert. The `git revert` command creates a new commit that undoes the changes while preserving the history. The `git reset --hard` command can also roll back changes to a specific commit, but it should be used with caution as it can permanently discard changes.

## Step 6: Secure the Local Code with Pre-Commit Hooks

```bash
# Install the pre-commit tool
pip install pre-commit  # Install the pre-commit tool globally
pip install black
pip install flake8
pip install isort

# Create a configuration file for pre-commit
# configure the pre-commit command
nano .pre-commit-config.yaml
```
#### Past the content below and save the file
```yaml
# See https://pre-commit.com/hooks.html for more hooks
# .pre-commit-config.yaml
repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v4.5.0
    hooks:
    -   id: trailing-whitespace
    -   id: end-of-file-fixer
    -   id: check-yaml
-   repo: https://github.com/psf/black-pre-commit-mirror
    rev: 24.1.1
    hooks:
    -   id: black
        language_version: python3.12
        args: ["--line-length=79"]
-   repo: https://github.com/pycqa/isort
    rev: 5.13.2
    hooks:
    -   id: isort
-   repo: https://github.com/pycqa/flake8
    rev: 7.0.0
    hooks:
    -   id: flake8
```
#### Run pre-commit command
```bash
pre-commit run --all-files
```
#### Install pre-commit hooks to run the checks on each commit
```bash
pre-commit install
```

## Step 7: Add Another Commit and See Pre-Commit in Action

```bash
echo 'Additional project details' >> README.md  # Modify the README file again
git add README.md                              # Stage the changes
git commit -m "Add more details to README"       # Commit the changes and see pre-commit hooks in action
```

**Explanation**: In this step, we modify the README file again and commit the changes. When running `git commit`, the pre-commit hooks will automatically execute, checking for issues like trailing whitespace or formatting errors. If any issues are detected, they must be fixed before the commit can proceed.

#### Add secrets scanner to pre-commit
```bash
pip install yelp 
```
#### Add the section below to .pre-commit-config.yaml
```bash
-   repo: https://github.com/Yelp/detect-secrets
    rev: v0.13.0
    hooks:
    -   id: detect-secrets
        exclude: .*/tests/.*
```
## Step 7: Add Another Commit and See Pre-Commit in Action

```bash
echo 'Additional project details' >> README.md  # Modify the README file again
git add README.md                              # Stage the changes
git commit -m "Add more details to README"       # Commit the changes and see pre-commit hooks in action
```

**Explanation**: In this step, we modify the README file again and commit the changes. When running `git commit`, the pre-commit hooks will automatically execute, checking for issues like trailing whitespace or formatting errors. If any issues are detected, they must be fixed before the commit can proceed.
