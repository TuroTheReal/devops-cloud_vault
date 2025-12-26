# Linux Security & Admin - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, linux, security, hardening, admin, sysadmin]
created: 2025-12-26
updated: 2025-12-26
```

**Related**: [[ssh-hardening]], [[ufw-firewall]], [[fail2ban]]

---

## 🔐 SSH Security

### SSH Connection
```bash
# Basic connection
ssh user@host

# Specific port
ssh -p 2222 user@host

# With specific key
ssh -i ~/.ssh/id_ed25519_custom user@host

# Verbose debug
ssh -v user@host
ssh -vv user@host                 # More verbose
ssh -vvv user@host                # Maximum verbose
```

### SSH Keys
```bash
# Generate key (ED25519 - recommended)
ssh-keygen -t ed25519 -C "comment/email"

# Generate RSA key (legacy)
ssh-keygen -t rsa -b 4096 -C "comment"

# Copy key to server
ssh-copy-id user@host
ssh-copy-id -i ~/.ssh/key.pub user@host

# Manual key copy
cat ~/.ssh/id_ed25519.pub | ssh user@host "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Test key
ssh -i ~/.ssh/id_ed25519 user@host
```

### SSH Config
```bash
# Edit SSH config
nano ~/.ssh/config

# Example config
Host myserver
    HostName 91.99.92.50
    User arthur
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_hetzner

# Use config
ssh myserver                      # Uses config automatically
```

### SSH Server Hardening
```bash
# Edit SSH daemon config
sudo nano /etc/ssh/sshd_config

# Key settings to change:
Port 2222                         # Custom port
PermitRootLogin no                # Disable root login
PasswordAuthentication no         # Keys only
PubkeyAuthentication yes          # Enable keys
MaxAuthTries 3                    # Limit attempts
X11Forwarding no                  # Disable X11
PermitEmptyPasswords no           # No empty passwords

# Test config
sudo sshd -t

# Restart SSH
sudo systemctl restart ssh        # Ubuntu/Debian
sudo systemctl restart sshd       # RHEL/CentOS
```

---

## 🛡️ Firewall (UFW)

### Basic Operations
```bash
# Status
sudo ufw status                   # Basic status
sudo ufw status verbose           # Detailed status
sudo ufw status numbered          # With rule numbers

# Enable/Disable
sudo ufw enable                   # Activate firewall
sudo ufw disable                  # Deactivate

# Default policy
sudo ufw default deny incoming    # Block all incoming
sudo ufw default allow outgoing   # Allow all outgoing
```

### Allow/Deny Rules
```bash
# Allow services
sudo ufw allow ssh                # Allow SSH (port 22)
sudo ufw allow 2222/tcp           # Allow specific port
sudo ufw allow 80/tcp             # HTTP
sudo ufw allow 443/tcp            # HTTPS

# Allow from specific IP
sudo ufw allow from 192.168.1.100 to any port 2222
sudo ufw allow from 192.168.1.0/24  # Allow subnet

# Deny
sudo ufw deny 23/tcp              # Block telnet
sudo ufw deny from 1.2.3.4        # Block IP

# Delete rules
sudo ufw status numbered          # See rule numbers
sudo ufw delete 2                 # Delete rule #2
sudo ufw delete allow 80/tcp      # Delete by specification

# Reset (remove all rules)
sudo ufw reset
```

### Advanced Rules
```bash
# Rate limiting (anti brute-force)
sudo ufw limit ssh                # Max 6 connections/30sec

# Allow specific interface
sudo ufw allow in on eth0 to any port 80

# Comments
sudo ufw allow 2222/tcp comment 'SSH custom port'
```

---

## 🚫 Fail2Ban

### Installation
```bash
# Install
sudo apt install fail2ban         # Ubuntu/Debian
sudo yum install fail2ban          # RHEL/CentOS

# Start
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

### Configuration
```bash
# Copy base config
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit config
sudo nano /etc/fail2ban/jail.local

# Example SSH jail config
[sshd]
enabled = true
port = 2222
maxretry = 3
bantime = 3600
findtime = 600

# Restart
sudo systemctl restart fail2ban
```

### Management
```bash
# Status
sudo fail2ban-client status       # List active jails
sudo fail2ban-client status sshd  # Jail details

# Ban/Unban IP
sudo fail2ban-client set sshd banip 1.2.3.4
sudo fail2ban-client set sshd unbanip 1.2.3.4

# View logs
sudo tail -f /var/log/fail2ban.log
```

---

## 👤 User Management

### User Operations
```bash
# Add user
sudo adduser username             # Interactive (Debian/Ubuntu)
sudo useradd username             # Manual (need -m for home dir)
sudo useradd -m -s /bin/bash username

# Delete user
sudo userdel username             # Keep home dir
sudo userdel -r username          # Remove home dir

# Modify user
sudo usermod -aG sudo username    # Add to sudo group
sudo usermod -s /bin/bash user    # Change shell
sudo usermod -L username          # Lock account
sudo usermod -U username          # Unlock account

# Change password
sudo passwd username              # Change user password
passwd                            # Change own password
```

### Group Operations
```bash
# List groups
groups                            # Current user groups
groups username                   # User's groups
cat /etc/group                    # All groups

# Add group
sudo groupadd groupname

# Add user to group
sudo usermod -aG groupname username
sudo gpasswd -a username groupname

# Remove user from group
sudo gpasswd -d username groupname

# Delete group
sudo groupdel groupname
```

### sudo Management
```bash
# Edit sudoers (SAFE)
sudo visudo

# Give user sudo access
# Add line:
username ALL=(ALL:ALL) ALL

# Sudo without password (careful!)
username ALL=(ALL) NOPASSWD: ALL

# Limited sudo (specific commands)
username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# Check sudo access
sudo -l                           # List sudo privileges
```

---

## 🔍 Security Auditing

### Check Authentication Logs
```bash
# View auth logs
sudo tail -f /var/log/auth.log    # Ubuntu/Debian
sudo tail -f /var/log/secure      # RHEL/CentOS

# Failed login attempts
sudo grep "Failed password" /var/log/auth.log
sudo grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn

# Successful logins
sudo grep "Accepted" /var/log/auth.log
last -a                           # Login history
lastb                             # Failed login attempts
```

### Active Connections
```bash
# Who is logged in
who                               # Currently logged users
w                                 # Detailed (what they're doing)

# Active SSH connections
ss -tnp | grep :22
netstat -tnp | grep :22           # Legacy

# Last logins
last                              # Recent logins
last username                     # Specific user
last -n 10                        # Last 10 logins
```

### Open Ports
```bash
# List listening ports
sudo ss -tuln                     # Modern
sudo netstat -tuln                # Legacy

# With process names
sudo ss -tulnp
sudo netstat -tulnp

# Check specific port
sudo ss -tuln | grep :80
sudo lsof -i :80                  # What's using port 80
```

---

## 📦 Package Management Security

### Updates (Ubuntu/Debian)
```bash
# Update package list
sudo apt update

# Upgrade packages
sudo apt upgrade                  # Standard upgrade
sudo apt dist-upgrade             # Smart upgrade (handles dependencies)
sudo apt full-upgrade             # Same as dist-upgrade

# Security updates only
sudo apt upgrade -s | grep -i security
sudo unattended-upgrade --dry-run

# Autoremove unused
sudo apt autoremove
```

### Automatic Updates
```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades

# Configure
sudo dpkg-reconfigure --priority=low unattended-upgrades

# Edit config
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades

# Key settings:
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
Unattended-Upgrade::Remove-Unused-Dependencies "true";

# Check logs
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```

---

## 🔒 File Permissions & Security

### Special Permissions
```bash
# SetUID (SUID) - run as owner
chmod u+s /path/to/file
chmod 4755 file                   # -rwsr-xr-x

# SetGID (SGID) - inherit group
chmod g+s /path/to/dir
chmod 2755 dir                    # drwxr-sr-x

# Sticky bit - only owner can delete
chmod +t /path/to/dir
chmod 1777 dir                    # drwxrwxrwt (like /tmp)

# Find SUID files (security check)
find / -perm -4000 2>/dev/null    # SUID files
find / -perm -2000 2>/dev/null    # SGID files
```

### Access Control Lists (ACL)
```bash
# View ACL
getfacl file.txt

# Set ACL
setfacl -m u:username:rw file.txt     # User permission
setfacl -m g:groupname:r file.txt     # Group permission
setfacl -m o::- file.txt              # Remove others permission

# Remove ACL
setfacl -b file.txt                   # Remove all
setfacl -x u:username file.txt        # Remove specific user

# Recursive
setfacl -R -m u:username:rwx dir/
```

### File Integrity
```bash
# Generate checksums
md5sum file.txt > file.md5        # MD5 (weak)
sha256sum file.txt > file.sha256  # SHA256 (strong)

# Verify
md5sum -c file.md5
sha256sum -c file.sha256

# Find modified files
find /etc -type f -mtime -1       # Modified in last 24h
find /etc -newer /tmp/reference   # Modified after reference file
```

---

## 🔎 Process Security

### Monitor Processes
```bash
# List all processes
ps aux                            # All processes
ps aux | grep process_name        # Find specific process
pstree                            # Process tree

# Real-time monitoring
top                               # Basic
htop                              # Better (install: apt install htop)

# Memory usage
ps aux --sort=-%mem | head        # Top memory users
ps aux --sort=-%cpu | head        # Top CPU users
```

### Kill Processes
```bash
# By PID
kill PID                          # SIGTERM (graceful)
kill -9 PID                       # SIGKILL (force)

# By name
killall process_name              # Kill all instances
pkill -f pattern                  # Kill by pattern

# Interactive kill
kill -l                           # List signals
kill -SIGHUP PID                  # Reload config
```

### Process Limits
```bash
# View limits
ulimit -a                         # All limits
ulimit -n                         # Max open files

# Set limits (temporary)
ulimit -n 4096                    # Increase open files

# Permanent limits
sudo nano /etc/security/limits.conf
# Add:
username soft nofile 4096
username hard nofile 8192
```

---

## 🌐 Network Security

### Network Connections
```bash
# Active connections
ss -tunapl                        # All connections with processes
ss -tulnp                         # Listening ports

# Connection states
ss -t state established           # Established TCP connections
ss -t state listening             # Listening sockets

# Monitor traffic
sudo tcpdump -i eth0              # Capture packets
sudo tcpdump -i eth0 port 80      # HTTP traffic only
```

### DNS & Hosts
```bash
# DNS lookup
nslookup domain.com
dig domain.com
host domain.com

# Edit hosts file
sudo nano /etc/hosts
# Add:
127.0.0.1 custom.local

# Flush DNS (Ubuntu)
sudo systemd-resolve --flush-caches
```

---

## 📊 System Monitoring

### Resource Usage
```bash
# CPU
top                               # Interactive
mpstat                            # CPU stats (install: apt install sysstat)
lscpu                             # CPU info

# Memory
free -h                           # Human-readable
cat /proc/meminfo                 # Detailed info
vmstat                            # Virtual memory stats

# Disk I/O
iostat                            # I/O stats (install: apt install sysstat)
iotop                             # Per-process I/O (sudo)
```

### Logs
```bash
# System logs (systemd)
sudo journalctl                   # All logs
sudo journalctl -u ssh            # Service logs
sudo journalctl -f                # Follow (tail)
sudo journalctl -b                # Since last boot
sudo journalctl --since "1 hour ago"
sudo journalctl -p err            # Errors only

# Traditional logs
sudo tail -f /var/log/syslog      # System log
sudo tail -f /var/log/auth.log    # Authentication
sudo tail -f /var/log/kern.log    # Kernel log
```

---

## 🔧 Service Management (systemd)

### Service Control
```bash
# Status
sudo systemctl status service_name
sudo systemctl is-active service_name
sudo systemctl is-enabled service_name

# Start/Stop
sudo systemctl start service_name
sudo systemctl stop service_name
sudo systemctl restart service_name
sudo systemctl reload service_name

# Enable/Disable (start at boot)
sudo systemctl enable service_name
sudo systemctl disable service_name

# List services
systemctl list-units --type=service
systemctl list-unit-files --type=service
```

### Service Logs
```bash
# View logs
sudo journalctl -u service_name
sudo journalctl -u service_name -f        # Follow
sudo journalctl -u service_name -n 50     # Last 50 lines
sudo journalctl -u service_name --since "10 minutes ago"
```

---

## 🚨 Emergency & Recovery

### Boot Issues
```bash
# Check boot logs
sudo journalctl -b                # Current boot
sudo journalctl -b -1             # Previous boot

# Emergency mode
# Add to kernel params: systemd.unit=emergency.target
# Or: systemd.unit=rescue.target
```

### Disk Issues
```bash
# Check disk errors
sudo dmesg | grep -i error
sudo journalctl -k | grep -i error

# Filesystem check (UNMOUNTED ONLY!)
sudo fsck /dev/sda1               # Check filesystem
sudo fsck -y /dev/sda1            # Auto-fix

# Disk health (SMART)
sudo apt install smartmontools
sudo smartctl -a /dev/sda
```

### Account Lockout
```bash
# Reset via console (if locked out of SSH)
# Boot to single-user mode or use console

# Unlock user
sudo usermod -U username

# Reset password
sudo passwd username

# Check SSH config
sudo sshd -t
sudo systemctl status ssh
```

---

## 📚 Security Best Practices

### Daily Security Tasks
```bash
# Check for updates
sudo apt update && sudo apt list --upgradable

# Review failed logins
sudo grep "Failed password" /var/log/auth.log | tail -20

# Check active connections
ss -tunapl | grep ESTABLISHED

# Monitor disk space
df -h

# Check for SUID changes
find / -perm -4000 -type f 2>/dev/null > /tmp/suid-current
diff /tmp/suid-baseline /tmp/suid-current
```

### Weekly Security Tasks
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Check listening ports
sudo ss -tulnp

# Review user accounts
cat /etc/passwd | grep -v nologin

# Check cron jobs
sudo crontab -l
ls -la /etc/cron.*
```

### Monthly Security Tasks
```bash
# Full system update
sudo apt update && sudo apt dist-upgrade -y
sudo apt autoremove -y

# Review firewall rules
sudo ufw status numbered

# Check Fail2Ban bans
sudo fail2ban-client status sshd

# Backup important configs
sudo tar -czf /backup/etc-backup-$(date +%Y%m%d).tar.gz /etc
```

---

## 🎯 Quick Security Checklist

### New Server Setup
- [ ] Update system: `sudo apt update && sudo apt upgrade -y`
- [ ] Create non-root user: `sudo adduser username`
- [ ] Add to sudo: `sudo usermod -aG sudo username`
- [ ] Setup SSH keys
- [ ] Harden SSH: Port, no root, no passwords
- [ ] Enable firewall: `sudo ufw enable`
- [ ] Install Fail2Ban: `sudo apt install fail2ban`
- [ ] Setup automatic updates
- [ ] Configure backups

### After Hardening
- [ ] Test SSH access (keep session open!)
- [ ] Verify firewall rules
- [ ] Check Fail2Ban status
- [ ] Review auth logs
- [ ] Document all changes

---

**Last updated**: 2025-12-26
**Difficulty**: ⭐⭐⭐ (3/5) - Intermediate to advanced