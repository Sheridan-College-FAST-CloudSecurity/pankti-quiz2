# Working with Remote Repositories in GitHub

In this lab, you’ll get hands-on experience using Git and GitHub for version control. Follow each step carefully and refer to the comments to understand each Git command.

---

## Prerequisites

- A local Git repository already set up with a main branch and a branch named `feature_fix`.

---

### Step 1: Sign up for a Free GitHub Account

1. Visit [GitHub](https://github.com) and click **Sign up**.
2. Follow the prompts to create a free account.

---

### Step 2: Create a New Public Repository

1. Once signed in, go to the GitHub homepage and click **New** to create a new repository.
2. Name your repository (e.g., `lab3`) and set it to **Public**.
3. Do **not** initialize with a README, `.gitignore`, or license (we'll add files from your local repo instead).
4. Click **Create repository**.

---

## Step 3: Add Remote Repository

```bash
git remote add origin <https://github.com/your-username/lab3.git>
```

This command adds a remote repository as `origin`. `origin` is the default name used in Git to refer to the main repository.

## Confirm the Remote Repository

Confirm the remote has been added by running:

```bash
git remote -v
```

This command lists all remote connections for your repository.

## Step 4: Push the Content of the "feature_fix" Branch to the Remote Repository

Push your `feature_fix` branch to the new GitHub repository:

```bash
git push -u origin feature_fix
```

This command pushes the `feature_fix` branch to the `origin` repository on GitHub. The `-u` flag sets up tracking so that future pushes and pulls on this branch are associated with the `origin` repository.

## Step 5: Open a Pull Request (PR) to Merge Changes into the Main Branch

On GitHub, navigate to your repository and you should see a notification suggesting to create a pull request for `feature_fix`.

Click **Compare & pull request**.

Verify that the base branch is `main` and the compare branch is `feature_fix`.

Add a title (e.g., "Feature Fix Updates") and description, then click **Create pull request**.

**Pull Request**: A PR is a request to merge changes from one branch (e.g., `feature_fix`) into another branch (e.g., `main`). It allows other team members to review the changes before merging.

## Step 6: Merge Pull Request into the Main Branch

On the PR page, click **Merge pull request**.

Confirm by clicking **Confirm merge**.

Once merged, you can delete the `feature_fix` branch from the GitHub repository by clicking **Delete branch**.

**Merge**: Merging applies all changes from the PR branch into the target branch (e.g., from `feature_fix` into `main`), updating `main` with the new code.
