# Networking Fundamentals

## 📋 Metadata

```yaml
tags: [concept, networking, fundamentals, tcp-ip, ports, status/learned]
created: 2025-12-26
updated: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 2h
```

**Prerequisites**: None (foundational)
**Related to**: [[docker-swarm-overlay-networks]], [[linux-firewall-ufw]]

---

## 🎯 TL;DR (30 seconds)

> Link between machine/docker or all entities requiring communication, secured or not depending on OSI layer
>
> Networking = communication between machines via protocols (TCP/IP). IP = address, Port = entry door, Firewall = access control.
>
> **Analogy**: IP = Hotel, Port = room number

---

## 🤔 When to Use?

### ✅ Good for
- **Server administration**: Understanding SSH, HTTP, network services
- **DevOps**: Docker networks, load balancing, reverse proxy
- **Debugging**: Troubleshoot connection issues

---

## 📚 Key Concepts

### 1. IP Addresses

**My understanding**:
IP = Unique identifier for machine on network
- IPv4: 192.168.1.10 (4 octets, 32 bits)
- IPv6: 2001:0db8::1 (128 bits)
- Private ranges: 192.168.x.x, 10.x.x.x (LAN)
- Public IP: Accessible from Internet

---

### 2. Ports and Protocols

**My understanding**:
Port = Entry point for service on machine
- Range: 0-65535
- Well-known: 22 (SSH), 80 (HTTP), 443 (HTTPS), 3306 (MySQL)
- TCP: Connection-oriented (HTTP, SSH)
- UDP: Connectionless (DNS, streaming)

---

### 3. DNS (Domain Name System)

**My understanding**:
DNS = Translation of domain name → IP
- `google.com` → `142.250.185.46`
- Avoids memorizing IPs
- Hierarchy: Root → TLD (.com) → Domain (google)

---

### 4. Firewall Basics

**My understanding**:
Firewall = Filter network traffic
- Allow/Deny rules by IP, port, protocol
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

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> What's the difference between TCP and UDP, and when would you use each?</summary>

**Answer**: TCP is connection-oriented with guaranteed delivery (HTTP, SSH) - slower but reliable. UDP is connectionless without delivery guarantee (DNS, streaming) - faster but packets can be lost. Use TCP when you need reliability, UDP when speed matters more than occasional packet loss.
</details>

<details>
<summary><strong>Q2:</strong> Why might a service be running but not accessible from the internet?</summary>

**Answer**: Firewall blocking the port from external IPs. The service might work on localhost but be blocked by host firewall (UFW) or cloud provider firewall. Always check both `systemctl status` to verify service is running AND firewall rules (`ufw status`, cloud console) to verify port is allowed.
</details>

<details>
<summary><strong>Q3:</strong> What's the purpose of DNS and why is it important for system administration?</summary>

**Answer**: DNS translates human-readable domain names to IP addresses (google.com → 142.250.185.46), avoiding the need to memorize IPs. Essential for administration because it allows service references by name instead of IP, makes configurations more maintainable, and enables services to move to different IPs without reconfiguration.
</details>

---

## 📊 Stats

```yaml
Total time: 2h (25% assisted / 75% autonomous)
Status: 🟡 Learning
Used in: [[2025-12-vps-hetzner-init-setup/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
