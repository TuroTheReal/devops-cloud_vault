# Git/GitHub - Commands Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, git, github]
created: 2025-12-30
version: 2.43.0
```

**Official docs**: [Git Documentation](https://git-scm.com/doc) | [GitHub Docs](https://docs.github.com)
**Related concepts**: [[git-github-fundamentals]]

---

## 📚 Table of Contents

1. [Daily Commands](#daily-commands)
2. [Branch Management](#branch-management)
3. [Inspection](#inspection)
4. [Rebase & Force Push](#rebase--force-push)
5. [Correction](#correction)
6. [Rescue Commands](#rescue-commands)
7. [Common Workflows](#common-workflows)

---

## Daily Commands

### Basic Operations

| Command | Action | Usage |
|---------|--------|-------|
| `git status` | Show working tree status | Check before commit |
| `git add .` | Stage all files | Prepare commit |
| `git add <file>` | Stage specific file | Partial commit |
| `git commit -m "message"` | Create commit | Save changes |
| `git push origin <branch>` | Push to GitHub | Share changes |
| `git pull origin main` | Update main | Sync main branch |
| `git pull --rebase origin main` | Update with rebase | Sync feature branch |

### Commit Message Format
```bash
# Format: <type>: <description>
git commit -m "feat: add user authentication"
git commit -m "fix: resolve login bug"
git commit -m "docs: update README"
git commit -m "refacto: improve error handling"
```

---

## Branch Management

### Navigation

| Command | Action | Usage |
|---------|--------|-------|
| `git branch` | List local branches | See current branch |
| `git branch -a` | List all branches | Local + remote |
| `git checkout main` | Switch to main | Navigation |
| `git checkout -b feature/new` | Create and switch | New feature |
| `git branch -d feature/old` | Delete branch (safe) | Cleanup merged branch |
| `git branch -D feature/old` | Force delete branch | Delete unmerged branch |

### Remote Branches
```bash
# List remote branches
git branch -r

# Delete remote branch
git push origin --delete feature/old

# Track remote branch
git checkout -b feature/monitoring origin/feature/monitoring
```

---

## Inspection

### Viewing Changes

| Command | Action | Usage |
|---------|--------|-------|
| `git fetch origin` | Download without merging | Safe check |
| `git log --oneline` | Short history | Quick view |
| `git log origin/main` | Remote history | Check GitHub |
| `git diff` | Show unstaged changes | Before add |
| `git diff --staged` | Show staged changes | Before commit |
| `git diff main origin/main` | Compare local vs remote | Sync check |

### Log Options
```bash
# Last 5 commits
git log --oneline -5

# Graph view
git log --oneline --graph --all

# Commits by author
git log --author="Your Name"

# Commits in date range
git log --since="2 weeks ago"
```

---

## Rebase & Force Push

### Rebase Commands

| Command | Action | Usage |
|---------|--------|-------|
| `git rebase origin/main` | Rebase on main | Update feature |
| `git rebase --continue` | Continue after conflict | Resolve conflicts |
| `git rebase --abort` | Cancel rebase | Emergency exit |
| `git push --force-with-lease origin <branch>` | Safe force push | After rebase |

### Rebase Workflow
```bash
# Update feature branch with latest main
git checkout main
git pull origin main
git checkout feature/monitoring
git rebase origin/main

# If conflicts occur
# 1. Fix conflicts in files
# 2. Stage resolved files
git add <resolved-file>
# 3. Continue rebase
git rebase --continue

# Push rebased branch
git push --force-with-lease origin feature/monitoring
```

**⚠️ IMPORTANT**: Never rebase shared branches (`main`, `develop`, `staging`)

---

## Correction

### Fixing Mistakes

| Command | Action | Danger | Usage |
|---------|--------|--------|-------|
| `git reset HEAD <file>` | Unstage file | ✅ Safe | Undo add |
| `git reset --soft HEAD~1` | Undo commit (keep changes) | ✅ Safe | Redo commit |
| `git reset --hard HEAD~1` | Delete commit | ❌ DANGER | Really undo everything |
| `git restore <file>` | Discard changes | ⚠️ Warning | Revert to last commit |

### Amending Commits
```bash
# Fix last commit message
git commit --amend -m "new message"

# Add forgotten files to last commit
git add forgotten-file.js
git commit --amend --no-edit

# ⚠️ Only amend unpushed commits!
```

---

## Rescue Commands

### Git Stash - Temporary Save

**Situation**: Need to switch branches but have uncommitted changes

```bash
# Save current work
git stash
# OR with message
git stash save "WIP: user authentication"

# Switch branches and work
git checkout main
# ... do other work ...

# Return and restore
git checkout feature/auth
git stash pop  # Restore and delete stash

# List stashes
git stash list

# Apply specific stash
git stash apply stash@{0}

# Delete stash
git stash drop stash@{0}
git stash clear  # Delete all stashes
```

---

### Git Reflog - Emergency Recovery

**Situation**: Made a mistake and want to undo

```bash
# View complete history of HEAD moves
git reflog

# Output example:
# 3a2b1c0 HEAD@{0}: commit: feat: monitoring
# 5d4e3f2 HEAD@{1}: rebase: checkout origin/main
# 7f6g5h4 HEAD@{2}: commit: fix bug

# Restore to previous state
git reset --hard HEAD@{2}
```

**Example: Recover deleted commits**
```bash
# Accidentally deleted commits
git reset --hard HEAD~3  # ❌ 3 commits lost

# Recovery
git reflog  # Find commit before deletion
git reset --hard HEAD@{5}  # ✅ Recovered
```

---

## Common Workflows

### Workflow 1: Feature with Pull Request (Recommended)

```bash
# === SETUP (once) ===
git checkout main
git pull origin main
git checkout -b feature/monitoring

# === MORNING (daily update) ===
git checkout main
git pull origin main
git checkout feature/monitoring
git rebase origin/main
git push --force-with-lease origin feature/monitoring

# === WORK (multiple times per day) ===
git add .
git commit -m "feat: add prometheus metrics"
git push origin feature/monitoring

# === FINISH FEATURE ===
# 1. Final push
git push origin feature/monitoring

# 2. On GitHub
# - Create Pull Request
# - feature/monitoring → main
# - Review + Merge

# 3. Cleanup
git checkout main
git pull origin main
git branch -d feature/monitoring
```

---

### Workflow 2: Hotfix

```bash
# From main
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# Quick fix
git add .
git commit -m "fix: critical security issue"
git push origin hotfix/critical-bug

# Fast PR or direct merge
# Then cleanup
git checkout main
git pull origin main
git branch -d hotfix/critical-bug
```

---

### Workflow 3: Local Feature (No PR)

```bash
# === SETUP ===
git checkout main
git pull origin main
git checkout -b feature/local-experiment

# === WORK ===
git add .
git commit -m "feat: something"
# No push if staying local

# === FINISH FEATURE ===
# 1. Update main
git checkout main
git pull origin main

# 2. Merge locally
git merge feature/local-experiment

# 3. Push to GitHub
git push origin main

# 4. Cleanup
git branch -d feature/local-experiment
```

---

## Quick Reference

### Daily Workflow

```bash
# Morning
git checkout main
git pull origin main
git checkout feature/my-work
git rebase origin/main
git push --force-with-lease origin feature/my-work

# Work (repeat as needed)
git add .
git commit -m "feat: description"
git push origin feature/my-work

# End of feature
# → GitHub: Create PR → Merge
git checkout main
git pull origin main
git branch -d feature/my-work
```

---

### Pre-Action Checklists

**Before commit:**
```bash
git status  # Check what's staged
git diff    # Review changes
git add .
git commit -m "message"
```

**Before push:**
```bash
git status  # Everything committed?
git push origin <your-branch>
```

**Before rebase:**
```bash
git status  # Working directory clean?
git fetch origin
git rebase origin/main
```

---

### Command Frequency

**Every day:**
```bash
git status
git add .
git commit -m "message"
git push origin <branch>
git pull origin main
git checkout <branch>
```

**Several times per week:**
```bash
git fetch origin
git rebase origin/main
git push --force-with-lease origin <branch>
git stash / git stash pop
git branch -d <branch>
```

**Occasionally:**
```bash
git reflog
git reset --soft HEAD~1
git merge <branch>
```

---

## Concepts Summary

### Branch Notation

| You write | Location | Description |
|-----------|----------|-------------|
| `main` | Local | Your main branch on PC |
| `origin/main` | Remote | Main branch on GitHub |
| `feature/monitoring` | Local | Your local feature |
| `origin/feature/monitoring` | Remote | Feature on GitHub |

**Rule**: `origin` = GitHub (almost always)

---

### Fetch vs Pull

| Command | Downloads | Modifies Files | Usage |
|---------|-----------|----------------|-------|
| `git fetch origin` | ✅ | ❌ | Safe check |
| `git pull origin main` | ✅ | ✅ | Update main |
| `git pull --rebase origin main` | ✅ | ✅ | Update feature (linear) |

---

### Rebase Principle

**Rebase = Move your branch to the end of another**

```
Before:
main:    A---B---E---F
          \
feature:   C---D

git rebase origin/main

After:
main:    A---B---E---F
                    \
feature:             C'---D'
```

**Syntax:**
```bash
git checkout <branch-to-move>
git rebase <destination>
```

---

### Force Push

**ONLY needed after rebase**

```bash
# ✅ Safe (checks if remote changed)
git push --force-with-lease origin feature

# ⚠️ Dangerous (overwrites without checking)
git push --force origin feature
```

**When to use:**
- ✅ After rebasing feature branch
- ❌ Regular commits (use normal push)
- ❌ Never on main/shared branches

---

## 💡 Personal Notes

**Mistakes made**:
- **Force pushed to main** → Fix: Never force push shared branches, use `--force-with-lease` only on feature branches
- **Committed secrets (.env)** → Fix: Always add `.env`, `credentials.*`, `*.pem` to `.gitignore` before first commit
- **Lost commits in detached HEAD** → Fix: Always create branch before committing, use `git reflog` to recover

**Daily workflow** (commands I actually use):
```bash
# Morning routine
git checkout main && git pull origin main
git checkout feature/my-work && git rebase origin/main
git push --force-with-lease origin feature/my-work

# During work
git add . && git commit -m "feat: description" && git push origin feature/my-work

# Emergency stash
git stash && git checkout main  # switch context
git checkout feature/my-work && git stash pop  # back to work
```

---

**Related**: [[git-github-fundamentals]]
**Last update**: 2025-12-30
**Tool version**: 2.43.0
