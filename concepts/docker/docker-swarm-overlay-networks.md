# Docker Swarm - Overlay Networks

## 📋 Metadata

```yaml
tags: [concept, docker, swarm, networking, overlay, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐⭐ (4/5)
time-to-master: 6h
```

**Prerequisites**: [[docker-networking-basics]], [[docker-swarm-basics]]
**Related to**: [[traefik-swarm]], [[service-discovery]]

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

## 📚 Key Concepts (In My Own Words)

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

## 🔧 Essential Commands

```bash
# List all networks
docker network ls

# Inspect overlay network (shows containers, services, subnets)
docker network inspect glasck_public-facing

# See which containers are on a network
docker network inspect glasck_internal-backend | jq '.[0].Containers'

# Create overlay network manually
docker network create --driver overlay --attachable my-network

# Remove unused networks
docker network prune

# Debug: Attach temporary container to network
docker run -it --rm --network glasck_public-facing alpine sh

# Check network connectivity from container
docker exec <container> ping <service-name>
docker exec <container> nslookup <service-name>

# View network MTU
docker network inspect glasck_public-facing | jq '.[0].Options'

# Swarm: See network across cluster
docker network inspect glasck_public-facing --format '{{json .Scope}}'
```

---

## 🧪 Tests Done

### Understanding
- [x] Explained out loud (3 min)
- [x] 3 use cases identified (multi-host, microservices, security isolation)
- [x] Compared with [[docker-bridge-networks]]

### Practice
- [x] **Lab 1**: Glasck Swarm multi-node deployment - ✅ Success (4h)
- [x] **Lab 2**: Debug MTU fragmentation issues - ✅ Success (3h)
- [x] **Lab 3**: Traefik routing across overlay networks - ✅ Success (2h)

### Retention (Day+7)
- Date: TBD
- Result: TBD
- Score: TBD

---

## ❓ Unresolved Questions

1. **Encryption overhead on overlay networks** - Blocking: No
2. **Best practices for network segmentation in multi-tenant Swarm** - To investigate

---

## 📊 Learning Timeline

```
From Glasck (2025-12): Mastered overlay networks through production deployment
2025-12-18: Discovered MTU fragmentation issue (3h debugging)
2025-12-19: Fixed Traefik multi-network routing (2h)
2025-12-23: Documentation extraction
```

### Time Invested

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Discovery | - | 2h | 2h |
| Practice (deployment) | 1h | 4h | 5h |
| Debugging (MTU, routing) | 30min | 4h30 | 5h |
| Documentation | 30min | 1h | 1h30 |
| **TOTAL** | **2h (15%)** | **11h30 (85%)** | **13h30** |

**Ratio**: 15% ✅ (target <30%)

---

## 📝 Status

**Current status**: ✅ Mastered

---

## 🔗 Resources

### Official
- [Docker Swarm networking](https://docs.docker.com/network/overlay/)
- [VXLAN explained](https://en.wikipedia.org/wiki/Virtual_Extensible_LAN)

### Personal
- [[cheatsheet-docker-swarm]]
- [[traefik-swarm-integration]]
- Project: Glasck deployment
- Source file: [Glasck/deployment/docker/stack.vps.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.vps.yml)

---

## ✅ Mastery Checklist

### Understanding
- [x] Explain 3 min without notes
- [x] 3 concrete use cases
- [x] 4 common pitfalls (MTU, multi-network routing, external networks, firewall)
- [x] Compare with bridge networks

### Application
- [x] Production multi-node deployment
- [x] Debug network connectivity issues
- [x] Explain MTU and VXLAN overhead
- [x] Traefik integration with multiple networks

### Solidification
- [x] Complete note with links
- [x] Extract to cheatsheet
- [x] Production deployment (Glasck)
- [ ] Retention test Day+7: TBD

**Validation date**: 2025-12-23
**Total time**: 13h30

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
