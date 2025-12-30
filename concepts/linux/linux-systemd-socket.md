# systemd Socket Activation

## 📋 Metadata

```yaml
tags: [concept, linux, systemd, socket-activation, ssh, status/learning]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 1h
```

**Prerequisites**: [[linux-basics]]
**Related to**: [[linux-ssh-hardening]]

---

## 🎯 TL;DR (30 seconds)

> systemd listens on the port, starts the service only when connection arrives.
>
> **Analogy**: Socket = doorbell that turns on the lights only when someone rings

---

## 🤔 When to Use?

### ✅ Good for
- **On-demand services**: Rarely used services (on-demand start saves RAM)
- **Parallel boot**: systemd can start services in parallel (socket ready before service)
- **Socket persistence**: Socket stays active even if service crashes/restarts

### ❌ Bad for
- **Custom ports**: Socket activation complicates configuration (overrides service config)
- **High-traffic services**: Minimal overhead but exists (always-on service simpler)
- **Complex dependencies**: Debug more difficult (indirection socket → service)

---

## 📚 Key Concepts

### 1. Socket Activation Mechanism

**My understanding**:
> **systemd acts as intermediary - listens on port, wakes up service when needed, hands off the connection**
>
> How it works:
> 1. **systemd creates socket** (ex: port 22) - this is just a listening endpoint, not the actual service
> 2. **systemd listens** for incoming connections on this socket
> 3. **Connection arrives** → systemd detects activity on the socket
> 4. **systemd starts the service** - only at this moment does the actual service process start
> 5. **Connection handoff** - service receives an already-connected socket file descriptor (passed by systemd)
> 6. **Service processes request** - handles the connection normally
> 7. **Service can stop** - after handling the request, the service can exit
> 8. **Socket persists** - the socket remains active in systemd, ready for the next connection
>
> The key mechanism: systemd uses Linux socket file descriptors and passes them to the service process. The service doesn't need to create or bind the socket itself - it receives a ready-to-use, already-connected socket from systemd via environment variables and file descriptor inheritance.

**Why important**:
> **Saves resources for infrequently used services, enables faster parallel boot**

**Traditional vs Socket Activation**:
```
Traditional (ssh.service):
  Boot → service starts → binds port 22 → listens forever
  Service always running (uses RAM)
  Connection arrives → service handles directly

Socket Activation (ssh.socket + ssh.service):
  Boot → systemd creates socket → binds port 22 → systemd listens
  Service NOT running (saves RAM)
  Connection arrives → systemd starts service → passes socket to service
  Service handles request → can stop after idle
  Socket remains in systemd, ready for next connection
```

**Technical details**:
- systemd uses the `socket()` and `bind()` system calls to create the listening socket
- When a connection arrives, systemd uses `accept()` to accept the connection
- The service is started with environment variable `$LISTEN_FDS` indicating available sockets
- File descriptors start at FD 3 (0=stdin, 1=stdout, 2=stderr, 3+=sockets)
- The service reads from these pre-opened file descriptors instead of creating its own
- This is transparent to well-designed services (they check for systemd sockets first, fall back to creating their own if not present)

---

### 2. Socket vs Service Files

**My understanding**:
> **Choose your setup mode**
>
> systemd uses 2 files:
>
> **`.socket` file**: Defines listening port/address
> - `ListenStream=22` = TCP port 22
> - Activated at boot, listens for connections
>
> **`.service` file**: Defines service to start
> - Triggered by socket when connection arrives
> - Can stop after processing

**Example - SSH**:
```bash
# /lib/systemd/system/ssh.socket
[Socket]
ListenStream=22           # ← Port hardcoded here!
Accept=no

[Install]
WantedBy=sockets.target

# /lib/systemd/system/ssh.service
[Service]
ExecStart=/usr/sbin/sshd -D
# Note: No "Port" specification here, uses socket
```

**Problem**: Socket file `ListenStream=22` overrides `/etc/ssh/sshd_config Port 2222`

---

### 3. Socket Override Behavior (The SSH Trap)

**My understanding**:
> **Socket binding takes precedence over service config - the classic SSH hardening trap**
>
> The SSH hardening trap:
> 1. You modify `/etc/ssh/sshd_config` → `Port 2222`
> 2. You restart SSH service → `systemctl restart ssh`
> 3. Service restarts but `ssh.socket` still active
> 4. Socket still listens on port 22 (from `ListenStream=22`)
> 5. SSH config `Port 2222` ignored!
>
> **Root cause**: Socket activation overrides service config.

**Why important**:
> **This is the #1 pitfall when hardening SSH - wastes time debugging why port won't change**

**Detection**:
```bash
# Check if socket activation is used
systemctl status ssh
# Look for:
#   TriggeredBy: ● ssh.socket  ← Socket is triggering service!

# Check socket listening port
systemctl cat ssh.socket | grep ListenStream
# Output: ListenStream=22  ← Hardcoded port
```

**Fix**:
```bash
# Disable socket activation, use direct service
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl enable ssh.service
sudo systemctl start ssh.service

# Now sshd_config Port setting is respected
```

---

### 4. When Socket Activation Is Useful

**My understanding**:
> **Socket activation makes sense for infrequent services, but adds complexity for always-on services like SSH**
>
> Legitimate use cases:
> - Rarely used services (ex: cups printing service)
> - Parallel boot (systemd can start everything simultaneously)
> - Failover (socket persists if service crashes)
>
> But for SSH in production = unnecessary overhead (SSH runs 24/7 anyway)

**Example - Printing (CUPS)**:
```bash
# cups.socket listens on port 631
# cups.service starts only when print job arrives
# → Saves RAM when not printing
```

**Why NOT useful for SSH**:
- SSH used frequently (service would run 24/7 anyway)
- Custom port needed (socket activation complicates config)
- Security-critical (simpler = better)

---

## 💻 Minimal Example

### Context
From VPS Hetzner hardening - Debugging SSH custom port not working, discovering socket activation, disabling it to use direct service.

### Code

**Problem scenario**:
```bash
# 1. Modified SSH config
sudo nano /etc/ssh/sshd_config
# Changed: Port 2222

# 2. Test config
sudo sshd -t
# Syntax OK

# 3. Restart SSH
sudo systemctl restart ssh

# 4. Check listening ports
ss -tuln | grep ssh
# BUG: Still shows port 22, not 2222!
```

**Diagnosis**:
```bash
# Check SSH service status
sudo systemctl status ssh

# Output shows:
#   Active: active (running)
#   TriggeredBy: ● ssh.socket  ← AH-HA! Socket is active
#   Main PID: 1234 (sshd)
#   Logs: Server listening on 0.0.0.0 port 22  ← Wrong port!

# Check socket file
systemctl cat ssh.socket

# Output:
# [Socket]
# ListenStream=22  ← Hardcoded port here!
# Accept=no
```

**Solution - Disable socket activation**:
```bash
# 1. Stop socket
sudo systemctl stop ssh.socket

# 2. Disable socket (prevent boot activation)
sudo systemctl disable ssh.socket

# 3. Enable direct service
sudo systemctl enable ssh.service

# 4. Start service
sudo systemctl start ssh.service

# 5. Verify listening port
ss -tuln | grep 2222
# ✅ tcp   LISTEN 0  128  0.0.0.0:2222  0.0.0.0:*

# 6. Verify status (no socket trigger)
systemctl status ssh
# ✅ No "TriggeredBy" line
# ✅ Logs: Server listening on 0.0.0.0 port 2222
```

**Permanent fix verification**:
```bash
# Reboot server
sudo reboot

# After reboot, verify SSH listens on 2222
ss -tuln | grep sshd

# Verify socket not active
systemctl is-active ssh.socket
# Output: inactive ✅
```

### Line-by-Line Explanation
```
systemctl status ssh
  → Shows service status + triggers
  → "TriggeredBy: ssh.socket" = socket activation active
  → Indicates socket is controlling service start

systemctl cat ssh.socket
  → Shows socket unit file content
  → ListenStream=22 = port hardcoded in socket
  → This overrides sshd_config Port setting

systemctl disable ssh.socket
  → Prevents socket from starting at boot
  → Doesn't remove socket file, just disables activation
  → Must also enable ssh.service to replace it

systemctl enable ssh.service
  → Configures ssh.service to start at boot
  → Now service binds port itself (uses sshd_config)
  → No socket intermediary
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: SSH Port Not Changing Despite Correct Config

**Symptom**:
> Modified SSH config to use custom port 2222, config syntax is valid, restarted service, but port 22 still listening instead of 2222.
>
> Example:
> ```
> Modified /etc/ssh/sshd_config → Port 2222
> Config syntax valid (sshd -t OK)
> Restarted SSH service
> But ss -tuln still shows port 22
> ```

**What I did wrong**:
> Didn't check for socket activation before modifying service config

**Why wrong**:
> Socket ListenStream=22 overrides service config Port 2222

**Solution**:
```bash
# ✅ Check for socket activation BEFORE config changes
systemctl status ssh | grep TriggeredBy

# If socket active → disable it
sudo systemctl disable ssh.socket
sudo systemctl enable ssh.service
sudo systemctl restart ssh
```

**Time wasted**: 30 minutes
**Lesson**: Always check `systemctl status` for TriggeredBy before modifying service config

---

### Pitfall 2: Restarting Wrong Unit

**Symptom**:
> Restarted ssh.service but changes didn't apply because socket was still controlling the service activation.

**What I did wrong**:
> Restarted only the service without stopping/disabling the socket first

**Solution**:
```bash
# ✅ When socket activation is used, stop socket first
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl restart ssh.service

# Verify socket is inactive
systemctl is-active ssh.socket
# Output: inactive
```

**Lesson**: When socket activation is active, you must disable the socket unit, not just restart the service

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why does changing the SSH port in sshd_config sometimes not work, and how do you fix it?</summary>

**Answer**: Socket activation overrides service config - if ssh.socket is active (check with `systemctl status ssh` for "TriggeredBy: ssh.socket"), the socket's hardcoded port (ListenStream=22) takes precedence over sshd_config. Fix by disabling socket activation: `systemctl disable ssh.socket`, `systemctl enable ssh.service`, then restart.
</details>

<details>
<summary><strong>Q2:</strong> What's the benefit of socket activation, and why might it not be useful for SSH in production?</summary>

**Answer**: Socket activation saves resources for infrequently used services - systemd listens on port, starts service only when connection arrives. But for SSH in production it's unnecessary overhead because SSH runs 24/7 anyway, and custom port configuration becomes more complex. Simpler is better for security-critical services.
</details>

<details>
<summary><strong>Q3:</strong> How does socket activation actually work at the technical level?</summary>

**Answer**: systemd creates and binds the listening socket, monitors for connections. When connection arrives, systemd starts the service and passes the already-connected socket file descriptor via environment variables. The service receives ready-to-use socket instead of creating its own. Socket persists in systemd even if service stops.
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
