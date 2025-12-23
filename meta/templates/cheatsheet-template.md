# [Tool] - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, [tech]]
created: YYYY-MM-DD
version: X.Y.Z
```

**Official docs**: [URL]
**Related concepts**: [[concept-1]]

---

## 🚀 Installation

```bash
# macOS
brew install [tool]

# Ubuntu
apt install [tool]

# Verification
[tool] --version
```

---

## 📚 Essential Commands

### List / Get
```bash
# List all
[tool] get [resource]

# With details
[tool] get [resource] -o wide

# YAML format
[tool] get [resource] -o yaml
```

### Create
```bash
# From file
[tool] create -f file.yaml

# Imperative
[tool] create [resource] [name] [options]
```

### Modify
```bash
# Apply changes
[tool] apply -f file.yaml

# Interactive edit
[tool] edit [resource] [name]
```

### Delete
```bash
# By name
[tool] delete [resource] [name]

# By selector
[tool] delete [resource] -l [label]
```

---

## 🔍 Debug / Inspection

### Logs
```bash
# View logs
[tool] logs [resource]

# Follow logs
[tool] logs [resource] -f

# Filter
[tool] logs [resource] | grep ERROR
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
# Shell into resource
[tool] exec -it [resource] -- /bin/bash

# One-shot command
[tool] exec [resource] -- [command]
```

---

## 🎨 Formatting

### Output Formats
```bash
# Wide
[tool] get [resource] -o wide

# JSON
[tool] get [resource] -o json

# Custom columns
[tool] get [resource] -o custom-columns=NAME:.metadata.name
```

### Filtering
```bash
# By label
[tool] get [resource] -l key=value

# By field
[tool] get [resource] --field-selector status=Running
```

---

## 🎯 Useful One-Liners

```bash
# [Description 1]
[complex command 1]

# [Description 2]
[command with pipe 2]

# [Description 3]
[loop command 3]
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

## 📝 Config File

```yaml
# [File description]
apiVersion: [version]
kind: [type]
metadata:
  name: [name]
spec:
  # Configuration
  [field]: [value]
```

**Required fields**:
- `field1`: [Description]
- `field2`: [Description]

---

## 🔗 Aliases

```bash
# ~/.bashrc or ~/.zshrc

alias k='[tool]'
alias kg='[tool] get'
alias kd='[tool] describe'
alias kl='[tool] logs'

# Useful function
function [name]() {
    [commands with $1, $2]
}
```

---

## ✅ Best Practices

### DO
- **[Practice 1]**: [Why]
  ```bash
  # ✅ Good
  [command]
  ```

- **[Practice 2]**: [Why]

### DON'T
- **[Anti-pattern 1]**: [Why bad]
  ```bash
  # ❌ Bad
  [dangerous command]

  # ✅ Good
  [safe command]
  ```

---

## 📖 Resources

- [Official Docs](URL)
- [[fundamental-concept]]
- [[troubleshooting-[tool]]]

---

## 💡 Personal Tips

### Daily workflow
```bash
# Typical routine
[commands I use often]
```

### Mistakes made
- **[Error 1]** → Fix: [Solution]
- **[Error 2]** → Fix: [Solution]

---

## 📌 Quick Reference Card

```
┌────────────────────────────────────┐
│     [TOOL] - ESSENTIALS            │
├────────────────────────────────────┤
│ LIST       [command]               │
│ CREATE     [command]               │
│ UPDATE     [command]               │
│ DELETE     [command]               │
│ LOGS       [command]               │
│ DEBUG      [command]               │
│                                    │
│ Flags:                             │
│   -o [format]                      │
│   -l [label]                       │
│   -f [file]                        │
└────────────────────────────────────┘
```

---

**Last update**: YYYY-MM-DD
**Tool version**: X.Y.Z
