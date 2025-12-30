# Git/GitHub Fundamentals

## 📋 Metadata

```yaml
tags: [concept, git, github, version-control, status/learned]
created: 2025-12-30
difficulty: ⭐⭐ (2/5)
time-to-master: 6h
```

**Prerequisites**: [[linux-basics]]
**Related to**: [[ci-cd-basics]]

---

## 🎯 TL;DR (30 seconds)

Git is a distributed version control system that tracks code changes. GitHub is a remote hosting platform for Git repositories, enabling collaboration and backup.

**Analogy**: Git is like a time machine for your code - you can save snapshots (commits), create parallel universes (branches), and merge different timelines together.

---

## 🤔 When to Use?

### ✅ Good for
1. **Version control**: Track all changes to your codebase over time
2. **Collaboration**: Multiple developers working on the same project
3. **Feature development**: Create isolated branches for new features
4. **Code review**: Pull requests enable peer review before merging

### ❌ Bad for
- **Large binary files** → Use Git LFS instead
- **Generated files** → Add to .gitignore
- **Secrets/credentials** → Use environment variables or secret managers

---

## 📚 Key Concepts

### 1. Local vs Remote

**My understanding**:
Your local repository is on your machine. The remote (usually called `origin`) is on GitHub. You sync between them with push/pull.

**Why important**:
Understanding this separation prevents confusion about why changes aren't immediately visible to your team.

**Mental model**:
```
Your PC (local)                 GitHub (remote)
┌─────────────┐                ┌──────────────┐
│   main      │   push →       │ origin/main  │
│ (your copy) │   ← pull       │  (GitHub)    │
└─────────────┘                └──────────────┘
```

---

### 2. Fetch vs Pull

**My understanding**:
- `fetch` downloads changes from GitHub but doesn't modify your files
- `pull` downloads AND applies changes to your working directory
- `pull = fetch + merge`

**Why important**:
Fetching first lets you review changes before merging them into your work.

**Real example**:
```bash
# Safe: download without modifying
git fetch origin

# Check what changed
git log origin/main
git diff main origin/main

# Apply changes when ready
git pull origin main
```

---

### 3. Rebase vs Merge

**My understanding**:
- Merge creates a new commit joining two branches
- Rebase moves your commits to the end of another branch, creating linear history

**Why important**:
Rebasing keeps history clean but rewrites commits. Never rebase shared branches like `main`.

**Mental model**:
```
Before rebase:
main:    A---B---E---F
          \
feature:   C---D

After rebase:
main:    A---B---E---F
                    \
feature:             C'---D'
```

**Rule**: ✅ Rebase your feature branches, ❌ Never rebase main/shared branches

---

### 4. Branch Notation

**My understanding**:
- `main` = your local main branch
- `origin/main` = the main branch on GitHub
- `origin` is the default name for your remote repository

**Why important**:
Knowing whether you're referencing local or remote branches prevents confusion.

---

### 5. Force Push

**My understanding**:
After rebasing, your local history diverges from remote. Force push overwrites remote history.

**Why important**:
Force pushing can destroy work if someone else pushed in between. Use `--force-with-lease` as safety.

**Real example**:
```bash
# After rebase
git rebase origin/main

# Safe force push (fails if remote changed)
git push --force-with-lease origin feature/monitoring

# Dangerous (overwrites without checking)
git push --force origin feature  # ⚠️ Avoid
```

---

## 💻 Minimal Example

### Context
Starting a new feature with proper Git workflow and keeping it synced with main.

### Code
```bash
# === SETUP ===
# Create feature branch from updated main
git checkout main
git pull origin main
git checkout -b feature/user-auth

# === DAILY WORK ===
# Morning: update feature with latest main
git checkout main
git pull origin main
git checkout feature/user-auth
git rebase origin/main  # Move feature commits to end of main
git push --force-with-lease origin feature/user-auth

# During day: commit and push
git add .
git commit -m "feat: add login endpoint"
git push origin feature/user-auth

# === FINISH FEATURE ===
# Final push
git push origin feature/user-auth

# On GitHub: Create Pull Request
# feature/user-auth → main
# After merge on GitHub:

# Cleanup
git checkout main
git pull origin main  # Get merged changes
git branch -d feature/user-auth  # Delete local branch
```

**Output**: Clean feature development with linear history and proper synchronization.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Pushing secrets to GitHub

**Symptom**: Accidentally committed `.env` file with API keys

**What I did wrong**:
```bash
# ❌ Wrong
git add .
git commit -m "feat: add config"
git push  # .env is now in history forever
```

**Solution**:
```bash
# ✅ Correct - use .gitignore
echo ".env" >> .gitignore
echo "credentials.json" >> .gitignore
git add .gitignore
git commit -m "chore: add gitignore"

# If already committed, remove from history
git filter-branch --tree-filter 'rm -f .env' HEAD
# OR use git-filter-repo (better tool)
```

**Time wasted**: 2h
**Lesson**: Always add `.env`, `credentials.*`, `*.pem` to `.gitignore` BEFORE first commit.

---

### Pitfall 2: Rebase conflicts

**Symptom**: `git rebase origin/main` shows conflicts, panicked and aborted

**What I did wrong**:
```bash
# ❌ Wrong - gave up
git rebase origin/main
# CONFLICT...
git rebase --abort  # Lost opportunity to learn
```

**Solution**:
```bash
# ✅ Correct - resolve conflicts
git rebase origin/main
# CONFLICT in file.js

# 1. Open file.js, fix conflicts (remove <<<<<<, ======, >>>>>> markers)
# 2. Mark as resolved
git add file.js
# 3. Continue rebase
git rebase --continue

# If multiple conflicts, repeat until done
```

**Time wasted**: 1h (avoided rebasing for weeks)
**Lesson**: Rebase conflicts are normal. Fix, `git add`, `git rebase --continue`. Always have `git rebase --abort` as escape hatch.

---

### Pitfall 3: Detached HEAD state

**Symptom**: Made commits, then lost them after `git checkout main`

**What I did wrong**:
```bash
# ❌ Wrong - checked out commit directly
git checkout abc123  # Detached HEAD
# Made changes and committed
git commit -m "fix"
git checkout main  # Lost the commit!
```

**Solution**:
```bash
# ✅ Correct - create branch from detached HEAD
git checkout abc123
# Make changes
git checkout -b fix/issue-123  # Create branch
git commit -m "fix: issue"
git push origin fix/issue-123

# OR recover lost commits
git reflog  # Find lost commit
git checkout -b recovered <commit-hash>
```

**Time wasted**: 30min
**Lesson**: Never commit in detached HEAD. Always create a branch first.

---

## 📊 Stats

```yaml
Total time: 6h (30% assisted / 70% autonomous)
Used in: [[project-vps-setup]], [[project-glasck-deployment]]
```

---

**Last update**: 2025-12-30
**Next review**: 2026-01-30 (+1 month)
