# VPS Production Hardening - Hetzner CX43

## 📋 Metadata

```yaml
tags: [project, linux, security, vps, hetzner, ssh-hardening, 2025-12]
started: 2025-12-26
completed: 2025-12-26
duration: 2h30
status: completed
```

**Technologies**: Ubuntu 24.04 LTS, UFW, Fail2Ban, SSH, Docker, Unattended-upgrades

**Goal**: Initialize, secure and prepare a production-ready VPS for dev/test multi-project environment

---

## 🎯 Project Context

### Objective

Setup a secure production-grade VPS to host multiple isolated projects:

**Initial need:**
- Current VPS (16GB) saturated with production site
- Need dedicated VPS for dev/test/school projects
- Multi-container Docker environment
- Stack: web apps, databases, monitoring, various services

**Constraints:**
- Budget: <20€/month including backups
- Security: 24/7 public Internet exposure
- Flexibility: Targeted Access
- Performance: Support 15 simultaneous containers + monitoring

### Architecture

**VPS Specs:**
```
Provider: Hetzner Cloud
Plan: CX43 (Shared vCPU - Cost Optimized)
CPU: 8 vCPU (shared)
RAM: 16GB
Storage: 160GB SSD
Location: Falkenstein (fsn1)
```

**Cost:**
```
VPS CX43: 10.8€/month
Auto backup: +2.5€/month
Total: 13.3€/month (159.6€/year)
```

**Multi-layer security:**
```
Internet → Hetzner Firewall (optional) → UFW (deny-all + port 2222)
  → SSH Hardened (keys only, custom port, no root)
  → Fail2Ban (ban after 3 attempts)
  → Docker (container isolation)
  → Applications
```

### Tech Stack

- **OS**: Ubuntu 24.04 LTS
- **Security**: OpenSSH (ED25519 keys), UFW, Fail2Ban, Unattended-upgrades
- **Containers**: Docker Engine 24.x, Docker Compose v2
- **Monitoring** (planned): Prometheus, Grafana, Loki

---

## 📚 What I Learned

### 1. Complete SSH Hardening
- Difficulty: ⭐⭐⭐ (3/5)
- Time: 1h20

**Key learnings:**
- SSH hardening = multi-layer security (port + keys + fail2ban)
- Default SSH = vulnerable (port 22, passwords accepted, root accessible)
- Hardening layers: custom port → keys only → no root → rate limiting → auto-ban
- VPS on Internet = attacked permanently (hundreds of attempts/day)

**Critical insight:**
`ssh.socket` systemd overrides SSH config → Must disable socket to use custom port

**Security layers:**
1. Custom port (avoid 90% automated scans)
2. Keys only (impossible to brute force)
3. No root (isolation)
4. Fail2Ban (ban after 3 failed attempts)

---

### 2. Firewall Defense-in-Depth
- Difficulty: ⭐⭐ (2/5)
- Time: 30min

**Key concept:**
UFW = simplified iptables frontend. Principle: deny-all by default, explicitly allow only necessary ports.

**Critical rule:** Always allow SSH port BEFORE enabling firewall (avoid lockout)

**Hetzner Firewall vs UFW:**
- Hetzner: Filters BEFORE packets reach VPS (DDoS protection, persists if VPS compromised)
- UFW: Granular control, scriptable (attacker can disable if root compromised)
- Both = defense-in-depth (optional but better)

---

### 3. Fail2Ban Intrusion Prevention
- Difficulty: ⭐⭐ (2/5)
- Time: 20min

**How it works:**
Monitors logs (auth.log), detects attack patterns (failed login attempts), auto-bans IPs with iptables.

**Optimal SSH config:**
```
maxretry = 3 attempts
findtime = 600 sec (10 min)
bantime = 3600 sec (1h)
→ 3 failures in 10 min = 1h ban
```

Even with hardened SSH, Fail2Ban limits reconnaissance and exploitation attempts.

---

### 4. systemd Socket Activation
- Difficulty: ⭐⭐⭐ (3/5)
- Time: 15min (debugging)

**Problem discovered:**
Ubuntu uses `ssh.socket` which defines listening port and overrides `/etc/ssh/sshd_config`.

Modified `Port 2222` in config but SSH still listened on port 22 because `ssh.socket` had `ListenStream=22` hardcoded.

**Solution:**
```bash
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

## ⚠️ Challenges & Solutions

### Challenge 1: VPS Sizing - Specs vs Budget

**Problem**: Choose optimal specs without over/under-provisioning

**Context**: Need 15 simultaneous containers + monitoring (Redis + PostgreSQL + Prometheus + Grafana = RAM-hungry)

**Root Cause**: Underestimated real RAM needs with full monitoring stack.

**Realistic projection:**
```
Apps: 2-3GB
Databases: 1.5-2GB
Redis: 800MB-1GB
Monitoring: 1.5-2GB
OS + Docker: 700MB
Builds (peaks): 1-2GB
Total: 7-10GB with peaks

CX33 (8GB): Too tight, OOM risk
CX43 (16GB): Comfortable, +7€ = excellent ROI
```

**Decision:** CX43 + auto backup = 13.3€/month

**Lesson:** Always project **worst-case** usage (all containers active + monitoring + build). Underestimate RAM = daily frustration. CX33→CX43 price difference (+7€) = negligible vs comfort gained.

**Time spent**: 30min

---

### Challenge 2: SSH Port Not Changing - systemd Socket Override

**Problem**: SSH doesn't listen on port 2222 despite correct config

**Context**: Modified `/etc/ssh/sshd_config` with `Port 2222`, syntax OK, service restarted, but `ss -tuln` shows port 22 still active.

**Symptom:**
```bash
sudo systemctl status ssh
# Showed: TriggeredBy: ● ssh.socket
# Logs: sshd[3972]: Server listening on 0.0.0.0 port 22.
```

**Root Cause**: Ubuntu uses `ssh.socket` (systemd socket activation) which defines listening port and overrides `sshd_config`.

**Solution:**
```bash
sudo systemctl stop ssh.socket
sudo systemctl disable ssh.socket
sudo systemctl stop ssh
sudo systemctl enable ssh
sudo systemctl start ssh
sudo ss -tuln | grep 2222  # Verify
```

**Lesson**: Always check which mechanism starts the service (socket vs direct). If unexpected behavior, check `systemctl status` for "TriggeredBy".

**Time wasted**: 15min

---

### Challenge 3: Firewall Lockout Risk Prevention

**Impact**: 0 (avoided through strict methodology)

**Potential risk:**
```
Wrong order:
1. ufw enable (blocks everything)
2. ufw allow 2222 (too late, already locked out)

Or:
1. Change SSH port → 2222
2. Restart SSH
3. Forget to open port 2222 in firewall
4. Disconnect → Can't reconnect
```

**Prevention workflow used:**
```
MANDATORY ORDER:
1. Allow new port BEFORE firewall activation
   sudo ufw allow 2222/tcp
2. Enable firewall
   sudo ufw enable
3. Configure SSH + restart
4. TEST new connection (Terminal 2)
   ssh -p 2222 user@IP
5. IF test OK → Close old port
6. ALWAYS keep current session open
```

**Safeguards:**
- ✅ Always 2 terminals (1 active session + 1 test)
- ✅ Test config before restart (`sudo sshd -t`)
- ✅ Hetzner Firewall GUI as backup
- ✅ VNC Console accessible (last resort)

**Lesson**: Firewall/SSH modifications = **high-risk**. Always follow strict workflow. One open SSH session = lifesaver during modifications.

---

## 📊 Time Breakdown

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Evaluation & specs | 20min | 10min | 30min |
| VPS creation + SSH keys | 5min | 5min | 10min |
| SSH Hardening | 30min | 50min | 1h20 |
| Troubleshooting (socket) | 10min | 5min | 15min |
| Testing & validation | - | 20min | 20min |
| Documentation | 5min | 10min | 15min |
| **TOTAL** | **1h10 (47%)** | **1h40 (53%)** | **2h50** |

**Assisted/Autonomous Ratio**: 47% (target <30% exceeded but acceptable for new concept discovery)

---

## 🎯 Outcomes & Results

### Success Metrics

- ✅ **VPS operational**: Created and accessible in 10min
- ✅ **Production security**: Complete hardening in 1h20
- ✅ **Zero lockout**: Safe workflow followed strictly
- ✅ **Budget respected**: 13.3€/month (target <15€)
- ✅ **Performance validated**: 16GB comfortable for planned usage
- ✅ **Multi-machine access**: SSH keys configured (school + home future)

### What Worked Well

**1. Strict hardening methodology:**
- Double terminal (active session + test)
- Test before commit (sshd -t, new connection)
- Operations order respected
→ Zero incidents, zero lockout

**2. CX43 choice:**
- 16GB comfortable for 15 containers + monitoring
- Shared CPU OK for dev/test
- Auto backup = peace of mind

**3. Documentation as we go:**
- Notes during setup
- Immediate challenge documentation
→ Clear, reusable feedback

### What Could Be Improved

**1. Systemd socket anticipation:**
- Didn't anticipate socket activation
→ Action: Check `systemctl status` BEFORE config modifications

**2. Automation script:**
- Hardening done manually
→ Action: Create `vps-hardening.sh` script for reuse

---

## 💡 Key Takeaways

### Technical Insights

**1. systemd socket complicates SSH hardening:**
Ubuntu uses `ssh.socket` by default which overrides `sshd_config`. For custom port, disable socket and use direct service. Check `systemctl status` for "TriggeredBy" indicator.

**2. Defense-in-depth > single layer:**
Firewall + SSH hardening + Fail2Ban + auto-updates = independent layers. One layer bypassed ≠ total compromise.

**3. Shared CPU VPS sufficient for dev/test:**
CX43 shared = 16GB for 10.8€. Excellent performance for multiple containers. Dedicated CPU only useful for high-load production.

### Process Improvements

**1. Double terminal = lockout insurance:**
Always keep SSH session open during firewall/SSH modifications. Test new connection in terminal 2 BEFORE closing terminal 1.

**2. Test config BEFORE restart:**
`sudo sshd -t` catches SSH config errors. `sudo ufw status` verifies rules before enable. Testing = 30 seconds, lockout recovery = 30 minutes.

**3. Document challenges immediately:**
Note problem + solution as soon as resolved. Waiting until project end = forgotten details.

### Personal Growth

**Confidence before:**
- VPS security: ⭐⭐ (1/5)
- SSH hardening: ⭐ (1/5)
- systemd: ⭐⭐ (2/5)

**Confidence after:**
- VPS security: ⭐⭐⭐ (3/5)
- SSH hardening: ⭐⭐⭐ (3/5)
- systemd: ⭐⭐⭐ (3/5)

**New skills:**
- Complete SSH hardening workflow
- systemd service/socket management
- Advanced UFW configuration
- Fail2Ban setup and debugging
- VPS sizing and cost optimization
- Safe workflow for critical modifications

---

## 📝 Extractions TODO

- [ ] Extract [[concepts/linux/linux-ssh-hardening]] (30 min)
  - SSH key generation, config hardening, port change, socket vs service

- [ ] Extract [[concepts/linux/linux-firewall-ufw]] (30 min)
  - UFW basics, rules management, defense-in-depth with cloud firewall

- [ ] Extract [[concepts/linux/linux-systemd-socket]] (20 min)
  - Socket activation, override behavior, troubleshooting

- [ ] Extract [[concepts/linux/linux-fail2ban]] (20 min)
  - Intrusion prevention, jail configuration, monitoring

- [ ] Update [[cheatsheets/linux/security-admin-commands]] (15 min)
  - Add SSH hardening commands, UFW, Fail2Ban

**Total extraction time**: ~2h (5 concepts + 1 cheatsheet)

---

**Last updated**: 2025-12-26
**Version**: 1.0
**Next review**: 2026-01-26