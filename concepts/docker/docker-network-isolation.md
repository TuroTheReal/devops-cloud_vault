# Docker Network Isolation - Zero-Trust Architecture

## 📋 Metadata

```yaml
tags: [concept, docker, network, security, isolation, zero-trust, status/learned]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 3h
```

**Prerequisites**: [[docker-containers-lifecycle]]
**Related to**: [[docker-swarm-overlay-networks]]

---

## 🎯 TL;DR (30 seconds)

Network isolation in Docker means segregating services into separate networks so they can't communicate unless explicitly allowed. Following zero-trust principle: database on private network, NOT accessible from internet. Only services that need DB access connect to that network. Reverse proxy on public network, backend on private network, frontend bridges both.

**Analogy**: Like office security zones - lobby (public), offices (employees only), server room (IT only). Each zone has badge readers. Frontend = receptionist (access to lobby + offices), Backend = employee (offices only), Database = servers (server room only).

---

## 🤔 When to Use?

### ✅ Good for
1. **Production environments**: Prevent unauthorized access to databases
2. **Microservices**: Each service only accesses what it needs
3. **Compliance**: PCI-DSS, HIPAA require network segmentation

### ❌ Bad for
- **Simple dev environments** → Single network is fine
- **Tightly coupled monoliths** → Everything needs everything anyway
- **Performance-critical same-host communication** → Network overhead minimal, but consider host networking

---

## 📚 Key Concepts

### 1. Zero-Trust Network Architecture

**My understanding**:
Traditional approach: "firewall at perimeter, everything inside trusted"
Zero-trust: "trust nothing, verify everything, least privilege access"

In Docker: Don't put all services on same network. Each service gets ONLY the network access it needs.

**Why important**:
If attacker compromises frontend, they shouldn't automatically have database access. Network isolation is defense-in-depth.

**Mental model**:
```
❌ BAD - Everyone on same network
[Internet] → [Nginx] ─┐
                      ├─ [default network] ─ [API] ─ [Database]
                      └─────────────────────────────┘
Problem: If Nginx compromised, attacker can reach Database directly

✅ GOOD - Segmented networks
[Internet] → [Nginx on public] ─→ [API on public + private] ─→ [DB on private]
              (external only)      (bridge both networks)        (internal only)
```

---

### 2. Network Segmentation Patterns

**From Glasck production setup**:

**Pattern 1: Public vs Internal**
```yaml
networks:
  public-facing:
    driver: overlay
    # Internet-accessible services

  internal-backend:
    driver: overlay
    internal: true  # No external access!
    # Backend services, databases
```

**Pattern 2: Service Assignment**
```yaml
services:
  # Reverse proxy - public only
  traefik:
    networks:
      - public-facing

  # Frontend - both networks (bridge)
  frontend:
    networks:
      - public-facing  # For Traefik routing
      - internal-backend  # To call API

  # API - both networks
  api:
    networks:
      - public-facing  # For Traefik routing (if needed)
      - internal-backend  # To call Redis/DB

  # Database - private only
  redis:
    networks:
      - internal-backend  # NO public access

  # Database - private only
  postgres:
    networks:
      - internal-backend  # NO public access
```

**Why this works**:
- Traefik can route to frontend (both on public-facing)
- Frontend can call API (both on internal-backend)
- API can call Redis (both on internal-backend)
- Internet CANNOT reach Redis (not on public-facing)

---

### 3. Internal Network Flag

**My understanding**:
```yaml
networks:
  backend:
    driver: bridge
    internal: true  # Disable external connectivity
```

`internal: true` means:
- Containers on this network CAN talk to each other
- Containers CANNOT reach internet
- Internet CANNOT reach containers (even with port published)

**Use case**: Database network should be internal. Even if someone accidentally publishes port (`-p 5432:5432`), internet can't reach it.

**From experience**:
```yaml
# Database network - no internet needed
database-net:
  driver: overlay
  internal: true  # DB doesn't need internet

# Worker network - needs internet for APIs
worker-net:
  driver: overlay
  # No internal flag - workers call external APIs
```

---

### 4. Network Policies (with Swarm)

**My understanding**:
In Swarm, networks are isolated by default between different stacks. Services can only communicate if:
1. On same network, OR
2. Networks explicitly made `attachable: true` and service joins it

**From Glasck**:
```yaml
# Infrastructure stack (stack.vps.yml)
networks:
  public-facing:
    driver: overlay
    attachable: true  # Allow app stack to join

# Application stack (stack.dev.yml)
networks:
  public-facing:
    external: true  # Use network from infrastructure stack
    name: glasck_public-facing
```

**Why this pattern**:
- Infrastructure stack creates base networks
- Application stack joins existing networks
- Clear separation: infra (Traefik, Redis) vs app (Frontend, API)

---

### 5. DNS and Service Discovery

**My understanding**:
Within a Docker network, services can reach each other by name (DNS). Service name = hostname.

```bash
# From frontend container on internal-backend network
curl http://api:3000/users
curl http://redis:6379

# Works! Docker's embedded DNS resolves:
# api → 172.20.0.5 (API container IP on internal-backend)
# redis → 172.20.0.3 (Redis container IP)
```

**Across networks**: If service is on multiple networks, it has different IPs on each network, but same name works.

**Security implication**: Service names are discoverable via DNS. Don't rely on "security through obscurity" - use network isolation.

---

## 💻 Minimal Example

### Context
From Glasck - Real production setup with public-facing Traefik, frontend bridging networks, API accessing Redis, and Redis completely isolated from internet.

### Code

**Infrastructure Stack** (creates networks and core services):
```yaml
# stack.vps.yml
version: '3.8'

services:
  # Reverse Proxy - Public only
  reverse-proxy:
    image: traefik:v3.0
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - public-facing  # ONLY on public network
    deploy:
      placement:
        constraints:
          - node.role == manager

  # Redis Cache - Private only
  redis-dev:
    image: redis:7-alpine
    volumes:
      - redis-data:/data
      - ./config/redis/redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    networks:
      - internal-backend  # ONLY on private network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

volumes:
  redis-data:

networks:
  # PUBLIC network - internet accessible
  public-facing:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"

  # PRIVATE network - internal only
  internal-backend:
    driver: overlay
    attachable: true
    driver_opts:
      com.docker.network.driver.mtu: "1450"
    # Consider: internal: true (prevents internet access even if port published)
```

**Application Stack** (uses existing networks):
```yaml
# stack.dev.yml
version: '3.8'

services:
  # Frontend - BRIDGES both networks
  front-website-dev:
    image: glasck-front-website:latest
    networks:
      - internal-backend  # To call API
      - public-facing     # For Traefik routing
    deploy:
      replicas: 1
      labels:
        # Traefik routing (on public-facing network)
        - "traefik.enable=true"
        - "traefik.docker.network=glasck_public-facing"  # CRITICAL!
        - "traefik.http.routers.frontend.rule=Host(`glasck.com`)"
        - "traefik.http.routers.frontend.entrypoints=websecure"
        - "traefik.http.services.frontend.loadbalancer.server.port=3000"

  # API - BRIDGES both networks
  api-fastify-dev:
    image: glasck-api-fastify:latest
    networks:
      - internal-backend  # To call Redis
      - public-facing     # For Traefik routing (if API exposed directly)
    environment:
      - REDIS_HOST=redis-dev  # DNS name resolves on internal-backend
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3001/api/health"]
      interval: 10s
      timeout: 5s
      retries: 10
    deploy:
      replicas: 1
      labels:
        # Optional: expose API directly
        - "traefik.enable=true"
        - "traefik.docker.network=glasck_public-facing"
        - "traefik.http.routers.api.rule=Host(`api.glasck.com`)"

networks:
  # Join existing networks from infrastructure stack
  public-facing:
    external: true
    name: glasck_public-facing

  internal-backend:
    external: true
    name: glasck_internal-backend
```

### Architecture Diagram
```
┌─────────────────────────────────────────────────────────────┐
│ Internet                                                      │
└───────────────────────────┬───────────────────────────────────┘
                            │
                    ┌───────▼────────┐
                    │   Traefik      │ ◄─── public-facing network only
                    │   (port 80/443)│
                    └───────┬────────┘
                            │
            ┌───────────────┼───────────────┐
            │               │               │
    ┌───────▼────────┐  ┌──▼──────────┐   │
    │   Frontend     │  │   API       │   │
    │  (port 3000)   │  │ (port 3001) │   │
    └───────┬────────┘  └──┬──────────┘   │
            │              │               │
            │   internal-backend network   │
            │              │               │
            └──────────────┼───────────────┘
                           │
                    ┌──────▼───────┐
                    │    Redis     │ ◄─── internal-backend only
                    │  (port 6379) │      (NO internet access)
                    └──────────────┘

Legend:
- public-facing: Internet → Traefik → Frontend/API
- internal-backend: Frontend/API → Redis (isolated from internet)
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Database Exposed to Internet

**Symptom**:
```
Security audit failed
PostgreSQL port 5432 accessible from internet
Database in security scan results
```

**What I did wrong**:
```yaml
# ❌ DANGEROUS - All on same network
version: '3.8'

services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    networks:
      - app-network

  api:
    image: myapi
    networks:
      - app-network

  postgres:
    image: postgres:15
    ports:
      - "5432:5432"  # Published port
    networks:
      - app-network  # SAME network as Nginx!

networks:
  app-network:
    driver: bridge
```

**Why wrong**:
1. All services on same network = lateral movement if one compromised
2. Port 5432 published = accessible from internet (even with firewall, bad practice)

**Solution**:
```yaml
# ✅ SECURE - Network isolation
services:
  nginx:
    networks:
      - public  # Internet-facing

  api:
    networks:
      - public   # For Nginx routing
      - private  # For DB access

  postgres:
    networks:
      - private  # ONLY internal
    # NO published ports!

networks:
  public:
    driver: bridge
  private:
    driver: bridge
    internal: true  # No internet access
```

**Time wasted**: 4h security remediation
**Lesson**: Databases should NEVER be on public network. Use separate private network with `internal: true`.

---

### Pitfall 2: Missing Network Specification (Multi-Network Service)

**Symptom**:
```
Traefik shows service discovered
502 Bad Gateway when accessing service
Traefik logs: "Cannot reach backend"
```

**What I did wrong**:
```yaml
# ❌ Wrong - Service on 2 networks, Traefik doesn't know which to use
frontend:
  networks:
    - public
    - private
  deploy:
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.web.rule=Host(`example.com`)"
      # Missing: traefik.docker.network label!
```

**Why wrong**: Traefik sees frontend on BOTH networks. Tries to reach it on wrong network (private), where Traefik itself isn't connected.

**Solution**:
```yaml
# ✅ Correct - Explicit network for Traefik
frontend:
  networks:
    - public
    - private
  deploy:
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=glasck_public"  # Tell Traefik which network!
      - "traefik.http.routers.web.rule=Host(`example.com`)"
```

**Time wasted**: 2h (encountered multiple times during Glasck deployment)
**Lesson**: When service on multiple networks, ALWAYS specify `traefik.docker.network` label.

---

### Pitfall 3: Internal Network Blocking Required Egress

**Symptom**:
```
Worker containers can't download data from external API
curl: Could not resolve host
apt-get update fails
```

**What I did wrong**:
```yaml
# ❌ Wrong - Internal network blocks ALL external access
services:
  worker:
    image: myworker
    networks:
      - backend

networks:
  backend:
    driver: bridge
    internal: true  # Blocks internet access!

# Worker needs to call external APIs → blocked!
```

**Why wrong**: `internal: true` means no external connectivity at all. Not even outbound.

**Solution**:
```yaml
# ✅ Option 1: Remove internal flag if workers need internet
networks:
  backend:
    driver: bridge
    # No internal flag - allows outbound internet

# ✅ Option 2: Separate networks for different needs
services:
  worker:
    networks:
      - backend  # For DB access
      - external # For API calls

networks:
  backend:
    driver: bridge
    internal: true  # DB network - no internet

  external:
    driver: bridge  # Worker network - has internet
```

**Time wasted**: 1h debugging DNS issues
**Lesson**: `internal: true` blocks ALL external access (in/out). Only use for services that truly don't need internet.

---

### Pitfall 4: Forgotten Network Cleanup

**Symptom**:
```
docker network create fails: "network already exists"
Old networks accumulating
docker network ls shows 50+ networks
```

**What I did wrong**:
```bash
# Deploy, test, remove stack, repeat...
docker stack deploy -c stack.yml myapp
docker stack rm myapp

# But networks persist!
docker network ls | grep myapp
# myapp_public-facing
# myapp_internal-backend
# ... dozens more from testing
```

**Why wrong**: `docker stack rm` doesn't remove networks if any container (even stopped) is still attached.

**Solution**:
```bash
# ✅ Full cleanup
docker stack rm myapp

# Wait for services to stop
sleep 10

# Remove networks
docker network rm myapp_public-facing myapp_internal-backend

# Or prune all unused networks
docker network prune -f

# Automated cleanup script
#!/bin/bash
STACK_NAME=$1
docker stack rm $STACK_NAME
sleep 10
docker network ls --filter name=${STACK_NAME}_ -q | xargs -r docker network rm
```

**Time wasted**: 30min manual cleanup
**Lesson**: Networks persist after stack removal. Automate cleanup or use `docker network prune` regularly.

---

## 📊 Stats

```yaml
Total time: 8h (40% assisted / 60% autonomous)
Status: ✅ Mastered
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Validation date**: 2025-12-23
**Total time**: 8h

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why is zero-trust network architecture important, and how does it differ from traditional perimeter security?</summary>

**Answer**: Traditional approach trusts everything inside the perimeter - if attacker compromises one service, they can access everything. Zero-trust assumes "trust nothing, verify everything" - each service gets ONLY the network access it needs. If frontend is compromised, attacker still can't reach database because they're on different networks.
</details>

<details>
<summary><strong>Q2:</strong> What's the purpose of having services connect to multiple networks, and which services typically do this?</summary>

**Answer**: Bridge services connect to both public and private networks to route traffic between isolated zones. Frontend/API connect to public-facing (for Traefik routing) AND internal-backend (to access databases). This creates controlled entry points while keeping sensitive services (databases, Redis) completely isolated from internet.
</details>

<details>
<summary><strong>Q3:</strong> What does `internal: true` network flag do, and when should you use it?</summary>

**Answer**: Prevents external connectivity - containers on this network can talk to each other but CANNOT reach internet and internet CANNOT reach them (even with published ports). Use for database networks that don't need internet access. Provides extra security layer - even accidental port publishing won't expose service.
</details>

<details>
<summary><strong>Q4:</strong> Why might Traefik show 502 Bad Gateway for a service on multiple networks?</summary>

**Answer**: Service on two networks but missing `traefik.docker.network` label. Traefik doesn't know which network to use for backend routing, may choose wrong one (private network where Traefik isn't connected). Always set `traefik.docker.network=stackname_public-facing` label to explicitly specify routing network.
</details>

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
