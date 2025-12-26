# Networking Fundamentals

## 📋 Metadata

```yaml
tags: [concept, networking, fundamentals, tcp-ip, ports, status/learning]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 2h
```

**Prerequisites**: None (foundational)
**Related to**: [[docker-swarm-overlay-networks]], [[linux-firewall-ufw]]

---

## 🎯 TL;DR (30 seconds)

> Lien entre machine/docker ou toutes entites necessitant un communication, securise ou non selon la couche OSI
>
> Networking = communication entre machines via protocoles (TCP/IP). IP = adresse, Port = porte d'entrée, Firewall = contrôle accès.
>
> **Analogy**: [IP = Hotel, Port = numéro de chambre]

---

## 🤔 When to Use?

### ✅ Good for
- **Server administration**: Comprendre SSH, HTTP, services réseau
- **DevOps**: Docker networks, load balancing, reverse proxy
- **Debugging**: Troubleshoot connection issues

---

## 📚 Key Concepts (In My Own Words)

### 1. IP Addresses

**My understanding**:
IP = Identifiant unique machine sur réseau
- IPv4: 192.168.1.10 (4 octets, 32 bits)
- IPv6: 2001:0db8::1 (128 bits)
- Private ranges: 192.168.x.x, 10.x.x.x (LAN)
- Public IP: Accessible depuis Internet

---

### 2. Ports and Protocols

**My understanding**:
Port = Point d'entrée service sur machine
- Range: 0-65535
- Well-known: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
- TCP: Connection-oriented (HTTP, SSH)
- UDP: Connectionless (DNS, streaming)

---

### 3. DNS (Domain Name System)

**My understanding**:
DNS = Traduction nom domaine → IP
- `google.com` → `142.250.185.46`
- Évite mémoriser IPs
- Hiérarchie: Root → TLD (.com) → Domain (google)

---

### 4. Firewall Basics

**My understanding**:
Firewall = Filtre trafic réseau
- Allow/Deny rules par IP, port, protocol
- Stateful (track connections) vs Stateless
- Host-based (UFW) vs Network-based (router)

---

## 💻 Minimal Example

### Essential Commands

```bash
# Check network configuration
ip addr show           # IP addresses
ip route show          # Routing table

# DNS lookup
nslookup google.com    # Query DNS
dig google.com         # Detailed DNS info

# Test connectivity
ping google.com        # ICMP echo (alive?)
traceroute google.com  # Path to destination

# Check listening ports
ss -tuln               # Active TCP/UDP listeners
netstat -tuln          # Alternative (deprecated)

# Test connection to port
telnet host 80         # Test TCP connection
nc -zv host 22         # Netcat port scan

# Check firewall (UFW)
sudo ufw status        # Firewall rules
```

---

## ⚠️ Key Pitfall: Firewall Blocking Access

**Symptom**:
```bash
# Service running but not accessible
sudo systemctl status ssh  # Active
curl localhost:22          # Works
curl PUBLIC_IP:22          # Timeout
```

**Cause**: Firewall blocks port from external IPs

**Lesson**: Always check firewall when connection fails (use `sudo ufw status` or check cloud provider firewall)

---

## 📊 Stats

```yaml
Total time: 2h (25% assisted / 75% autonomous)
Status: 🟡 Learning
Used in: [[2025-12-vps-hetzner-init-setup]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
