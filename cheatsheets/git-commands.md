# Git Commands Cheatsheet

## Configuration

```bash
# Set user name and email
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# View configuration
git config --list
git config user.name

# Set default editor
git config --global core.editor "code --wait"

# Set default branch name
git config --global init.defaultBranch main
```

## Repository Setup

```bash
# Initialize new repository
git init

# Clone existing repository
git clone <url>
git clone <url> <folder-name>
git clone --branch <branch> <url>    # Clone specific branch

# Add remote
git remote add origin <url>
git remote -v                         # View remotes
git remote remove origin              # Remove remote
```

## Basic Workflow

```bash
# Check status
git status
git status -s                         # Short format

# Stage changes
git add <file>
git add .                             # Stage all
git add -A                            # Stage all including deletions
git add -p                            # Interactive staging

# Commit
git commit -m "message"
git commit -am "message"              # Add and commit tracked files
git commit --amend                    # Modify last commit
git commit --amend --no-edit          # Amend without changing message

# Push
git push
git push origin <branch>
git push -u origin <branch>           # Set upstream
git push --force                      # Force push (dangerous!)
git push --force-with-lease           # Safer force push

# Pull
git pull
git pull origin <branch>
git pull --rebase                     # Pull with rebase
```

## Branching

```bash
# List branches
git branch                            # Local branches
git branch -r                         # Remote branches
git branch -a                         # All branches

# Create branch
git branch <branch-name>
git checkout -b <branch-name>         # Create and switch
git switch -c <branch-name>           # Create and switch (newer)

# Switch branch
git checkout <branch>
git switch <branch>                   # Newer command

# Delete branch
git branch -d <branch>                # Safe delete
git branch -D <branch>                # Force delete
git push origin --delete <branch>     # Delete remote branch

# Rename branch
git branch -m <old> <new>
git branch -m <new>                   # Rename current branch
```

## Merging & Rebasing

```bash
# Merge
git merge <branch>
git merge --no-ff <branch>            # Create merge commit
git merge --squash <branch>           # Squash commits

# Rebase
git rebase <branch>
git rebase -i HEAD~3                  # Interactive rebase last 3 commits
git rebase --abort                    # Cancel rebase
git rebase --continue                 # Continue after resolving conflicts

# Cherry-pick
git cherry-pick <commit-hash>
git cherry-pick --no-commit <hash>    # Apply without committing
```

## Viewing History

```bash
# Log
git log
git log --oneline                     # Compact view
git log --graph                       # ASCII graph
git log --all --oneline --graph       # Full history graph
git log -n 5                          # Last 5 commits
git log --author="name"               # By author
git log --since="2024-01-01"          # Date range
git log --grep="bug"                  # Search commit messages
git log -p                            # Show patches
git log -- <file>                     # History of specific file

# Show commit details
git show <commit>
git show HEAD
git show HEAD~2                       # 2 commits before HEAD

# Diff
git diff                              # Unstaged changes
git diff --staged                     # Staged changes
git diff <branch1> <branch2>          # Between branches
git diff <commit1> <commit2>          # Between commits
git diff HEAD~3 HEAD                  # Last 3 commits
```

## Undoing Changes

```bash
# Unstage files
git reset <file>
git restore --staged <file>           # Newer command

# Discard local changes
git checkout -- <file>
git restore <file>                    # Newer command

# Reset commits
git reset --soft HEAD~1               # Keep changes staged
git reset --mixed HEAD~1              # Keep changes unstaged (default)
git reset --hard HEAD~1               # Discard changes

# Revert (create new commit undoing changes)
git revert <commit>
git revert HEAD                       # Revert last commit

# Clean untracked files
git clean -n                          # Dry run
git clean -f                          # Remove untracked files
git clean -fd                         # Remove files and directories
```

## Stashing

```bash
# Stash changes
git stash
git stash save "message"
git stash -u                          # Include untracked files

# List stashes
git stash list

# Apply stash
git stash apply                       # Apply latest, keep in stash
git stash pop                         # Apply latest, remove from stash
git stash apply stash@{2}             # Apply specific stash

# Drop stash
git stash drop stash@{0}
git stash clear                       # Remove all stashes

# Show stash contents
git stash show -p stash@{0}
```

## Tags

```bash
# List tags
git tag
git tag -l "v1.*"                     # Filter tags

# Create tag
git tag v1.0.0                        # Lightweight tag
git tag -a v1.0.0 -m "Release 1.0"   # Annotated tag
git tag -a v1.0.0 <commit>           # Tag specific commit

# Push tags
git push origin v1.0.0               # Push single tag
git push origin --tags               # Push all tags

# Delete tag
git tag -d v1.0.0                    # Local
git push origin --delete v1.0.0      # Remote
```

## Remote Operations

```bash
# Fetch
git fetch                             # Fetch all remotes
git fetch origin                      # Fetch specific remote
git fetch --prune                     # Remove deleted remote branches

# Pull
git pull
git pull --rebase

# Push
git push origin <branch>
git push -u origin <branch>           # Set upstream

# Track remote branch
git branch --set-upstream-to=origin/<branch>
git checkout --track origin/<branch>
```

## Useful Commands

```bash
# Blame (who changed what)
git blame <file>
git blame -L 10,20 <file>            # Lines 10-20

# Bisect (find bug-introducing commit)
git bisect start
git bisect bad                        # Current is bad
git bisect good <commit>              # Known good commit
git bisect reset                      # End bisect

# Reflog (recovery)
git reflog
git checkout HEAD@{2}                # Go to specific state

# Search
git grep "pattern"
git grep -n "pattern"                # With line numbers

# Archive
git archive --format=zip HEAD > repo.zip
```

## Common Workflows

### Feature Branch Workflow
```bash
# Start feature
git checkout -b feature/new-feature main

# Work on feature
git add .
git commit -m "Add new feature"

# Update with main
git fetch origin
git rebase origin/main

# Push and create PR
git push -u origin feature/new-feature
```

### Hotfix Workflow
```bash
# Create hotfix branch
git checkout -b hotfix/bug-fix main

# Fix and commit
git add .
git commit -m "Fix critical bug"

# Merge to main and release
git checkout main
git merge hotfix/bug-fix
git tag -a v1.0.1 -m "Hotfix release"
git push origin main --tags

# Merge back to develop
git checkout develop
git merge hotfix/bug-fix
```

### Undo Last Commit
```bash
# Keep changes
git reset --soft HEAD~1

# Discard changes
git reset --hard HEAD~1

# Already pushed? Create revert commit
git revert HEAD
git push
```

## .gitignore Patterns

```gitignore
# Ignore specific file
secret.txt

# Ignore all .log files
*.log

# Ignore directory
node_modules/
target/

# Ignore all files in directory
logs/*

# Negate (don't ignore)
!important.log

# Ignore in any directory
**/temp/

# Ignore only in root
/config.local.json
```

---

## Quick Reference

| Action | Command |
|--------|---------|
| Initialize | `git init` |
| Clone | `git clone <url>` |
| Stage all | `git add .` |
| Commit | `git commit -m "msg"` |
| Push | `git push` |
| Pull | `git pull` |
| New branch | `git checkout -b <name>` |
| Switch branch | `git checkout <name>` |
| Merge | `git merge <branch>` |
| View history | `git log --oneline` |
| Undo staged | `git reset <file>` |
| Stash | `git stash` |
