# Git cheatsheet

## Branches
| Command | Description |
| - | - |
| `git <branch>`                                | List local branches; current branch marked with *. `-r`: remote-tracking branches (e.g., origin/main), `-a` - all branches(local + remote) <br> `-vv` to see tracking info and last commits
| `git branch <branch>`                         | Create a new branch |
| `git branch <branch> <start-point>`           | Create a branch from a specific commit, tag, or branch (e.g., `git branch feature-x origin/main`). |
| `git checkout <branch>`                            | Switch to a branch  <br> `git switch branch`|
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
| Set tracking branches explicitly when needed to ensure smooth push/pull | `git branch --set-upstream-to=origin/<branch> <local-branch>` or <br> `git push --set-upstream origin <branch>` or <br> `git branch -u origin/<branch-name>` |
| Automate branch cleanup in CI/CD | Integrate `git branch --merged origin/main \| grep -v "\* main" \| xargs -n 1 git branch -d` into your CI pipeline to automatically prune merged branches after successful builds, keeping the repository lean. |
| Use branch naming conventions with automation | Enforce prefixes like `feature/`, `bugfix/`, or `release/` in team workflows. Script validation with `git branch \| grep -vE "^(feature\|bugfix\|release)/" \| xargs -I {} echo "Invalid branch name: {}"` to flag non-compliant branches. |
| Track branch staleness with commit age | Combine `git branch --sort=-committerdate` with `git for-each-ref --sort=-committerdate refs/heads/ --format='%(refname:short) last commit: %(authordate)'` to generate a report of branches and their last commit dates, helping identify abandoned work. |
| Safe branch deletion with dry-run | Before deleting branches with `git branch \| grep "<pattern>" \| xargs git branch -D`, preview the list with `git branch \| grep "<pattern>"` to avoid accidental deletions. |
| Recover deleted branches efficiently | Use `git reflog \| grep <branch-name>` to find the SHA of a deleted branch, then recreate it with `git branch <branch-name> <commit-sha>`. Set `gc.reflogExpire=90d` in `.gitconfig` to extend reflog retention for safer recovery. |

