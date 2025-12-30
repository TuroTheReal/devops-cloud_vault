# UFW - Uncomplicated Firewall

## 📋 Metadata

```yaml
tags: [concept, linux, security, firewall, ufw, iptables, status/learned]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 1h
```

**Prerequisites**: [[linux-basics]], [[networking-fundamentals]]
**Related to**: [[linux-ssh-hardening]], [[linux-fail2ban]]

---

## 🎯 TL;DR (30 seconds)

> Simplified frontend for iptables, deny-all by default + allow specific ports.
>
> **Analogy**: Receptionist who checks if your key matches the right room number

---

## 🤔 When to Use?

### ✅ Good for
- **Production servers**: Any VPS exposed on Internet
- **Defense-in-depth**: Additional layer with cloud firewall (Hetzner, AWS Security Groups)
- **Quick rule management**: Simple syntax vs raw iptables

### ❌ Bad for
- **Complex routing**: Need advanced NAT, policy routing → iptables direct
- **High-performance firewall**: UFW = minimal overhead but exists
- **Desktop Linux**: Rarely necessary (no ports exposed publicly)

---

## 📚 Key Concepts

### 1. UFW = iptables Frontend

**My understanding**:
> **Monitors specified ports**
>
> Explains the UFW ↔ iptables relationship:
> - iptables = Linux kernel firewall (powerful but complex syntax)
> - UFW = user-friendly wrapper around iptables
> - UFW generates iptables rules automatically
>
> Example: `ufw allow 22/tcp` → generates ~5 iptables lines

**Why important**:
> **Without this, possibility to enter through any door directly**

**Mental model**:
```
User Command (UFW)
  ↓
UFW translates
  ↓
iptables rules
  ↓
Kernel netfilter (packet filtering)
```

---

### 2. Default Deny Policy

**My understanding**:
> **Allows accepting nobody unless there are rules to say yes**
>
> Zero-trust firewall principle:
> - By default: DENY all incoming, ALLOW all outgoing
> - Then explicitly ALLOW necessary ports (SSH, HTTP, HTTPS)
> - Everything else = blocked
>
> Opposite of "allow all by default" (dangerous)

**Why important**:
> **Zero trust principle == less attack surface**

**Default UFW behavior**:
```bash
# After enabling UFW:
Incoming: DENY (all ports blocked by default)
Outgoing: ALLOW (server can reach Internet)
Routed: DENY (no packet forwarding between interfaces)

# Then explicitly allow needed services:
ufw allow 22/tcp  # SSH
ufw allow 80/tcp  # HTTP
ufw allow 443/tcp # HTTPS
```

---

### 3. Defense-in-Depth: Cloud Firewall + UFW

**My understanding**:
> **Manages traffic according to cloud provider firewall rules, if inside, lets connections arrive, UFW takes over here**
>
> Why 2 firewalls (ex: Hetzner Firewall + UFW):
>
> **Cloud Firewall (Hetzner, AWS SG)**:
> - Filters BEFORE packets arrive at VPS
> - DDoS protection (packets never consume VPS resources)
> - Persists even if VPS compromised (attacker can't disable)
>
> **UFW (on VPS)**:
> - Granular control (per-application rules)
> - Scriptable (automation, dynamic rules)
> - But if attacker gets root → can disable UFW
>
> Together = defense-in-depth (2 independent layers)

**Why important**:
> **Avoids connection management overload on VPS**

**Mental model**:
> **Analogy**: Cloud firewall = checkpoint at parking entrance, UFW = car alarm

---

### 4. Application Profiles

**My understanding**:
UFW supports "application profiles" = preset rules for common services.

```bash
# List available profiles
sudo ufw app list
# Output: OpenSSH, Nginx Full, Nginx HTTP, Apache, etc.

# Allow by profile (instead of manual port)
sudo ufw allow OpenSSH
# Equivalent to: sudo ufw allow 22/tcp
```

**Profiles location**: `/etc/ufw/applications.d/`

**Custom profile example**:
```ini
[CustomApp]
title=My Custom Application
description=Custom app on port 8080
ports=8080/tcp
```

**Use case**: Cleaner rules (`ufw allow Nginx Full` vs `ufw allow 80/tcp && ufw allow 443/tcp`)

---

## 💻 Minimal Example

### Context
From VPS Hetzner hardening - Basic UFW setup with SSH custom port, deny-all policy, safe enablement workflow.

### Code

**1. Initial setup (before enabling)**:
```bash
# Check UFW status (should be inactive on fresh Ubuntu)
sudo ufw status
# Status: inactive

# Set default policies (CRITICAL - do this BEFORE enabling)
sudo ufw default deny incoming   # Block all incoming by default
sudo ufw default allow outgoing  # Allow server to reach Internet
sudo ufw default deny routed     # No forwarding between interfaces
```

**2. Allow SSH port (CRITICAL - avoid lockout)**:
```bash
# BEFORE enabling UFW, allow your SSH port
# If custom port (example: 2222)
sudo ufw allow 2222/tcp comment 'SSH custom port'

# If standard port 22
sudo ufw allow 22/tcp comment 'SSH'

# Verify rule added
sudo ufw status verbose
```

**3. Add other needed services**:
```bash
# HTTP/HTTPS for web server
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'

# Custom application port
sudo ufw allow 8080/tcp comment 'App backend'

# Port range (if needed)
sudo ufw allow 6000:6010/tcp comment 'App range'
```

**4. Enable firewall**:
```bash
# Enable UFW (will warn about disrupting SSH)
sudo ufw enable

# Verify status
sudo ufw status verbose

# Output:
# Status: active
# Logging: on (low)
# Default: deny (incoming), allow (outgoing), deny (routed)
#
# To                         Action      From
# --                         ------      ----
# 2222/tcp                   ALLOW IN    Anywhere    # SSH custom port
# 80/tcp                     ALLOW IN    Anywhere    # HTTP
# 443/tcp                    ALLOW IN    Anywhere    # HTTPS
```

**5. Common operations**:
```bash
# Delete rule (by number)
sudo ufw status numbered
sudo ufw delete 3

# Delete rule (by specification)
sudo ufw delete allow 8080/tcp

# Disable UFW (emergency - reopens all ports!)
sudo ufw disable

# Reload rules
sudo ufw reload

# Reset to defaults (deletes all rules!)
sudo ufw reset
```

### Line-by-Line Explanation
```
sudo ufw default deny incoming
  → Sets default policy: all incoming connections DENIED
  → Unless explicitly allowed with `ufw allow` command
  → Prevents unexpected services from being accessible

sudo ufw allow 2222/tcp comment 'SSH custom port'
  → Explicitly allows TCP port 2222 (custom SSH)
  → Comment helps document purpose of rule (shown in status)
  → MUST do this BEFORE `ufw enable` to avoid lockout

sudo ufw enable
  → Activates firewall and applies all rules
  → Persists across reboots (enabled at boot)
  → WARNING: Can cut SSH if port not allowed first

sudo ufw status verbose
  → Shows active rules, default policies, logging level
  → "verbose" adds more detail (default policies, logging)
  → Use to verify rules before/after changes
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Enabling UFW Without Allowing SSH First

**Symptom**:
> Running `sudo ufw enable` immediately drops SSH connection. Can't reconnect because port 22 is blocked. Need VPS console access to fix.
>
> Example:
> ```
> Run `sudo ufw enable`
> SSH connection immediately drops
> Can't reconnect (port 22 blocked)
> Need VPS console access to fix
> ```

**What I did wrong**:
> Wrong order of settings - enabled firewall before allowing SSH port

**Why wrong**:
> Blocks UFW access before registering SSH key == lockout

**Solution**:
```bash
# ✅ Correct order:
# 1. FIRST allow SSH port
sudo ufw allow 2222/tcp

# 2. Verify rule added
sudo ufw status

# 3. THEN enable firewall
sudo ufw enable

# 4. Test SSH still works
# (keep current session open, test in new terminal)
```

**Lesson**: Always allow SSH port BEFORE enabling firewall - verify rules are set before activating

---

### Pitfall 2: UFW Rules vs Docker Port Publishing

**Symptom**:
> Published Docker container port is accessible from Internet even though UFW shows no allow rule for that port. UFW appears to be bypassed.
>
> Example:
> ```
> Published Docker container port 5432 (PostgreSQL)
> UFW shows no rule for 5432
> But port accessible from Internet!
> ```

**What I did wrong**:
> Assumed UFW would control all ports, didn't realize Docker modifies iptables directly

**Why wrong**:
Docker modifies iptables directly, bypassing UFW rules.

**Solution**:
```bash
# ✅ Option 1: Bind to localhost only
docker run -p 127.0.0.1:5432:5432 postgres
# → Only accessible from VPS, not Internet

# ✅ Option 2: Configure Docker to respect UFW
# /etc/docker/daemon.json
{
  "iptables": false
}
# Then manually manage iptables/UFW rules

# ✅ Option 3: Use Docker networks (best practice)
# Don't publish ports, use reverse proxy (Traefik)
```

**Lesson**: Docker bypasses UFW by default - always bind to localhost or configure Docker to respect firewall rules

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why is the default-deny policy critical for firewall security, and what would happen without it?</summary>

**Answer**: Default-deny (block all incoming traffic unless explicitly allowed) implements the zero-trust principle and minimizes attack surface. Without it, every service on your system would be exposed to the internet by default, including services you didn't intend to make public or don't know about.
</details>

<details>
<summary><strong>Q2:</strong> Why do you need BOTH a cloud firewall (like Hetzner Firewall) AND UFW on your VPS?</summary>

**Answer**: Defense-in-depth with two independent layers. Cloud firewall filters BEFORE packets reach your VPS (DDoS protection, persists even if VPS compromised). UFW provides granular control and is scriptable, but can be disabled if attacker gets root. Together they provide redundancy and complementary protection.
</details>

<details>
<summary><strong>Q3:</strong> What's the most critical mistake when enabling UFW for the first time, and how do you prevent it?</summary>

**Answer**: Enabling UFW before allowing your SSH port causes immediate lockout - you can't reconnect because port 22 is blocked. Always allow SSH port FIRST (`ufw allow 2222/tcp`), verify the rule is added, THEN enable UFW. Test in a new terminal while keeping current session open.
</details>

<details>
<summary><strong>Q4:</strong> Why does Docker bypass UFW rules, and what should you do about it?</summary>

**Answer**: Docker modifies iptables directly, bypassing UFW's rules. Published ports (like `-p 5432:5432`) are accessible from internet even without UFW allow rules. Solution: bind to localhost only (`-p 127.0.0.1:5432:5432`), configure Docker to respect UFW, or use Docker networks with reverse proxy instead of publishing ports.
</details>

---

## 📊 Stats

```yaml
Total time: 1h (45% assisted / 55% autonomous)
Used in: [[2025-12-vps-hetzner-init-setup/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
