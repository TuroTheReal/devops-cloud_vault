# SSH Hardening - Production Security

## 📋 Metadata

```yaml
tags: [concept, linux, security, ssh, hardening, status/learned]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 1h30
```

**Prerequisites**: [[linux-basics]], [[networking-fundamentals]]
**Related to**: [[linux-firewall-ufw]], [[linux-fail2ban]], [[linux-systemd-socket]]

---

## 🎯 TL;DR (30 seconds)

> SSH hardening = securing SSH access through multiple layers: custom port, key-only auth, no root login, and Fail2Ban. Essential for any public-facing server to prevent brute-force attacks and unauthorized access.
>
> **Analogy**: Medieval castle with moat (custom port), drawbridge (firewall), guards checking IDs (keys), and banning troublemakers (Fail2Ban)

---

## 🤔 When to Use?

### ✅ Good for
- **Production VPS**: Any server exposed on Internet 24/7
- **Multi-user systems**: Multiple admins access the server
- **Sensitive data**: Servers hosting critical/private data

### ❌ Bad for
- **Local dev VM**: Local machines behind NAT (unnecessary overhead)
- **Isolated networks**: Servers on private network without Internet access
- **Learning environments**: Disposable VMs for testing (complicates debugging)

---

## 📚 Key Concepts

### 1. Multi-Layer SSH Security

**My understanding**:
> **SSH security works best with multiple independent layers - if one fails, others still protect**
>
> Different layers of SSH security:
> - Custom port (avoid automated scans)
> - Keys only (no passwords)
> - No root login
> - Fail2Ban
>
> Why multi-layer? A single layer can be bypassed, but multiple layers make it exponentially harder.

**Why important**:
> **Defense in depth - no single point of failure**

**Mental model**:
> **SSH security = medieval castle with moat, walls, guards, and dungeon for intruders**

---

### 2. SSH Keys vs Passwords

**My understanding**:
SSH supports 2 authentication methods:

**Passwords**:
- ❌ Can be brute-forced (dictionaries, rainbow tables)
- ❌ Reused across services (if compromised, cascade failure)
- ❌ Transmitted over network (if encryption broken = leak)
- ✅ Easy to use (no files to manage)

**SSH Keys (ED25519/RSA)**:
- ✅ Impossible to brute-force (2048-4096 bits, cryptographically secure)
- ✅ Public/private pair (public on server, private never transmitted)
- ✅ Can be protected by passphrase (2-factor)
- ❌ More complex (file management, backups)

**Best practice**: Keys only, disable password auth.

**Difference with default**:
```bash
# Default Ubuntu SSH
PasswordAuthentication yes  # ❌ Accepts passwords
PermitRootLogin yes         # ❌ Root can connect

# Hardened SSH
PasswordAuthentication no   # ✅ Keys only
PermitRootLogin no          # ✅ Forces normal user + sudo
```

---

### 3. Custom SSH Port (Security by Obscurity)

**My understanding**:
> **To reduce obvious attacks, change the base port pattern**
>
> Why change SSH port from 22 → 2222 (or other):
> - Port 22 = scanned 24/7 by bots
> - Custom port = reduces 90%+ of noise (logs, CPU)
> - This is NOT real security (port scan will find it anyway)
> - But reduces attack surface (fewer attempts = fewer risks)

**Why important**:
> **Reduces attack surface**

**Gotcha - systemd socket activation**:
> `ssh.socket` overrides `sshd_config`, so you must disable ssh socket for sshd_config to take over
> See [[linux-systemd-socket]] for details.

---

### 4. Root Login Prohibition

**My understanding**:
> **Would allow via X method to have access to VPS root or other**
>
> Why `PermitRootLogin no`:
> - Root = all permissions (compromise = game over)
> - Normal user + sudo = audit trail (logs who did what)
> - Isolation (user can't accidentally rm -rf /)
>
> Workflow: SSH as user → sudo for admin tasks

**Why important**:
> **Avoids having a second malicious root who could delete EVERYTHING for example**

**Mental model**:
> **Having a second you but malicious - we don't want that**

---

## 💻 Minimal Example

### Context
From VPS Hetzner hardening - Complete SSH hardening workflow with custom port, keys only, no root, systemd socket fix.

### Code

**1. Generate SSH key (client side)**:
```bash
# Generate ED25519 key (modern, secure, fast)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Alternative: RSA 4096 (more compatible but slower)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Output:
# Private key: ~/.ssh/id_ed25519 (NEVER share)
# Public key: ~/.ssh/id_ed25519.pub (copy to server)
```

**2. Copy public key to server**:
```bash
# Method 1: ssh-copy-id (automatic)
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server_ip

# Method 2: Manual
cat ~/.ssh/id_ed25519.pub | ssh user@server_ip "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

**3. Harden SSH config** (`/etc/ssh/sshd_config`):
```bash
# Custom port (avoid automated scans)
Port 2222

# Keys only, no passwords
PasswordAuthentication no
PubkeyAuthentication yes

# No root login
PermitRootLogin no

# Disable empty passwords
PermitEmptyPasswords no

# Disable X11 forwarding (if not needed)
X11Forwarding no

# Max auth attempts
MaxAuthTries 3

# Disconnect if no login within 30s
LoginGraceTime 30
```

**4. Fix systemd socket (Ubuntu)**:
```bash
# Check if ssh.socket active
systemctl status ssh
# Look for "TriggeredBy: ● ssh.socket"

# Disable socket, enable direct service
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl enable ssh
sudo systemctl restart ssh

# Verify listening port
ss -tuln | grep 2222
```

**5. Test BEFORE closing current session**:
```bash
# In terminal 1: Keep current SSH session open
# In terminal 2: Test new connection
ssh -p 2222 user@server_ip

# If works → close terminal 1
# If fails → debug in terminal 1 (you're not locked out!)
```

### Line-by-Line Explanation
```
Port 2222
  → SSH listens on custom port instead of default 22
  → Reduces automated scan attempts by 90%+
  → NOT real security (port scan will find it) but reduces noise

PasswordAuthentication no
  → Only SSH keys accepted, passwords rejected
  → Impossible to brute-force (keys are cryptographically secure)
  → Even if password leaked elsewhere, can't be used here

PermitRootLogin no
  → Root user cannot login via SSH (even with valid key)
  → Forces login as normal user, then sudo for admin
  → Creates audit trail (who did what, when)

systemctl disable ssh.socket
  → Systemd socket activation overrides sshd_config Port setting
  → Socket defines listening port (hardcoded to 22 by default)
  → Must disable socket to use custom port from sshd_config
  → See [[linux-systemd-socket]] for deep dive
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: SSH Port Not Changing - systemd Socket Override

**Symptom**:
> Connection refused automatically after changing port in config
>
> Example:
> ```
> Modified /etc/ssh/sshd_config with Port 2222
> Restarted SSH service
> But `ss -tuln` still shows port 22
> ```

**What I did wrong**:
> Did not verify SSH startup mode - socket vs service

**Why wrong**:
> Need to change ssh socket binding or disable socket to use sshd_config rules

**Solution**:
```bash
# ✅ Correct - Disable socket, use direct service
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl enable ssh
sudo systemctl restart ssh
```

**Time wasted**: 30 minutes
**Lesson**: Check SSH startup mode (socket vs service) before changing port configuration

---

### Pitfall 2: Locked Out - Forgot to Update Firewall

**Symptom**:
> Changed SSH port to 2222, restarted SSH service, closed terminal. Can't reconnect because firewall (UFW) still only allows port 22, blocks new port 2222.
>
> Example:
> ```
> Changed SSH port to 2222
> Restarted SSH service successfully
> Closed terminal
> Can't reconnect (firewall blocks new port)
> Need VPS console access to fix
> ```

**What I did wrong**:
> Changed SSH port but forgot to update firewall rules to allow the new port

**Why wrong**:
> SSH listening on 2222 but UFW only allows 22 - complete lockout

**Solution**:
```bash
# ✅ Correct workflow:
# 1. FIRST allow new port in firewall
sudo ufw allow 2222/tcp

# 2. Change SSH config
sudo nano /etc/ssh/sshd_config  # Port 2222

# 3. Restart SSH
sudo systemctl restart ssh

# 4. Test in NEW terminal (keep old one open)
ssh -p 2222 user@server

# 5. Only after successful test, remove old port
sudo ufw delete allow 22/tcp
```

**Time wasted**: 20-45 minutes (requires VPS console access)
**Lesson**: Always keep 2 terminals open when changing SSH config, test before closing, update firewall BEFORE restarting SSH

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why change SSH port from 22 to 2222 if port scanners can find it anyway?</summary>

**Answer**: Reduces 90%+ of automated bot attacks (they scan port 22 only). Not real security but reduces noise in logs and CPU load. Fewer attack attempts = fewer risks of zero-day exploits.
</details>

<details>
<summary><strong>Q2:</strong> What's the correct workflow to change SSH port without getting locked out?</summary>

**Answer**:
1. `ufw allow 2222/tcp` (FIRST - open new port in firewall)
2. Edit `sshd_config` → `Port 2222`
3. `systemctl restart ssh`
4. Test in NEW terminal (keep old one open!)
5. Only after success: `ufw delete allow 22/tcp`
</details>

<details>
<summary><strong>Q3:</strong> Why does changing Port in sshd_config sometimes not work? How to detect this?</summary>

**Answer**: systemd socket activation - `ssh.socket` has `ListenStream=22` which overrides `sshd_config`.
**Detection**: `systemctl status ssh | grep TriggeredBy` shows `ssh.socket`.
**Fix**: Disable socket, enable direct service.
</details>

<details>
<summary><strong>Q4:</strong> Why "PermitRootLogin no" instead of just using strong password for root?</summary>

**Answer**:
- Root = all permissions (compromise = game over)
- Normal user + sudo = audit trail (logs who did what)
- Isolation (can't accidentally `rm -rf /`)
Workflow: SSH as user → sudo for admin tasks
</details>

---

## 📊 Stats

```yaml
Total time: 1h30 (45% assisted / 55% autonomous)
Used in: [[2025-12-vps-hetzner-init-setup/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
