# MOC - Networking Fundamentals

> **What**: From OSI layers to cloud networking - essential network knowledge for DevOps
>
> **Who**: DevOps engineers, SysAdmins, anyone managing infrastructure
>
> **Why**: Troubleshoot connectivity issues, configure VPCs, understand Docker/K8s networking

---

## 🎯 Learning Objective

Master network fundamentals from Layer 3 protocols (IP, ICMP) to cloud networking (VPC, Security Groups). You'll understand how packets travel across networks, diagnose connectivity issues with ping/traceroute, and apply this knowledge to Docker networking, Kubernetes Services, and cloud infrastructure.

**Real-world use case**: Debug why your Kubernetes pod can't reach a database, configure AWS VPC subnets correctly, understand the difference between a Layer 4 vs Layer 7 load balancer, troubleshoot Docker container networking issues.

---

## 📚 Learning Path

### Phase 1: Network Basics (10h total)

**Goal**: Understand IP addresses, ports, DNS, and basic connectivity testing

1. **[[networking-fundamentals]]** ⭐⭐⭐ (10h)
   - **Why now**: Foundation - understand how machines communicate before diving deeper
   - **You'll learn**: IP addresses, ports, TCP vs UDP, DNS resolution, firewall basics
   - **Hands-on**: Use `ip addr`, `ss -tuln`, `nslookup` to inspect your system

---

### Phase 2: OSI Model & Layer 3 Protocols (3h total)

**Goal**: Master Layer 3 networking with ICMP protocol (ping, traceroute)

2. **[[networking-fundamentals#OSI Model]]** ⭐⭐ (1h)
   - **Why now**: After basic concepts, understand the OSI stack to know which layer you're troubleshooting
   - **You'll learn**: 7 layers of OSI, focus on L3 (Network), L4 (Transport), L7 (Application)
   - **Mental model**: Sending a package = Layer 7 (content) → L4 (delivery method) → L3 (routing) → L1 (truck)

3. **Practice with ping/traceroute** ⭐⭐⭐ (10h)
   - **Why now**: Apply Layer 3 knowledge with real tools
   - **You'll learn**: ICMP Echo Request/Reply, TTL manipulation, network path discovery
   - **Hands-on**:
     ```bash
     ping 8.8.8.8              # Test Layer 3 connectivity
     traceroute google.com      # Discover network path (TTL trick)
     ping -c 5 -i 0.5 1.1.1.1  # Advanced ping options
     ```
   - **Reference projects**: [[ft-ping-traceroute-learnings]] (42 School implementation)

---

### Phase 3: Docker Networking (4h total)

**Goal**: Understand container networking, bridge vs overlay networks

4. **[[docker-network-isolation]]** ⭐⭐⭐ (2h)
   - **Why now**: Containers need to communicate - understand Docker's networking stack
   - **You'll learn**: Bridge networks (single host), overlay networks (multi-host), network namespaces
   - **Connection to OSI**: Bridge = Layer 2, Overlay = Layer 3 (VXLAN encapsulation)

5. **[[docker-swarm-overlay-networks]]** ⭐⭐⭐⭐ (2h)
   - **Why now**: Production containers run on multiple hosts - need overlay networks
   - **You'll learn**: Service discovery, routing mesh, load balancing across nodes
   - **Real use**: Docker Swarm, Kubernetes pod networking uses similar concepts

---


---

## 🛠️ Practice Project

**Apply your knowledge**: Build a multi-tier application with full network stack

**What you'll build** (after Phase 3):
- Docker Compose app with isolated networks (frontend, backend, database)
- Multi-container application with bridge and overlay networks
- Network troubleshooting across Docker containers

**Why this project**: Apply Layer 3 knowledge to Docker networking - understand how containers communicate across networks

**Future expansions** (when concepts available):
- Add AWS VPC with public/private subnets (requires AWS concepts)
- Deploy to Kubernetes with Services and Ingress (requires K8s concepts)
- Add Layer 7 load balancer with Traefik

---

## 📊 Progress Tracking

```yaml
Total estimated time: 17h
Difficulty: ⭐⭐⭐⭐ (4/5)
Prerequisites: Linux basics, command line comfort
Current progress: Phase 1-2 completed (~13h), Phase 3 available
```

**Completion checklist**:
- [x] Can explain OSI layers 3, 4, 7 with examples (✅ from ft_ping/ft_traceroute)
- [x] Comfortable using ping, traceroute for Layer 3 troubleshooting (✅ implemented from scratch)
- [x] Understand ICMP protocol, TTL, RTT measurement (✅ core knowledge from projects)
- [ ] Comfortable with nc, ss, tcpdump for troubleshooting (partial)
- [ ] Understand Docker bridge vs overlay networks (available in concepts)

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **[[MOC-Docker-Production]]**: Apply network knowledge to container orchestration
- **Cloud Networking**: AWS VPC, Security Groups (when AWS concepts available)
- **Kubernetes Networking**: Pod networking, Services, Ingress (when K8s concepts available)

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **OSI Model**: L3 = IP routing, L4 = TCP/UDP ports, L7 = HTTP/DNS
2. **ICMP**: ping (Echo Request/Reply), traceroute (TTL manipulation)
3. **Docker networks**: Bridge (single host), Overlay (multi-host with VXLAN)
4. **Security**: Always check BOTH firewall rules AND Security Groups
5. **Load Balancers**: L4 = TCP/UDP port forwarding, L7 = HTTP path/header routing
6. **K8s networking**: Flat network (pod-to-pod), Services (L4 abstraction), Ingress (L7 routing)

**Common pitfalls**:
- Service running but not accessible → Check firewall rules (L3-L4)
- DNS not resolving → Check DNS configuration or /etc/resolv.conf
- Container can't reach internet → Missing route to gateway or NAT
- Docker containers can't communicate → Check network configuration (bridge vs overlay)

**Troubleshooting workflow**:
```bash
# Layer 3 - Can I reach the host?
ping <host>

# Layer 3 - What's the path?
traceroute <host>

# Layer 4 - Is the port open?
nc -zv <host> <port>

# Layer 4 - What's listening locally?
ss -tuln

# Layer 7 - Application level test
curl -v http://<host>
```

---

**Created**: 2025-12-29
**Last updated**: 2025-12-29
**Projects using this knowledge**: [[ft-ping-traceroute-learnings]], Docker Swarm deployments, K8s clusters
