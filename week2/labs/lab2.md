# Lab 2: Git Branches, Merges, Understanding Git HEAD and detached HEAD

- **Step 1: Examine the current position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all  # View the current commit history and position of HEAD
  ```

  *Explanation*: The `--decorate` option shows where HEAD is pointing.

- **Step 2: Make changes to a file**

  ```bash
  echo 'Adding new line to demonstrate HEAD' >> file.txt  # Make changes to a tracked file
  ```

- **Step 3: Re-examine the current position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all
  ```

- **Step 4: Add the changed file to the staging area**

  ```bash
  git add file.txt  # Stage the modified file
  ```

- **Step 5: Re-examine the current position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all
  ```

- **Step 6: Commit the changes**

  ```bash
  git commit -m "Updated file with new changes"
  ```

- **Step 7: Examine the current position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all
  ```

*Explanation*: These steps illustrate how the HEAD pointer moves throughout the Git workflow—from making changes to staging and committing them. Each step lets you observe how HEAD changes position, which is fundamental in understanding branching and navigating Git.

- **Step 8: Create a New Branch**

  ```bash
  git branch new-feature  # Create a new branch called 'new-feature'
  ```

  *Explanation*: This command creates a new branch named 'new-feature'. Branching allows you to work on new features without affecting the main branch.

- **Step 9: Move HEAD to the New Branch**

  ```bash
  git checkout new-feature  # Switch to the new branch and move HEAD to it
  ```

  *Explanation*: The `git checkout` command is used to switch branches. After running this command, HEAD will point to the latest commit in the 'new-feature' branch.

- **Step 10: Verify the Current Branch**

  ```bash
  git branch  # List all branches and verify which one is currently checked out
  ```

  *Explanation*: This command lists all branches and highlights the current branch with an asterisk (\*). This helps confirm that HEAD has moved to the new branch.

- **Step 11: Modify file.txt on the 'new-feature' Branch**

  ```bash
  echo 'Adding a new feature line' >> file.txt  # Modify the file on the new-feature branch
  ```

  *Explanation*: Add a new line to 'file.txt' while on the 'new-feature' branch. This modification is specific to the branch and will be committed in the following steps.

- **Step 12: Stage the Modified File**

  ```bash
  git add file.txt  # Stage the modified file for commit
  ```

  *Explanation*: The `git add` command stages the changes, preparing them to be committed.

- **Step 13: Commit the Changes**

  ```bash
  git commit -m "Added a new feature line to file.txt"
  ```

  *Explanation*: The `git commit` command saves the staged changes in the 'new-feature' branch, adding a commit with a descriptive message.

- **Step 14: Examine the Current Position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all
  ```

  *Explanation*: This command will help you verify that HEAD is pointing to the latest commit in the 'new-feature' branch.

-

- **Step 15: Switch to the Main Branch**

```bash
git checkout main  # Switch back to the main branch
```

*Explanation*: Before merging, you need to switch to the main branch where you want to integrate the changes from 'new-feature'.

- **Step 16: Merge the 'new-feature' Branch into Main**

  ```bash
  git merge new-feature  # Merge the changes from 'new-feature' into 'main'
  ```

  *Explanation*: This command merges the changes from the 'new-feature' branch into the 'main' branch. If there are no conflicts, the merge will proceed automatically.

- **Step 17: Examine the Current Position of the HEAD**

  ```bash
  git log --oneline --decorate --graph --all
  ```

  *Explanation*: After merging, use this command to verify that the latest commit in the main branch includes the changes from 'new-feature'.
  - **Visual Representation**:
  ![Feature-fix branch merged to the main branch](branch_merged_to_main.png)

-

- **Step 18: Demonstrate a Detached HEAD Situation**

```bash
git checkout <commit-hash>  # Checkout a specific commit to create a detached HEAD
```

*Explanation*: When you checkout a specific commit by its hash, HEAD is now pointing directly to that commit rather than a branch. This means you're in a 'detached HEAD' state.

- **Step 19: Understand the Problem with Detached HEAD**

  *Explanation*: In a detached HEAD state, any new commits will not be associated with any branch. If you create new commits and then switch to another branch, those commits may be lost unless explicitly referenced or moved to a branch. This situation poses a risk of losing work if not handled carefully.

    - - **Step 20: Create a Few Commits in Detached HEAD State**

  ```bash
  echo 'Detached commit 1' >> file.txt  # Modify file in detached HEAD state
  git add file.txt  # Stage the changes
  git commit -m "Detached commit 1"  # Commit the changes

  echo 'Detached commit 2' >> file.txt  # Modify file again in detached HEAD state
  git add file.txt  # Stage the changes
  git commit -m "Detached commit 2"  # Commit the changes
  ```

  *Explanation*: These commands create two commits while in a detached HEAD state. Note that these commits are not associated with any branch.

- **Step 21: Switch to the 'feature-fix' Branch**

  ```bash
  git checkout feature-fix  # Switch to an existing branch named 'feature-fix'
  ```

  *Explanation*: Switching to the 'feature-fix' branch moves the HEAD pointer to the latest commit in that branch. The commits made in the detached HEAD state are not linked to this or any other branch.

- **Step 22: Demonstrate that the Detached HEAD Commits Have Been Lost**

  ```bash
  git log --oneline --decorate --graph --all
  ```

  *Explanation*: Viewing the commit history with `git log` will show that the commits made in the detached HEAD state are not part of the current branch. These commits may be lost unless they are explicitly referenced or added to a branch.

- **Visual Representation**:
  ![Detached HEAD Commit Loss](detached_head_loss.png)

  *Explanation*: The image illustrates the detached HEAD state and how commits made in this state are not linked to any branch. This can lead to losing track of these commits if they are not appropriately managed.
