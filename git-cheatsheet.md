# Git cheatsheet

## 1. Global configurations
| Command | Description |
|---------|-------------|
| `git config --global user.name "example"`         | Set user name |
| `git config --global user.email "x@example.com"`  | Set user email |
| `git config --global pull.rebase true`            | Default to rebasing instead of merging when running `git pull`, keeping branch histories linear and reducing merge conflicts. |
| `git config --global branch.sort -committerdate`  | Set default branch sorting to show most recently committed branches first when running `git branch`, aiding in identifying stale branches. |
| `git config --global alias.last 'log -1 HEAD'`    | Get last commit `git last`

## 2. Branches
| Command | Description |
|---------|-------------|
| `git <branch>`                                | List local branches; current branch marked with *. `-r`: remote-tracking branches (e.g., origin/main), `-a` - all branches(local + remote) <br> `-vv` to see tracking info and last commits
| `git branch <branch>`                         | Create a new branch |
| `git branch <branch> <start-point>`           | Create a branch from a specific commit, tag, or branch (e.g., `git branch feature-x origin/main`). |
| `git checkout <branch>`                       | Switch to a branch  <br> `git switch branch`|
| `git checkout -b <branch>`                    | Create and switch in one command <br> `git switch -c branch` |
| `git branch -d <branch>`                      | Delete a fully merged branch (safe).
| `git branch -D <branch>`                      | Force delete an unmerged branch (use with caution).
| `git branch --set-upstream-to=origin/<branch>` | Set a local branch to track a remote branch. <br> `git branch -u origin/<branch>`

### Pro tips
| Description | Command |
| - | - |
| Sort branches by last commit date to identify stale branches. | `git branch --sort=-committerdate` |
| Clean up multiple stale (merged) branches. | `git branch --merged main \| grep -v "\* main" \| xargs -n 1 git branch -d` |
| Delete multiple branches matching a pattern  | `git branch \| grep "<pattern>" \| xargs git branch -D` <br> (e.g., `git branch \| grep "feature/.*" \| xargs git branch -D` to delete all `feature/*` branches).|
| List branches not merged into a specific branch (e.g., `main`) to identify work in progress. | `git branch --no-merged main` |
| Check which branches contain a specific commit (useful for tracking bug introductions). | `git branch --contains <commit-sha>` |
| Rename a branch and sync with remote by pushing the new name and deleting the old one. | `git branch -m <old-name> <new-name>` && <br> `git push origin <new-name>` && <br> `git push origin --delete <old-name>` |

### Good practices

| Practice | Description |
|----------|-------------|
| Sync branches constantly | Regularly run `git fetch --all --prune` to update local tracking branches and remove stale remote references. Combine with `git pull --rebase` on active branches to keep them in sync with `origin/main`, minimizing merge conflicts in fast-moving teams. |
| Set tracking branches explicitly when needed to ensure smooth push/pull | `git branch --set-upstream-to=origin/<branch> <local-branch>` or <br> `git push --set-upstream origin <branch>` or <br> `git branch -u origin/<branch>` |
| Automate branch cleanup in CI/CD | Integrate `git branch --merged origin/main \| grep -v "\* main" \| xargs -n 1 git branch -d` into your CI pipeline to automatically prune merged branches after successful builds, keeping the repository lean. |
| Use branch naming conventions with automation | Enforce prefixes like `feature/`, `bugfix/`, or `release/` in team workflows. Script validation with `git branch \| grep -vE "^(feature\|bugfix\|release)/" \| xargs -I {} echo "Invalid branch name: {}"` to flag non-compliant branches. |
| Track branch staleness with commit age | Combine `git branch --sort=-committerdate` with `git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) last commit: %(authordate)'` to generate a report of branches and their last commit dates, helping identify abandoned work. |
| Safe branch deletion with dry-run | Before deleting branches with `git branch \| grep "<pattern>" \| xargs git branch -D`, preview the list with `git branch \| grep "<pattern>"` to avoid accidental deletions. |
| Recover deleted branches efficiently | Use `git reflog \| grep <branch>` to find the SHA of a deleted branch, then recreate it with `git branch <branch> <commit-sha>`. Set `gc.reflogExpire=90d` in `.gitconfig` to extend reflog retention for safer recovery. |

## 3. Commits
| Command | Description |
|---------|-------------|
| `git commit -m '<message>'` | Commit with an inline message (e.g., `git commit -m "Add user authentication endpoint"`). |
| `git commit -a` | Stage all tracked, modified files and commit in one step. Alias: `--all`. |
| `git commit --amend` | Modify the most recent commit by adding staged changes or editing the message. Use `--no-edit` to keep the original message. |
| `git commit -C <commit-sha>` | Reuse the message from a previous commit (e.g., `git commit -C abc123`). Use `--no-edit` with `--amend` for similar effect. |
| `git commit --no-verify` | Bypass pre-commit hooks (e.g., linters, tests) for quick commits. Use sparingly to avoid breaking CI/CD pipelines. |

### Pro tips
| Description | Command |
| - | - |
| Add all files and commit  | `git commit -am '<message>'` |
| Add changes to the previous commit without altering message | `git commit --amend --no-edit` |
| Trigger CI/CD without commit message or making milestones   | `git commit -m '<message>' --allow-empty` <br> or `git commit -C <commit-sha>` |

## 4. Stash
| Command | Description |
|---------|-------------|
| `git stash`       | Stash current changes (staged and unstaged) <br> `git stash push`. <br>`-u(--include-untracked)` - including untracked files <br> `-a(--all)` - including untracked and ignored files. |
| `git stash -m '<message>'` | Stash changes with a custom message <br> (e.g., `git stash -m "WIP: feature-x UI changes"`). |
| `git stash list` | List all stashes with their indices and messages |
| `git stash apply` | Apply the most recent stash (`stash@{0}`) without removing it from the stash list. <br>Specify a stash with `git stash apply stash@{n}`. |
| `git stash pop` | Apply the most recent stash and remove it from the stash list. <br>Specify a stash with `git stash pop stash@{n}`. |
| `git stash show` | Show changes in the most recent stash. <br>Use `git stash show stash@{n}` for a specific stash; add `-p` for a full diff. |
| `git stash drop` | Remove the most recent stash from the list. <br>Specify a stash with `git stash drop stash@{n}`. |
| `git stash clear` | Remove all stashes from the stash list (use with caution). |
| `git stash branch <branch>` | Create and switch to a new branch from a stash, applying the stash’s changes (e.g., `git stash branch feature-x stash@{0}`). Useful for saving incomplete work to a new branch. |

#### Good practices
| Description | Command |
|-------------|---------|
|Use descriptive messages for easier identification in long-running projects. | `git stash -m '<message>'` |
| Inspect stash changes before applying to prevent accidental overwrites in complex workflows. | `git stash show -p stash@{n}` |

## 5. Remote
| Command | Description |
|---------|-------------|
| `git remote` | List all configured remotes (e.g., `origin`, `upstream`). |
| `git remote -v` | List remotes with their fetch and push URLs for detailed inspection. |
| `git remote add <name> <url>` | Add a new remote repository. |
| `git remote remove <name>` | Remove a remote from the repository. |
| `git remote rename <old-name> <new-name>` | Rename a remote. |
| `git remote set-url <name> <new-url>` | Update the URL for a remote. |
| `git remote show <name>` | Display detailed information about a remote, including tracked branches and fetch/push configurations. |

### Pro Tips
| Description | Command |
|-------------|---------|
| Optimize fetch performance in large repositories with shallow fetches. | `git fetch <remote> --depth 1` |
| Set up tracking for a local branch to simplify push/pull operations. | `git push --set-upstream <remote> <branch>` or `git branch --set-upstream-to=<remote>/<branch>` |

## Git branching
https://www.youtube.com/watch?v=0chZFIZLR_0

## 5. Rebase

Rebasing applies the feature branch's commits on top of the main branch, creating a linear history.

### Flow
Step 1: Switch to the feature branch & Rebase the feature branch onto main
```sh
git checkout feature
git rebase main

# Shorthand
shgit rebase main feature
```

Step 2: Switch to the main branch & Merge the rebased feature branch into main
```sh
git checkout main
git merge feature

# Shorthand
shgit merge feature main
```


## 6. Merge
Merging combines the feature branch into the main branch, preserving the branch's history and potentially creating a merge commit.

### Flow

Step 1: Switch to the feature branch & Rebase the feature branch onto main
```sh
git checkout feature
git merge main

# Shorthand
git merge main feature
```

Step 2: Switch to the main branch & Merge the rebased feature branch into main
```sh
git checkout main
git merge feature

# Shorthand
git merge feature main
```


## 7. Cherry Picking


## Others
### `git fetch` vs `git merge`
https://stackoverflow.com/questions/292357/what-is-the-difference-between-git-pull-and-git-fetch

| Feature             | Git Fetch        | Git Pull       |
|---------------------|-----------------|-----------------|
| **Purpose**         | Gets remote changes without merging          | Gets and merges remote changes              |
| **Action**          | Updates remote-tracking branches (e.g., `origin/main`) | Fetches and merges into current branch      |
| **Effect on Local** | No changes to local files or branches        | Updates local files and branch              |
| **Command Example** | `git fetch origin main`                     | `git pull origin main`                      |
| **Safety**          | Safe, lets you review changes first          | May cause conflicts if local changes exist  |
| **Use Case**        | Check remote updates before merging          | Quick sync with remote branch               |
| **Equivalent To**   | Just fetching                               | `git fetch` + `git merge`                  |

**Explanation**:
- `git pull` tries to automatically merge after fetching commits. It is context sensitive, so all pulled commits will be merged into your currently active branch. git pull automatically merges the commits without letting you review them first. If you don’t carefully manage your branches, you may run into frequent conflicts.

- `git fetch` gathers any commits from the target branch that do not exist in the current branch and stores them in your local repository. However, it does not merge them with your current branch. This is particularly useful if you need to keep your repository up to date, but are working on something that might break if you update your files. To integrate the commits into your current branch, you must use git merge afterwards.
  - Run `git diff ...origin` to see differences before merging.