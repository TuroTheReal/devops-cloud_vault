# Linux Basics - Fundamental Concepts

## 📋 Metadata

```yaml
tags: [concept, linux, fundamentals, cli, filesystem, status/learned]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 2h
```

**Prerequisites**: None (foundational)
**Related to**: [[linux-ssh-hardening]], [[linux-firewall-ufw]]

---

## 🎯 TL;DR (30 seconds)

> Linux = open-source OS, multi-user, permissions-based. Everything is a file. Administration via CLI (bash).
>
> **Analogy**: Like flour in bread - you have different types (Windows, Fedora, ...) but you need one to get your computer running

---

## 🤔 When to Use?

### ✅ Good for
- **Server infrastructure**: Dominant for VPS, containers, cloud
- **Development environments**: DevOps, automation, scripting
- **Cost-effective**: Free, stable, secure

### ❌ Bad for
- **Desktop gaming**: Limited compatibility (improving but not there yet)
- **Enterprise Office**: MS Office, Adobe suite (alternatives exist but different)

---

## 📚 Key Concepts

### 1. Everything is a File

**My understanding**:
> **Everything can be treated as a file, you can interact with it using standard file operations to get information or change its behavior**
>
> Linux philosophy: Hardware, processes, sockets = represented as a file
> - Disk: `/dev/sda`
> - Process: `/proc/1234` (process info accessible as files, PID = Process ID)
> - Socket: `/var/run/docker.sock`
>
> Advantage: Uniform interface (read, write, permissions)

---

### 2. File System Hierarchy

**My understanding**:
> **Every directory has its own purpose, rules & authorizations**
>
> Standardized directory structure:
> - `/etc` = Configuration files
> - `/var` = Variable data (logs, cache)
> - `/home` = User directories
> - `/usr` = User programs
> - `/tmp` = Temporary files

---

### 3. Users and Permissions

**My understanding**:
> **User MUST be defined properly to avoid HUGE security fail, less privileges for all users (ex: a personal user should not be used to run an app in background)**
>
> Permissions model: User / Group / Others
> - `rwx` = Read, Write, Execute
> - `chmod 755` = Owner full, others read+execute
> - `sudo` = Execute as root temporarily

---

### 4. Package Management

**My understanding**:
> **Depending on OS, tools to install everything you need on your setup**
>
> Package manager by distro:
> - Debian/Ubuntu: `apt` (Advanced Package Tool)
> - RHEL/CentOS: `yum`/`dnf`
> - Arch: `pacman`
>
> Installs software + dependencies automatically

---

## 💻 Minimal Example

### Essential Commands

```bash
# Navigation
pwd                    # Print working directory
ls -la                 # List files (all, long format)
cd /path/to/dir        # Change directory

# File operations
cp source dest         # Copy file
mv source dest         # Move/rename file
rm file                # Remove file
mkdir dir              # Create directory

# Viewing files
cat file               # Display entire file
less file              # Page through file
head -n 10 file        # First 10 lines
tail -f file           # Follow file (logs)

# Permissions
chmod 644 file         # rw-r--r--
chown user:group file  # Change owner
sudo command           # Run as root

# Package management (Ubuntu)
sudo apt update        # Update package lists
sudo apt install pkg   # Install package
sudo apt remove pkg    # Remove package

# System info
uname -a               # Kernel version
df -h                  # Disk usage
free -h                # RAM usage
top                    # Process monitor
```

---

## ⚠️ Key Pitfall: Running Commands as Root

**Symptom**:
```bash
sudo rm -rf /important/data  # Typo, deletes everything!
```

**Lesson**:
> Always double-check sudo commands, there's no undo button. Running commands as root bypasses all safety checks - a single typo can wipe your entire system. Use sudo only when necessary, and always verify paths before executing destructive commands (rm, mv, chmod, etc.).

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why is the "everything is a file" philosophy important in Linux administration?</summary>

**Answer**: It provides a uniform interface for interacting with different system components. Hardware devices, processes, and network sockets can all be accessed using standard file operations (read, write, permissions), making system administration more consistent and scriptable.
</details>

<details>
<summary><strong>Q2:</strong> What's the danger of running commands with sudo, and how should you approach it?</summary>

**Answer**: Running commands as root bypasses all safety checks - a single typo can wipe your entire system with no undo. Always double-check sudo commands (especially destructive ones like rm, mv, chmod) and verify paths before executing. Use sudo only when necessary.
</details>

<details>
<summary><strong>Q3:</strong> Why should you use separate users for different purposes rather than running everything as root or your personal user?</summary>

**Answer**: Following the principle of least privilege - each user should have only the minimum permissions needed. A personal user shouldn't run background apps, and apps shouldn't have full root access. This limits damage if a service is compromised and prevents security failures.
</details>

---

## 📊 Stats

```yaml
Total time: 2h (30% assisted / 70% autonomous)
Used in: [[2025-12-vps-hetzner-init-setup/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
