# MOC - Linux Security Hardening

> **What**: Secure a Linux server from initial setup to production-ready state
>
> **Who**: DevOps engineers, sysadmins managing public-facing servers
>
> **Why**: Deploy servers that resist brute-force attacks, unauthorized access, and common exploits

---

## 🎯 Learning Objective

Master the essential security practices to harden a Linux server from scratch. You'll learn layered defense: OS fundamentals, SSH hardening, firewall configuration, intrusion prevention, and systemd security. After this path, you'll confidently deploy secure VPS instances that follow industry best practices.

**Real-world use case**: Setting up a production VPS (Hetzner, AWS, DigitalOcean) that runs web applications, APIs, or databases exposed to the internet while maintaining security against automated attacks and unauthorized access attempts.

---

## 📚 Learning Path

### Phase 1: Linux Foundations (4h total)

**Goal**: Understand core Linux concepts required for secure system administration

1. **[[linux-basics]]** ⭐ (2h)
   - **Why now**: Foundation for everything - you can't secure what you don't understand
   - **You'll learn**: File system hierarchy, permissions model, package management, and why "everything is a file" matters for security

2. **[[networking-fundamentals]]** ⭐ (2h)
   - **Why now**: Security requires understanding how services communicate and expose attack surfaces
   - **You'll learn**: IP addressing, ports, protocols (TCP/UDP), DNS basics, and firewall concepts

---

### Phase 2: Access Control & Authentication (3h45 total)

**Goal**: Lock down server access and prevent unauthorized entry

3. **[[linux-ssh-hardening]]** ⭐⭐ (1h30)
   - **Why now**: SSH is your primary access point - first thing attackers target
   - **You'll learn**: Disable password auth, use SSH keys, change default port, disable root login, implement multi-factor authentication

4. **[[linux-firewall-ufw]]** ⭐⭐ (1h30)
   - **Why now**: After hardening SSH, control which ports are accessible from internet
   - **You'll learn**: UFW basics, default-deny policy, allow only necessary services, port management

5. **[[linux-fail2ban]]** ⭐⭐ (45min)
   - **Why now**: Even with hardened SSH, automated attacks persist - need dynamic IP banning
   - **You'll learn**: Monitor logs, detect brute-force patterns, automatically ban malicious IPs, configure jails and filters

---

### Phase 3: Advanced Security (2h total)

**Goal**: Implement defense-in-depth with systemd and service isolation

6. **[[linux-systemd-socket]]** ⭐⭐⭐ (2h)
   - **Why now**: After securing access, optimize how services start and isolate network exposure
   - **You'll learn**: Socket activation, on-demand service startup, reduce attack surface, file descriptor passing

---

## 🛠️ Practice Project

**Apply your knowledge**: [[2025-12-vps-hetzner-init-setup/learnings]]

**What you'll build**: Deploy a production-ready VPS from scratch with SSH hardening, UFW firewall, Fail2Ban intrusion prevention, and systemd socket activation for web services.

**Why this project**: Combines all 6 concepts in realistic scenario - exactly what you'd do for real production deployment. Tests your understanding of layered security and proper configuration order.

---

## 📊 Progress Tracking

```yaml
Total estimated time: 9h45
Difficulty: ⭐⭐ (2/5)
Prerequisites: None (foundational path)
```

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **[[MOC-Docker-Production]]**: Deploy containerized applications securely on your hardened server
- **Monitoring & Observability**: Add monitoring (Prometheus/Grafana) to track security events and system health
- **Terraform Infrastructure**: Automate server provisioning with infrastructure-as-code

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **Layered security**: No single defense is perfect - SSH keys + firewall + Fail2Ban + service isolation
2. **Default-deny**: Block everything, only allow what's necessary (UFW default policy, minimal open ports)
3. **Least privilege**: Services run as non-root users, users have minimal permissions needed
4. **Defense-in-depth**: Multiple security layers so failure of one doesn't compromise entire system

**Common pitfalls**:
- Locking yourself out (change SSH port AFTER opening it in firewall, test new SSH session before closing old one)
- Using localhost for service communication (containers on different nodes can't reach localhost)
- Forgetting to restart services after config changes (sshd, fail2ban, ufw)
- Banning your own IP with Fail2Ban (whitelist your IP in ignoreip before testing)
- Running destructive commands with sudo without verification (no undo for `rm -rf /`)

---

**Created**: 2025-12-26
**Last updated**: 2025-12-26
