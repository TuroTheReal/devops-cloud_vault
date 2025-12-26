# Docker Swarm - Overlay Networks

## 📋 Metadata

```yaml
tags: [concept, docker, swarm, networking, overlay, status/learning]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐⭐ (4/5)
time-to-master: 6h
```

**Prerequisites**: [[docker-containers-lifecycle]]
**Related to**: [[traefik-integration]]

---

## 🎯 TL;DR (30 seconds)

Overlay networks in Docker Swarm create virtual networks that span multiple hosts, allowing containers on different machines to communicate as if they were on the same local network. They enable service discovery, load balancing, and network isolation in distributed deployments.

**Analogy**: Like a VPN for containers - containers on different physical servers can talk to each other securely through an encrypted tunnel, with automatic DNS resolution.

---

## 🤔 When to Use?

### ✅ Good for
1. **Multi-host deployments**: Services spread across multiple servers
2. **Microservices architecture**: Internal vs external network separation
3. **Zero-trust security**: Isolate backend services from public internet

### ❌ Bad for
- **Single-host docker-compose** → Use bridge networks instead
- **Host networking requirements** → Can't use overlay with `network_mode: host`
- **External service communication** → Use ingress or published ports

---

## 📚 Key Concepts

### 1. Overlay vs Bridge Networks

**My understanding**:
Bridge networks work on a single Docker host - containers connect via a virtual switch. Overlay networks extend this concept across multiple hosts using VXLAN encapsulation, creating a virtual Layer 2 network that spans the cluster.

**Why important**:
In Swarm, services can have replicas on different nodes. Without overlay networks, containers on node1 can't directly talk to containers on node2.

**Mental model**:
- Bridge network = Office LAN (single building)
- Overlay network = Corporate WAN (multiple offices connected)

---

### 2. Driver Options: MTU Configuration

**From Glasck deployment**:
```yaml
networks:
  public-facing:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"
```

**My understanding**:
MTU (Maximum Transmission Unit) defines the largest packet size. Default is 1500 bytes, but VXLAN adds 50 bytes overhead. Setting MTU to 1450 prevents fragmentation when overlay encapsulation is applied.

**Why important**:
Wrong MTU causes mysterious connectivity issues - packets work for small data but fail for large transfers. This manifests as "connection hangs" that are hard to debug.

**Difference with default**:
- Default 1500 MTU: Works locally, breaks across hosts (fragmentation)
- Custom 1450 MTU: Accounts for VXLAN overhead, prevents fragmentation

---

### 3. Attachable Networks

**My understanding**:
By default, only Swarm services can connect to overlay networks. The `attachable: true` flag allows standalone containers (via `docker run`) to also join the overlay network.

**Why important**:
Needed when you have mix of Swarm services and standalone containers that must communicate. Also useful for debugging - you can spin up a temporary container on the network.

**Use case from Glasck**:
Traefik (Swarm service) needs to route to services that might run as standalone containers during development.

---

### 4. Network Separation: Public vs Internal

**From Glasck stack.vps.yml**:
```yaml
networks:
  public-facing:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"

  internal-backend:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"
```

**My understanding**:
Zero-trust architecture - services are split across two networks:
- `public-facing`: Only Traefik and frontend services (internet-exposed)
- `internal-backend`: Backend services, databases, Redis (isolated)

Services like frontend connect to BOTH networks to bridge public and internal. This prevents direct internet access to databases.

**Mental model**:
- Public network = Storefront (customers can enter)
- Internal network = Back office (employees only)
- Frontend = Cashier (has access to both)

---

## 💻 Minimal Example

### Context
From Glasck Swarm deployment - Creating isolated networks for a web application with Traefik reverse proxy, frontend, API, and Redis.

### Code
```yaml
# Infrastructure stack (stack.vps.yml)
services:
  redis-dev:
    image: redis:7-alpine
    networks:
      - internal-backend  # Only internal access
    deploy:
      placement:
        constraints:
          - node.role == manager

  reverse-proxy:
    image: traefik:v3.0
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    networks:
      - public-facing  # Only public access
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    deploy:
      placement:
        constraints:
          - node.role == manager

networks:
  public-facing:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"

  internal-backend:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"

# Application stack (stack.dev.yml)
services:
  front-website-dev:
    image: glasck-front-website:latest
    networks:
      - internal-backend  # Access to API
      - public-facing     # Exposed via Traefik
    deploy:
      replicas: 1
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=glasck_public-facing"

  api-fastify-dev:
    image: glasck-api-fastify:latest
    networks:
      - internal-backend  # Access to Redis, isolated from internet
    environment:
      - REDIS_HOST=redis-dev  # DNS resolution via overlay network

networks:
  public-facing:
    external: true  # Use network from infrastructure stack
    name: glasck_public-facing

  internal-backend:
    external: true
    name: glasck_internal-backend
```

### Line-by-Line Explanation
```
networks:
  public-facing:
    driver: overlay
      → Create virtual network that spans Swarm nodes
      → Uses VXLAN encapsulation for cross-host communication

    attachable: true
      → Allow standalone containers to join (useful for debug)
      → Without this, only Swarm services can connect

    driver_opts:
      com.docker.network.driver.mtu: "1450"
      → Set MTU to 1450 (1500 - 50 for VXLAN overhead)
      → Prevents packet fragmentation across hosts

networks:
  public-facing:
    external: true
      → Don't create new network, use existing one
      → Infrastructure stack creates it first

    name: glasck_public-facing
      → Explicit network name (with stack prefix)
      → Swarm auto-prefixes with stack name
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Incorrect MTU Causing Hangs

**Symptom**:
```
Small HTTP requests work fine
Large requests (file uploads, big JSON) hang indefinitely
curl -v shows connection established but no data transfer
```

**What I did wrong**:
```yaml
# ❌ Wrong - Default MTU 1500 doesn't account for VXLAN
networks:
  public-facing:
    driver: overlay
    # Missing MTU configuration
```

**Why wrong**: VXLAN adds 50 bytes overhead. Packets larger than 1450 bytes get fragmented, but fragmentation often fails across networks, causing silent drops.

**Solution**:
```yaml
# ✅ Correct - Set MTU to account for VXLAN overhead
networks:
  public-facing:
    driver: overlay
    driver_opts:
      com.docker.network.driver.mtu: "1450"
```

**Time wasted**: 3h debugging "connection hangs"
**Lesson**: Always set MTU to 1450 for overlay networks to prevent fragmentation issues.

---

### Pitfall 2: Wrong Traefik Network Label

**Symptom**:
```
Traefik dashboard shows service discovered
Frontend not accessible (502 Bad Gateway)
Traefik logs: "no active backends for frontend"
```

**What I did wrong**:
```yaml
# ❌ Wrong - Missing or wrong network specification
services:
  front-website:
    networks:
      - public-facing
      - internal-backend
    deploy:
      labels:
        - "traefik.enable=true"
        # Missing: traefik.docker.network
```

**Why wrong**: Frontend is on TWO networks. Traefik doesn't know which network to use for routing. Without explicit label, it guesses wrong or uses first network (internal-backend).

**Solution**:
```yaml
# ✅ Correct - Explicitly tell Traefik which network
services:
  front-website:
    networks:
      - public-facing
      - internal-backend
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=glasck_public-facing"  # Explicit!
```

**Time wasted**: 2h
**Lesson**: When service connects to multiple overlay networks, ALWAYS specify `traefik.docker.network` label.

---

### Pitfall 3: External Network Doesn't Exist

**Symptom**:
```bash
$ docker stack deploy -c stack.dev.yml glasck
network glasck_public-facing not found
```

**What I did wrong**:
```yaml
# ❌ Wrong - Trying to use external network that doesn't exist
# In stack.dev.yml
networks:
  public-facing:
    external: true
    name: glasck_public-facing

# But stack.vps.yml (infrastructure) not deployed yet!
```

**Why wrong**: `external: true` means "use existing network". If infrastructure stack isn't deployed first, network doesn't exist.

**Solution**:
```bash
# ✅ Correct - Deploy in order

# 1. Deploy infrastructure (creates networks)
docker stack deploy -c stack.vps.yml glasck

# 2. Wait for networks to be created
docker network ls | grep glasck

# 3. Deploy application stack
docker stack deploy -c stack.dev.yml glasck
```

**Alternative - Let Swarm create networks**:
```yaml
# Or remove external: true and let each stack create its own
networks:
  public-facing:
    driver: overlay
    # No external: true
```

**Time wasted**: 30 min
**Lesson**: Deploy infrastructure stack BEFORE application stack. Verify networks exist with `docker network ls`.

---

### Pitfall 4: Services Can't Communicate Across Nodes

**Symptom**:
```
Service1 on node1 can't reach Service2 on node2
Works when both on same node
Error: "No route to host"
```

**What I did wrong**:
```bash
# ❌ Wrong - Swarm ports not open on firewall
# Firewall only allows 80, 443
# Missing Swarm communication ports!
```

**Why wrong**: Overlay networks use specific ports for inter-node communication:
- TCP/UDP 7946: Container network discovery
- UDP 4789: Overlay network traffic (VXLAN)

**Solution**:
```bash
# ✅ Correct - Open Swarm ports on all nodes

# On each node firewall
sudo ufw allow 2377/tcp   # Swarm management
sudo ufw allow 7946/tcp   # Container network discovery
sudo ufw allow 7946/udp   # Container network discovery
sudo ufw allow 4789/udp   # Overlay network traffic (VXLAN)
```

**Time wasted**: 1h30
**Lesson**: Swarm overlay networks require specific firewall rules. Open ports 2377, 7946, 4789 between nodes.

---

## 📊 Stats

```yaml
Total time: 12h (50% assisted / 50% autonomous)
Status: ✅ Mastered
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Validation date**: 2025-12-23
**Total time**: 13h30

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why do you need to set MTU to 1450 for overlay networks, and what happens without it?</summary>

**Answer**: VXLAN encapsulation adds 50 bytes overhead. Default 1500 MTU means packets larger than 1450 bytes get fragmented when encapsulated. Fragmentation often fails across networks causing silent drops. Symptoms: small requests work, large requests (uploads, big JSON) hang indefinitely. Setting MTU to 1450 prevents fragmentation.
</details>

<details>
<summary><strong>Q2:</strong> What's the difference between overlay and bridge networks, and when do you need overlay?</summary>

**Answer**: Bridge networks work on single Docker host - containers connect via virtual switch. Overlay networks span multiple hosts using VXLAN to create virtual Layer 2 network across cluster. Need overlay for Swarm when services have replicas on different nodes that must communicate directly.
</details>

<details>
<summary><strong>Q3:</strong> Why might services fail to communicate across Swarm nodes, and what ports need to be open?</summary>

**Answer**: Overlay networks require specific firewall ports for inter-node communication. Need TCP/UDP 7946 (container network discovery), UDP 4789 (VXLAN overlay traffic), TCP 2377 (Swarm management). Without these, services work on same node but get "No route to host" across nodes. Open on all node firewalls.
</details>

<details>
<summary><strong>Q4:</strong> What does `attachable: true` do on an overlay network, and why is it useful?</summary>

**Answer**: By default only Swarm services can connect to overlay networks. `attachable: true` allows standalone containers (docker run) to also join. Useful for mixed environments (services + standalone containers) and debugging - you can spin up temporary container on network to test connectivity.
</details>

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
