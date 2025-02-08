### Git Rebase and Squash Merge

Git provides several ways to integrate changes from one branch to another. Two of the most commonly used strategies are **rebase** and **squash merge**. 

#### 1. Git Rebase

##### Importance
Rebasing is a powerful tool for streamlining a feature branch by moving or “replaying” your work on top of another branch, typically the `main` branch. Instead of creating a merge commit, rebasing applies each commit in sequence, maintaining a cleaner history.

##### How Git Rebase Works
Consider a situation where your feature branch is based on an earlier commit in the `develop` branch, and the `develop` branch has since received new commits.

```bash
# Feature branch history
o---o---o (feature)
     \
      o---o---o (develop)
```

If you use `git merge` to merge `develop` into `feature`, Git will create a merge commit:

```bash
# After git merge develop into feature
o---o---o---M (feature)
     \     /
      o---o---o (develop)
```

Instead, `git rebase` will replay your changes on top of the latest commits in `develop`:

```bash
# After git rebase develop
o---o---o (develop)
           \
            o---o---o (feature)
```

##### Command for Git Rebase
```bash
git checkout feature
git rebase develop
```

##### Advantages of Git Rebase
- **Clean History**: Rebasing results in a clean and linear commit history, as there are no merge commits to complicate the history.
- **Easier to Understand**: A rebased history looks like a sequence of well-organized commits, which makes it easier to review changes.
- **Conflict Resolution**: You only resolve conflicts once (during the rebase process), rather than at each merge point.

##### Disadvantages of Git Rebase
- **Rewriting History**: Rebasing rewrites commit history, which can lead to problems if the feature branch is shared with others. If others are working on the same branch, rebasing will result in **diverged history**, causing confusion or conflicts.

##### Example of Git Rebase
Consider you have two branches, `feature` and `develop`. The `develop` branch has new commits while you're working on your `feature` branch. To incorporate the latest changes from `develop` into your feature branch and reapply your commits on top of them, you can:

```bash
# Step 1: Switch to your feature branch
git checkout feature

# Step 2: Rebase the feature branch onto develop
git rebase develop

# Step 3: Push the changes (force push may be needed if the branch was already pushed)
git push --force
```

---

#### 2. Git Squash Merge

##### Importance
Squash merge consolidates multiple commits into a single commit before merging them into the develop branch. This is especially useful when a feature branch has numerous small commits that don’t need to be tracked individually in the develop history.

##### How Git Squash Merge Works
Squashing takes a sequence of commits in your branch and combines them into a single commit, making the history cleaner and easier to read. After squashing, Git performs the merge into the target branch.

```bash
# Before squash merge
o---o---o (develop)
           \
            o---o---o---o (feature)

# After squash merge
o---o---o---O (develop)
```

##### Command for Git Squash Merge
1. **Interactive Rebase to Squash Commits** (prior to merge)
    ```bash
    git rebase -i HEAD~n
    ```
    Replace `n` with the number of commits you want to squash (e.g., `HEAD~3` for the last 3 commits). In the interactive editor, change all but the first commit's `pick` to `squash` or `s`.

2. **Squash During Merge**:
    ```bash
    git checkout develop
    git merge --squash feature
    git commit
    ```

##### Advantages of Git Squash Merge
- **Simplified History**: Combines all commits into one, which helps when many small commits aren’t valuable to keep in the develop history.
- **Improved Readability**: Reviewers can focus on the overall change rather than every individual commit.
- **Organized**: Squash merges are ideal for repositories where commits should be logical units of work rather than a running log of every minor change.

##### Disadvantages of Git Squash Merge
- **Loss of Granular History**: If individual commits had meaningful steps, they will be lost in the squash, as only a single commit is created.
- **Difficult to Trace Changes**: If debugging an issue, tracing the exact point where a problem was introduced can be harder since the commits are combined.

##### Example of Git Squash Merge
Consider you’ve made several commits in your `feature` branch that you want to squash into one commit and merge into `develop`:

```bash
# Step 1: Checkout develop branch
git checkout develop

# Step 2: Perform a squash merge
git merge --squash feature

# Step 3: Commit the squashed changes
git commit -m "Merged feature with squashed commits"

# Step 4: Push to the remote repository
git push
```
---

### Tool

#### Visual Studio Code (VS Code)

##### Setup

- Install GitLens - Git supercharged
- Open Command Palette and type in **GitLens: Enable Interactive Rebase Editor** and select it to enable rebase editor
     ![image](https://github.com/user-attachments/assets/f1e41573-e25b-4bb4-91a7-a62120bda1af)
- Open a terminal and execute below command to launch rebase editor (like VS Code in our case) when we use rebase command
```bash
git config --global core.editor "code --wait"
```

##### Rebase & Squash

- Create a feature branch for your local development and have some commits in that feature branch
- Run below command to perfrom a rebase with develop branch (make sure your local develop branch has latest code - pull latest from origin develop) and it will apply your feature branch changes on top of the latest commits in `develop`. You may need to resolve any merge conflit that comes up.
```bash
git rebase -i origin/develop 
```
- It will prompt below GitLens Interactibe Rebase UI that can also allow to perform squash merge

![image](https://github.com/user-attachments/assets/f143fa5a-fc03-47b1-a0c1-ea021bd95a82)

-  Keep one commit as pick and choose rest as squash from dropdown

![image](https://github.com/user-attachments/assets/857a7ba3-1a56-48e4-a9d1-c9e11a390476)

- Then click on Start Rebase to apply rebase

![image](https://github.com/user-attachments/assets/511a911e-8ef6-46e4-848c-d88df8705d76)

- After rebase, it will prompt for commit message that you can modify as shown below

![image](https://github.com/user-attachments/assets/c7d60c89-c971-4ff2-bd2e-4b9aeab11d7f)

- You can modify commit message as per your need, save and close that COMMIT_EDITMSG window

![image](https://github.com/user-attachments/assets/e9930681-fee5-4d92-90e4-d444456df12e)

![image](https://github.com/user-attachments/assets/4bce29c2-aabc-40a4-8a25-28cb7c0e9867)

- Now run below command to push your squash commit changes to origin

```bash
git push -f
```
![image](https://github.com/user-attachments/assets/89b3ad03-4684-4bf1-8911-5d9c9e6ba4f9)

![image](https://github.com/user-attachments/assets/d37c478f-1acf-41e7-b6c7-0adb51e33ce3)



