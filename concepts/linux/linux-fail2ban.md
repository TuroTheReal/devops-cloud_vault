# Fail2Ban - Intrusion Prevention System

## 📋 Metadata

```yaml
tags: [concept, linux, security, fail2ban, intrusion-prevention, ips, status/learning]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 1h
```

**Prerequisites**: [[linux-basics]], [[linux-firewall-ufw]]
**Related to**: [[linux-ssh-hardening]]

---

## 🎯 TL;DR (30 seconds)

> Fail2Ban monitors logs, detects attack patterns (failed logins), bans IPs automatically with iptables.
>
> **Analogy**: Bouncer who checks if you're allowed to enter + if you came before and weren't OK, bans you

---

## 🤔 When to Use?

### ✅ Good for
- **Public-facing SSH**: Servers accessible from Internet (brute-force attacks 24/7)
- **Web applications**: Protect login pages, admin panels
- **Multi-service protection**: SSH + Apache + Nginx + Postfix simultaneously

### ❌ Bad for
- **Internal networks**: Servers behind VPN (no external attacks)
- **API keys only**: Services without password auth (nothing to brute-force)
- **High-traffic sites**: Risk of banning legitimate IPs (false positives)

---

## 📚 Key Concepts

### 1. How Fail2Ban Works

**My understanding**:
> **Checks an IP pattern and manages to define if OK or not**
>
> Workflow:
> 1. Fail2Ban monitors log files (ex: /var/log/auth.log for SSH)
> 2. Detects failure patterns (ex: "Failed password for")
> 3. Counts failures in time window (ex: 3 failures in 10min)
> 4. If threshold exceeded → ban IP with iptables/UFW
> 5. Automatic unban after timeout (ex: 1h)

**Why important**:
> **Allows final filtering after ports for remaining attacks or not**

**Mental model**:
```
Attack → Logs → Fail2Ban → Firewall Rule → IP Banned

Example:
1. Attacker tries SSH: ssh user@server (wrong password)
2. Log entry: "Failed password for user from 1.2.3.4"
3. Fail2Ban sees pattern, increments counter
4. After 3 failures → Fail2Ban runs: iptables -A INPUT -s 1.2.3.4 -j DROP
5. IP 1.2.3.4 blocked for 1 hour
```

---

### 2. Jails and Filters

**My understanding**:
> **Jails are the rule containers that define how to protect each service, filters are the patterns that detect attacks**
>
> **Jail** = Service protection configuration
> - Which log to monitor
> - Filter to use (regex pattern)
> - Action to take (ban, email alert)
> - Ban parameters (maxretry, findtime, bantime)
>
> **Filter** = Regex pattern to detect attacks
> - Example SSH: `Failed password for .* from <HOST>`
> - `<HOST>` = IP address to ban

**Configuration structure**:
```
/etc/fail2ban/
├── jail.conf         # Default config (don't edit)
├── jail.local        # Your overrides (edit this)
├── filter.d/         # Regex patterns for attack detection
│   ├── sshd.conf     # SSH filter
│   └── nginx-*.conf  # Nginx filters
└── action.d/         # Actions to take (iptables, ufw, email)
    ├── iptables.conf
    └── ufw.conf
```

**Example jail**:
```ini
[sshd]
enabled = true                 # Enable this jail
port = ssh                     # Which port to protect
filter = sshd                  # Use sshd filter (regex)
logpath = /var/log/auth.log    # Log to monitor
maxretry = 3                   # Ban after 3 failures
findtime = 600                 # Within 10 minutes
bantime = 3600                 # Ban for 1 hour
action = ufw                   # Use UFW to ban
```

---

### 3. Ban Parameters (maxretry, findtime, bantime)

**My understanding**:
> **Used to define the ban rules**
>
> **maxretry**: How many failures before ban
> **findtime**: Time window to count failures (seconds)
> **bantime**: Duration of ban (seconds, -1 = permanent)
>
> Example: `maxretry=3, findtime=600, bantime=3600`
> → 3 failures within 10 minutes = 1 hour ban

**Tuning strategies**:
```bash
# Aggressive (strict security)
maxretry = 2
findtime = 300  (5 min)
bantime = 7200  (2 hours)
→ 2 failures in 5 min = 2h ban

# Moderate (balanced - recommended)
maxretry = 3
findtime = 600  (10 min)
bantime = 3600  (1 hour)
→ 3 failures in 10 min = 1h ban

# Lenient (high false-positive risk)
maxretry = 5
findtime = 1200 (20 min)
bantime = 1800  (30 min)
→ 5 failures in 20 min = 30min ban

# Permanent ban (dangerous - can ban yourself!)
bantime = -1
```

**Why important**:
> **Without this, no ban pattern, can retry quickly == lose the benefit of fail2ban itself**

---

### 4. Actions: iptables vs UFW

**My understanding**:
> **Fail2Ban can use different methods to ban IPs - iptables is fast but invisible to UFW, while UFW action integrates better**
>
> Fail2Ban can use different actions to ban:
>
> **iptables** (default):
> - Adds rule directly to iptables
> - Fast, lightweight
> - But bypasses UFW (rules invisible in `ufw status`)
>
> **UFW**:
> - Uses `ufw deny from IP` command
> - Rules visible in `ufw status`
> - Better integration if you manage firewall via UFW
>
> Recommendation: If you use UFW → configure Fail2Ban to use UFW action

**Configuration**:
```ini
# /etc/fail2ban/jail.local
[DEFAULT]
banaction = ufw  # Use UFW instead of iptables

[sshd]
enabled = true
action = ufw     # Or specify per-jail
```

---

## 💻 Minimal Example

### Context
From VPS Hetzner hardening - Basic Fail2Ban setup for SSH protection with UFW integration.

### Code

**1. Installation**:
```bash
# Install Fail2Ban
sudo apt update
sudo apt install fail2ban

# Start and enable service
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo systemctl status fail2ban
```

**2. Configuration** (`/etc/fail2ban/jail.local`):
```ini
# Create local override (don't edit jail.conf)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Modify these sections:

[DEFAULT]
# Ban settings
bantime = 3600        # 1 hour ban
findtime = 600        # 10 minute window
maxretry = 3          # 3 failures = ban
banaction = ufw       # Use UFW instead of iptables

# Email alerts (optional)
destemail = your@email.com
sendername = Fail2Ban
action = %(action_mwl)s  # Ban + email with logs

[sshd]
enabled = true
port = 2222           # Custom SSH port
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

**3. Restart and verify**:
```bash
# Restart Fail2Ban
sudo systemctl restart fail2ban

# Check jail status
sudo fail2ban-client status
# Output:
# |- Number of jail: 1
# `- Jail list: sshd

# Check specific jail
sudo fail2ban-client status sshd
# Output:
# Status for the jail: sshd
# |- Filter
# |  |- Currently failed: 0
# |  `- Total failed: 12
# `- Actions
#    |- Currently banned: 1
#    |- Total banned: 3
#    `- Banned IP list: 192.168.1.100
```

**4. Common operations**:
```bash
# Unban IP manually
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Ban IP manually
sudo fail2ban-client set sshd banip 192.168.1.100

# View logs
sudo tail -f /var/log/fail2ban.log

# Reload configuration
sudo fail2ban-client reload
```

**5. Test Fail2Ban (from another machine)**:
```bash
# Intentionally fail SSH login 3 times
ssh user@server -p 2222
# Enter wrong password 3 times

# Check if banned
sudo fail2ban-client status sshd
# Should show your IP in "Banned IP list"

# Check UFW (if using UFW action)
sudo ufw status
# Should show: DENY from YOUR_IP
```

### Line-by-Line Explanation
```
bantime = 3600
  → IP banned for 3600 seconds (1 hour)
  → After timeout, IP automatically unbanned
  → Set to -1 for permanent ban (risky!)

findtime = 600
  → Failure counter resets every 600 seconds (10 min)
  → If 3 failures within 10 min → ban
  → If failures spread over >10 min → no ban

maxretry = 3
  → Ban after 3 failed attempts
  → Lower = stricter (risk: ban yourself)
  → Higher = lenient (more brute-force attempts allowed)

banaction = ufw
  → Use UFW to ban IPs (instead of raw iptables)
  → Banned IPs visible in `sudo ufw status`
  → Better integration if managing firewall via UFW

logpath = /var/log/auth.log
  → Which log file to monitor
  → SSH logs authentication attempts here
  → Filter regex scans this file for patterns
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Banned Yourself (Wrong Password 3 Times)

**Symptom**:
> Testing SSH login with wrong password multiple times, after 3 failures connection gets refused completely. Can't reconnect even with correct password because Fail2Ban banned your IP.
>
> Example:
> ```
> Testing SSH with wrong password
> After 3 failures, connection refused
> Can't reconnect even with correct password
> Fail2Ban banned your IP!
> ```

**What I did wrong**:
> Testing authentication without checking Fail2Ban configuration first

**Why wrong**:
> Fail2Ban doesn't distinguish you from attacker - any IP that fails authentication gets banned automatically

**Solution**:
```bash
# ✅ Option 1: Wait for bantime to expire (1 hour)

# ✅ Option 2: Unban via VPS console
# (Hetzner Cloud Console → Open web console)
sudo fail2ban-client set sshd unbanip YOUR_IP

# ✅ Option 3: Whitelist your IP (prevention)
# /etc/fail2ban/jail.local
[DEFAULT]
ignoreip = 127.0.0.1/8 YOUR_HOME_IP
```

**Time wasted**: 15-30 minutes
**Lesson**: Always whitelist your IP before testing, or keep VPS console access open as backup

---

### Pitfall 2: Fail2Ban Not Banning (Wrong Log Path)

**Symptom**:
> Fail2Ban service is running but IPs are not getting banned despite multiple failed login attempts. Checking status shows zero bans even after intentional failed logins.

**What I did wrong**:
> Assumed default log path without verifying where the system actually logs authentication attempts

**Why wrong**:
Fail2Ban monitoring wrong log file (doesn't see authentication failures).

**Solution**:
```bash
# ✅ Verify log path exists and is correct
sudo tail -f /var/log/auth.log
# Try SSH login → should see "Failed password" entries

# Check Fail2Ban is monitoring correct file
sudo fail2ban-client get sshd logpath
# Output: /var/log/auth.log

# If wrong → fix in jail.local
[sshd]
logpath = /var/log/auth.log
```

**Lesson**: Always verify log paths match your system configuration - different distros may use different locations

---

### Pitfall 3: UFW and Fail2Ban Conflict

**Symptom**:
> Fail2Ban shows banned IPs when checking status, but UFW status shows no corresponding deny rules. IPs appear to be banned by Fail2Ban but traffic still gets through to the service.

**What I did wrong**:
> Used default Fail2Ban configuration which uses iptables action, bypassing UFW entirely

**Solution**:
```bash
# ✅ Configure Fail2Ban to use UFW action
# /etc/fail2ban/jail.local
[DEFAULT]
banaction = ufw

# Restart Fail2Ban
sudo systemctl restart fail2ban

# Verify banned IPs appear in UFW
sudo ufw status
sudo fail2ban-client status sshd
# Both should show same banned IPs
```

**Lesson**: When using UFW for firewall management, configure Fail2Ban to use UFW action for consistency and visibility

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> How do the three ban parameters (maxretry, findtime, bantime) work together, and what happens if you set them incorrectly?</summary>

**Answer**: maxretry sets failure threshold, findtime defines the time window to count failures, bantime sets ban duration. Example: maxretry=3, findtime=600, bantime=3600 means "3 failures within 10 minutes = 1 hour ban". If set too aggressively (like maxretry=2, findtime=300), you risk banning yourself during legitimate troubleshooting. Too lenient allows more brute-force attempts.
</details>

<details>
<summary><strong>Q2:</strong> Why should you configure Fail2Ban to use UFW action instead of the default iptables action?</summary>

**Answer**: If you manage your firewall via UFW, using iptables action bypasses UFW - banned IPs won't appear in `ufw status`, making it invisible and confusing. Using the UFW action integrates properly so banned IPs are visible in UFW and management is consistent. Set `banaction = ufw` in jail.local.
</details>

<details>
<summary><strong>Q3:</strong> What's the danger of testing Fail2Ban with wrong passwords, and how do you recover from banning yourself?</summary>

**Answer**: Fail2Ban doesn't distinguish you from attackers - any IP that fails authentication gets banned automatically. Recovery options: 1) Wait for bantime to expire, 2) Access via VPS console and unban manually (`fail2ban-client set sshd unbanip YOUR_IP`), or 3) Prevention: whitelist your IP in jail.local with `ignoreip = YOUR_HOME_IP`.
</details>

<details>
<summary><strong>Q4:</strong> Why might Fail2Ban not be banning IPs despite multiple failed logins?</summary>

**Answer**: Most likely monitoring wrong log path - different distros use different locations. Verify with `tail -f /var/log/auth.log` that you see "Failed password" entries when testing SSH. Check Fail2Ban is monitoring correct file with `fail2ban-client get sshd logpath`. Fix by updating logpath in jail.local if wrong.
</details>

---

## 📊 Stats

```yaml
Total time: 45min (45% assisted / 55% autonomous)
Used in: [[2025-12-vps-hetzner-init-setup/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
