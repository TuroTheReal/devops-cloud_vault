# Semantic Versioning - Git

## 📋 Metadata

```yaml
tags: [concept, git, versioning, release, semver, status/learning]
created: 2026-01-20
difficulty: ⭐⭐ (2/5)
time-to-master: 2h
```

**Prerequisites**: [[git-github-fundamentals]]
**Related to**: [[ci-cd-basics]] (coming soon), [[docker-backup-strategies]]

---

## 🎯 TL;DR (30 seconds)

> Semantic Versioning (SemVer) = `MAJOR.MINOR.PATCH` (e.g., v2.1.3). MAJOR = breaking changes, MINOR = new features (backward compatible), PATCH = bug fixes. Enables clear communication about changes between releases.
>
> **Analogy**: Like a contract with users - "upgrading from 2.1.3 to 2.1.4 is safe, but 2.x to 3.0 might break things."

---

## 🤔 When to Use?

### ✅ Good for
1. **Public APIs/Libraries**: Users need to know if upgrade is safe
2. **Docker images**: Tag images with version for rollback capability
3. **Release management**: Track what changed between versions
4. **CI/CD pipelines**: Automate versioning based on commit types

### ❌ Bad for
- **Internal scripts** → Keep it simple, maybe just date-based
- **Rapid prototyping** → Versioning overhead not worth it early on
- **Single-user projects** → Unless you need rollback capability

---

## 📚 Key Concepts

### 1. The Three Numbers

**My understanding**:
> `MAJOR.MINOR.PATCH` - each number has a specific meaning about the type of change.

**Why important**:
> Looking at version numbers tells you immediately if an upgrade is risky or safe.

**Breakdown**:
```
v2.1.3
│ │ └── PATCH (3): Bug fixes, no new features, safe to upgrade
│ └──── MINOR (1): New features, backward compatible, safe to upgrade
└────── MAJOR (2): Breaking changes, may require code changes to upgrade

Examples:
1.0.0 → 1.0.1  Bug fix (safe)
1.0.1 → 1.1.0  New feature added (safe)
1.1.0 → 2.0.0  Breaking change (careful!)
```

---

### 2. Pre-release and Build Metadata

**My understanding**:
> You can add suffixes for pre-releases (`-alpha`, `-beta`, `-rc.1`) and build metadata (`+build.123`).

**Why important**:
> Indicates stability level - alpha < beta < rc (release candidate) < stable.

**Examples**:
```
1.0.0-alpha      First test version, unstable
1.0.0-beta       Feature complete, still testing
1.0.0-rc.1       Release candidate 1, almost ready
1.0.0-rc.2       Release candidate 2, fixes from rc.1
1.0.0            Stable release

1.0.0+build.456  Build metadata (ignored in precedence)
```

**Sorting order**:
```
1.0.0-alpha < 1.0.0-alpha.1 < 1.0.0-beta < 1.0.0-rc.1 < 1.0.0
```

---

### 3. Git Tags for Versions

**My understanding**:
> Git tags mark specific commits as releases. Convention: prefix with `v` (v1.0.0). Tags are immutable references to commits.

**Why important**:
> Tags let you checkout any previous version and are used by CI/CD for release automation.

**Key commands**:
```bash
# Create annotated tag (recommended)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Push tag to remote
git push origin v1.0.0

# Push all tags
git push origin --tags

# List tags
git tag -l "v1.*"

# Checkout specific version
git checkout v1.0.0

# Delete tag (local)
git tag -d v1.0.0

# Delete tag (remote)
git push origin --delete v1.0.0
```

---

### 4. When to Bump Each Number

**My understanding**:
> Rules for deciding which number to increment based on type of change.

**Decision tree**:
```
Did you change the public API in a backward-incompatible way?
├── YES → Bump MAJOR (reset MINOR and PATCH to 0)
│         Example: Renamed function, removed parameter, changed return type
│
└── NO → Did you add new functionality?
         ├── YES → Bump MINOR (reset PATCH to 0)
         │         Example: New endpoint, new optional parameter, new feature
         │
         └── NO → Bump PATCH
                  Example: Bug fix, performance improvement, typo fix
```

**Real examples**:
```
v1.2.3 → v1.2.4   Fixed null pointer exception
v1.2.4 → v1.3.0   Added new /users/search endpoint
v1.3.0 → v2.0.0   Changed /users response format (breaking)
```

---

### 5. Conventional Commits (Optional but Recommended)

**My understanding**:
> Commit message format that can automate version bumps: `type(scope): description`. Tools like `semantic-release` read these to auto-bump versions.

**Why important**:
> Removes human decision from versioning - commits determine the next version automatically.

**Format**:
```
type(scope): description

Types:
- fix:      Bug fix              → PATCH bump
- feat:     New feature          → MINOR bump
- BREAKING CHANGE: in footer     → MAJOR bump
- chore:    Maintenance          → No bump
- docs:     Documentation        → No bump
- refactor: Code restructure     → No bump (or PATCH)
```

**Examples**:
```bash
git commit -m "fix: resolve null pointer in user service"
# → Triggers PATCH bump (1.2.3 → 1.2.4)

git commit -m "feat: add user search endpoint"
# → Triggers MINOR bump (1.2.4 → 1.3.0)

git commit -m "feat!: change user response format

BREAKING CHANGE: user.name is now user.fullName"
# → Triggers MAJOR bump (1.3.0 → 2.0.0)
```

---

## 💻 Minimal Example

### Context
Release workflow: develop feature, tag release, push to trigger CI/CD.

### Code
```bash
# === FEATURE DEVELOPMENT ===
git checkout -b feature/search-users
# ... develop ...
git commit -m "feat: add user search endpoint"
git push origin feature/search-users
# Create PR, get review, merge to main

# === RELEASE ===
git checkout main
git pull origin main

# Check current version
git tag -l "v*" | sort -V | tail -1
# Output: v1.2.3

# Decide new version (added feature = MINOR bump)
# v1.2.3 → v1.3.0

# Create annotated tag
git tag -a v1.3.0 -m "Release v1.3.0: Add user search endpoint"

# Push tag (triggers CI/CD release)
git push origin v1.3.0

# === DOCKER IMAGE TAGGING ===
# In CI/CD or manually:
docker build -t myapp:v1.3.0 .
docker tag myapp:v1.3.0 myapp:latest
docker push myapp:v1.3.0
docker push myapp:latest
```

**Output**: Tagged release, Docker image with version, CI/CD triggered.

---

## 📊 Versioning in Different Contexts

| Context | Version Location | Example |
|---------|------------------|---------|
| Git repository | Tags | `git tag v1.3.0` |
| Docker image | Image tag | `myapp:v1.3.0` |
| Node.js | package.json | `"version": "1.3.0"` |
| Python | pyproject.toml | `version = "1.3.0"` |
| Go | go module + tags | `module/v2` for major |
| API | URL or header | `/api/v2/users` |

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Forgetting to push tags

**Symptom**: Tagged locally, CI/CD didn't trigger, wondering why.

**What I did wrong**:
```bash
# ❌ Wrong - tag exists only locally
git tag -a v1.0.0 -m "Release"
git push origin main  # Tag NOT pushed!
```

**Solution**:
```bash
# ✅ Correct - push tag explicitly
git tag -a v1.0.0 -m "Release"
git push origin v1.0.0  # Push specific tag

# OR push all tags
git push origin --tags
```

**Lesson**: Tags are not pushed with `git push` - you must push them explicitly.

---

### Pitfall 2: Starting at v0.x.x vs v1.0.0

**Symptom**: Confused about when to use 0.x.x

**My understanding**:
```
v0.x.x = Development phase, API unstable, anything can change
v1.0.0 = First stable release, semver rules now apply

Rule: Stay at 0.x.x until your API is stable and documented.
      Once at 1.0.0, breaking changes require MAJOR bump.
```

**Lesson**: Don't rush to v1.0.0 - it's a commitment to stability.

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> What does each number in v2.1.3 represent?</summary>

**Answer**:
- 2 = MAJOR (breaking changes)
- 1 = MINOR (new features, backward compatible)
- 3 = PATCH (bug fixes)
</details>

<details>
<summary><strong>Q2:</strong> You added a new optional parameter to an API endpoint. Which number do you bump?</summary>

**Answer**: MINOR. New feature that doesn't break existing code = MINOR bump. v1.2.3 → v1.3.0
</details>

<details>
<summary><strong>Q3:</strong> How do you create and push a Git tag for a release?</summary>

**Answer**:
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0
```
Tags must be pushed explicitly - they don't go with regular `git push`.
</details>

---

## 📊 Stats

```yaml
Total time: 1h (40% assisted / 60% autonomous)
Used in: [[docker-backup-strategies]], [[ci-cd-basics]]
```

---

**Last update**: 2026-01-20
**Next review**: 2026-02-20 (+1 month)
