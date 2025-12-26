# [Tool] - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, [tech]]
created: YYYY-MM-DD
version: X.Y.Z
```

**Official docs**: [URL]
**Related concepts**: [[concept-1]], [[concept-2]]

---

## 🚀 Quick Start

```bash
# Installation
[install commands for macOS, Ubuntu, etc.]

# Basic usage (3-5 essential commands)
[tool] [command1]
[tool] [command2]
[tool] [command3]
```

---

## 📚 Commands by Operation

### List / Get
```bash
# List all
[tool] get [resource]

# With details
[tool] get [resource] -o wide

# Filter
[tool] get [resource] -l key=value
```

### Create
```bash
# From file
[tool] create -f file.yaml

# Imperative
[tool] create [resource] [name] [options]
```

### Update / Modify
```bash
# Apply changes
[tool] apply -f file.yaml

# Edit interactively
[tool] edit [resource] [name]
```

### Delete
```bash
# By name
[tool] delete [resource] [name]

# By selector
[tool] delete [resource] -l [label]

# Force delete
[tool] delete [resource] [name] --force
```

---

## 🔍 Debug / Inspection

### Logs
```bash
# View logs
[tool] logs [resource]

# Follow logs
[tool] logs [resource] -f

# Last N lines
[tool] logs [resource] --tail=100
```

### Describe / Info
```bash
# Full details
[tool] describe [resource] [name]

# Status
[tool] get [resource] [name] -o wide
```

### Shell / Exec
```bash
# Interactive shell
[tool] exec -it [resource] -- /bin/bash

# One-shot command
[tool] exec [resource] -- [command]
```

---

## 🎯 Useful One-Liners

```bash
# [Description of what this does]
[complex command from real usage]

# [Description]
[command with pipe]

# [Description]
[loop or advanced command]
```

---

## 🆘 Troubleshooting

### Problem 1: [Symptom]
```bash
# Diagnostic
[check command]

# Fix
[fix command]
```

### Problem 2: [Symptom]
```bash
# Check + Fix
[commands]
```

---

## 💡 Personal Notes

**Mistakes made**:
- **[Error 1]** → Fix: [Solution]
- **[Error 2]** → Fix: [Solution]

**Daily workflow** (commands I actually use):
```bash
[Your typical routine commands]
```

---

## 📝 Config Example (optional)

```yaml
# [Common config file structure]
apiVersion: [version]
kind: [type]
metadata:
  name: [name]
spec:
  [key fields with comments]
```

---

**Last update**: YYYY-MM-DD
**Tool version**: X.Y.Z
