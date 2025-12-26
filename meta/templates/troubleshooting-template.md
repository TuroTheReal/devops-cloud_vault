# [Error/Issue Name] - Quick Fix

## 📋 Metadata

```yaml
tags: [troubleshooting, technology, error-type, status/documented]
created: YYYY-MM-DD
updated: YYYY-MM-DD
severity: 🔴 Critical / 🟠 High / 🟡 Medium / 🟢 Low
time-to-fix: Xmin
```

**Related concept**: [[concepts/technology/related-concept]]
**Affected versions**: Technology vX.X+

---

## 🔍 Symptom

> **Short, clear description of the observable problem**
>
> What the user sees/experiences.

```bash
# Example error message or command output showing the problem
Error: connection refused
Port 22 not accessible
```

**Common scenarios where this occurs**:
- Scenario 1
- Scenario 2

---

## 🎯 Root Cause

> 1-2 sentences explaining WHY this happens.
> Technical explanation concise.

**Why this happens**:
- Root cause 1
- Root cause 2

---

## ✅ Quick Fix

### Solution 1 (Recommended)

```bash
# Step-by-step commands to fix
sudo command-to-fix

# Verify fix worked
command-to-verify
```

**Explanation**: Why this works.

---

### Solution 2 (Alternative)

```bash
# Alternative approach if Solution 1 doesn't work
alternative-command
```

**When to use**: If [specific condition].

---

## 🔬 Diagnosis Steps

**If quick fix doesn't work, debug with**:

```bash
# 1. Check service status
systemctl status service-name

# 2. View logs
journalctl -u service-name -n 50

# 3. Verify configuration
command-to-check-config
```

---

## 🛡️ Prevention

> How to avoid this issue in the future.

**Best practices**:
- Prevention tip 1
- Prevention tip 2

---

## 🔗 Related Issues

- [[troubleshooting/related-issue-1]]
- [[troubleshooting/related-issue-2]]

**For deep understanding**: See [[concepts/technology/detailed-concept]]

---

## 📊 Stats

```yaml
Encountered: X times
Last occurrence: YYYY-MM-DD
Typical fix time: Xmin
Projects affected: [[project-1]], [[project-2]]
```

---

**Last update**: YYYY-MM-DD
