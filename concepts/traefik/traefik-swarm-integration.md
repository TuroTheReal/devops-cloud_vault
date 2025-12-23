# Traefik - Docker Swarm Integration & Dynamic Routing

## 📋 Metadata

```yaml
tags: [concept, traefik, reverse-proxy, docker-swarm, dynamic-routing, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐⭐ (4/5)
time-to-master: 8h
```

**Prerequisites**: [[docker-swarm-basics]], [[docker-overlay-networks]], [[http-basics]]
**Related to**: [[nginx-reverse-proxy]], [[load-balancing]]

---

## 🎯 TL;DR (30 seconds)

Traefik is a dynamic reverse proxy that automatically discovers Docker Swarm services and configures routing based on labels. Instead of manually writing config files, you declare routing rules as Docker labels, and Traefik watches the Docker API to update routes in real-time.

**Analogy**: Like a smart receptionist who automatically learns where each department is by reading door signs (labels), instead of needing a pre-written directory.

---

## 🤔 When to Use?

### ✅ Good for
1. **Microservices architectures**: Automatic service discovery and routing
2. **Dynamic environments**: Services scale up/down, Traefik adapts automatically
3. **Zero-config HTTPS**: Built-in Let's Encrypt integration

### ❌ Bad for
- **Static routing with few services** → Nginx with manual config might be simpler
- **Non-HTTP protocols** (unless using TCP/UDP routers)
- **When you need full control over every routing decision** → More rigid proxies better
- **PERMISSION** Need access to "unix:///var/run/docker.sock"
---

## 📚 Key Concepts (In My Own Words)

### 1. Traefik Architecture in Swarm

**My understanding**:
Traefik connects to Docker socket (`/var/run/docker.sock`) and watches for service changes. When a service starts with `traefik.enable=true` label, Traefik:
1. Discovers the service on the specified network
2. Reads routing labels (`traefik.http.routers.*`)
3. Creates routes, middlewares, and load balancers
4. Updates configuration without restart

**Why important**:
Traditional reverse proxies (Nginx, Apache) require manual config edits + reload for each service change. Traefik is "configuration as data" - routing rules live alongside services.

**Mental model**:
- Traditional proxy = Paper phonebook (update manually, redistribute)
- Traefik = Digital phonebook (auto-updates when contacts change)

---

### 2. Providers: Docker vs Swarm

From Glasck traefik.yml:
```yaml
providers:
  swarm:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    network: "glasck_public-facing"
    watch: true
```

**My understanding**:
- `provider: docker` → Watches standalone containers
- `provider: swarm` → Watches Swarm SERVICES (not individual tasks/containers)

**Difference**:
- Docker provider reads container labels
- Swarm provider reads SERVICE deployment labels

**Why `exposedByDefault: false`**: Security. Services opt-IN to Traefik via `traefik.enable=true` label. Without this, ALL services would be exposed.

**Why `network` specified**: When service connects to multiple networks, tells Traefik which network to use for backend communication.

---

### 3. EntryPoints: web vs websecure

From Glasck traefik.yml:
```yaml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
```

**My understanding**:
EntryPoints are listening ports. Traefik listens on:
- Port 80 (web) → HTTP traffic
- Port 443 (websecure) → HTTPS traffic

Routers then specify which entrypoint to use via labels:
```yaml
"traefik.http.routers.frontend-http.entrypoints=web"
"traefik.http.routers.frontend-https.entrypoints=websecure"
```

**Mental model**:
- EntryPoint = Door to building (80 or 443)
- Router = Elevator that takes you to correct floor (service)

---

### 4. Routers, Services, and Middlewares

**From Glasck stack.dev.yml**:
```yaml
labels:
  # ROUTER (HTTP) - Listen on port 80
  - "traefik.http.routers.frontend-http.rule=Host(`glasck.com`)"
  - "traefik.http.routers.frontend-http.entrypoints=web"
  - "traefik.http.routers.frontend-http.middlewares=redirect-to-https"
  - "traefik.http.routers.frontend-http.service=frontend"

  # ROUTER (HTTPS) - Listen on port 443
  - "traefik.http.routers.frontend-https.rule=Host(`glasck.com`)"
  - "traefik.http.routers.frontend-https.entrypoints=websecure"
  - "traefik.http.routers.frontend-https.tls=true"
  - "traefik.http.routers.frontend-https.middlewares=rate-limit,inflight"
  - "traefik.http.routers.frontend-https.service=frontend"

  # MIDDLEWARE - Redirect HTTP to HTTPS
  - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
  - "traefik.http.middlewares.redirect-to-https.redirectscheme.permanent=true"

  # SERVICE (Load Balancer) - Backend configuration
  - "traefik.http.services.frontend.loadbalancer.server.port=3000"
  - "traefik.http.services.frontend.loadbalancer.sticky.cookie=true"
  - "traefik.http.services.frontend.loadbalancer.healthcheck.path=/api/health"
```

**Component breakdown**:

**Router** = Matches incoming requests and decides where to send them
- Rule: `Host()`, `Path()`, `Headers()`, etc.
- EntryPoint: Which port to listen on
- Middlewares: Transformations to apply
- Service: Where to send matched requests

**Middleware** = Request/response transformations
- Redirect HTTP → HTTPS
- Rate limiting
- Authentication
- Header manipulation

**Service** (Traefik concept, not Docker service) = Load balancer configuration
- Backend port
- Health checks
- Sticky sessions
- Load balancing algorithm

---

### 5. Multi-Network Challenge

**From Glasck** - Service on TWO networks:
```yaml
front-website:
  networks:
    - internal-backend  # To talk to API
    - public-facing     # For Traefik routing
  deploy:
    labels:
      - "traefik.enable=true"
      - "traefik.docker.network=glasck_public-facing"  # CRITICAL!
```

**My understanding**:
When a service connects to multiple overlay networks, Traefik doesn't know which network to use for backend communication. You MUST explicitly specify via `traefik.docker.network` label.

**Why important**:
Without this label, Traefik might choose the wrong network (internal-backend), causing 502 errors because Traefik can't reach the backend.

---

## 💻 Minimal Example

### Context
From Glasck - Complete Traefik setup with Swarm, including HTTP→HTTPS redirect, rate limiting, sticky sessions, and healthchecks.

### Code

**traefik.yml** (Static configuration):
```yaml
api:
  dashboard: true
  insecure: true  # Dashboard on :8080 (use firewall for protection)

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
  traefik:
    address: ":8080"

providers:
  swarm:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false    # Opt-in security
    network: "glasck_public-facing"  # Default network
    watch: true                # Auto-update on changes

ping:
  entryPoint: traefik  # Enable /ping for healthchecks

log:
  level: INFO

accessLog:
  filePath: "/dev/stdout"
```

**stack.vps.yml** (Traefik service):
```yaml
services:
  reverse-proxy:
    image: traefik:v3.0
    ports:
      - target: 80
        published: 80
        mode: host     # Preserve client IP
      - target: 443
        published: 443
        mode: host
      - target: 8080
        published: 8082  # Dashboard
        mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro  # Read-only!
      - ../config/traefik/traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - public-facing
    extra_hosts:
      - "host.docker.internal:host-gateway"  # Access host from container
    healthcheck:
      test: ["CMD", "traefik", "healthcheck", "--ping"]
      interval: 5s
      timeout: 3s
      retries: 5
      start_period: 10s
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager  # Must run on manager for Docker socket access
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 5
      resources:
        limits:
          cpus: '1.0'
          memory: 1.5G
        reservations:
          cpus: '0.25'
          memory: 128M
      labels:
        - "traefik.enable=false"  # Don't route to Traefik itself!
```

**stack.dev.yml** (Application with Traefik labels):
```yaml
services:
  front-website-dev:
    image: glasck-front-website:latest
    networks:
      - internal-backend  # Access to API/Redis
      - public-facing     # Traefik routing
    deploy:
      replicas: 1
      labels:
        # Enable Traefik
        - "traefik.enable=true"
        - "traefik.docker.network=glasck_public-facing"

        # HTTP Router (redirect to HTTPS)
        - "traefik.http.routers.frontend-http.rule=Host(`glasck.com`) || Host(`www.glasck.com`)"
        - "traefik.http.routers.frontend-http.entrypoints=web"
        - "traefik.http.routers.frontend-http.middlewares=redirect-to-https,www-redirect"
        - "traefik.http.routers.frontend-http.service=frontend"

        # HTTPS Router (main)
        - "traefik.http.routers.frontend-https.rule=Host(`glasck.com`) || Host(`www.glasck.com`)"
        - "traefik.http.routers.frontend-https.entrypoints=websecure"
        - "traefik.http.routers.frontend-https.tls=true"
        - "traefik.http.routers.frontend-https.service=frontend"
        - "traefik.http.routers.frontend-https.middlewares=www-redirect,inflight-frontend,rate-limit-frontend"

        # Middlewares
        - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-to-https.redirectscheme.permanent=true"
        - "traefik.http.middlewares.www-redirect.redirectregex.regex=^https?://www\\.(.+)"
        - "traefik.http.middlewares.www-redirect.redirectregex.replacement=https://$${1}"
        - "traefik.http.middlewares.www-redirect.redirectregex.permanent=true"

        # Rate Limiting
        - "traefik.http.middlewares.rate-limit-frontend.ratelimit.average=50"
        - "traefik.http.middlewares.rate-limit-frontend.ratelimit.period=1s"
        - "traefik.http.middlewares.rate-limit-frontend.ratelimit.burst=200"

        # InFlight Request Limit
        - "traefik.http.middlewares.inflight-frontend.inflightreq.amount=1000"

        # Load Balancer (Service)
        - "traefik.http.services.frontend.loadbalancer.server.port=3000"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie=true"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie.name=lb_cookie"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie.httpOnly=true"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie.secure=true"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie.sameSite=lax"

        # Health Check
        - "traefik.http.services.frontend.loadbalancer.healthcheck.path=/api/health"
        - "traefik.http.services.frontend.loadbalancer.healthcheck.interval=10s"
        - "traefik.http.services.frontend.loadbalancer.healthcheck.timeout=5s"

networks:
  public-facing:
    external: true
    name: glasck_public-facing
  internal-backend:
    external: true
    name: glasck_internal-backend
```

### Line-by-Line Explanation
```
providers:
  swarm:
    endpoint: "unix:///var/run/docker.sock"
      → Connect to Docker API via Unix socket
      → Read-only mount in container for security

    exposedByDefault: false
      → Services NOT exposed unless traefik.enable=true
      → Security: opt-in model

    network: "glasck_public-facing"
      → Default network for backend communication
      → Can be overridden per-service with traefik.docker.network label

    watch: true
      → Poll Docker API for changes
      → Auto-update routes when services start/stop

"traefik.http.routers.frontend-http.rule=Host(`glasck.com`)"
  → Match requests with Host header = glasck.com
  → Backticks required for special characters

"traefik.http.routers.frontend-http.middlewares=redirect-to-https,www-redirect"
  → Chain middlewares (applied in order)
  → First redirect www → non-www, then HTTP → HTTPS

"traefik.http.services.frontend.loadbalancer.server.port=3000"
  → Backend container listening on port 3000
  → Traefik forwards matched requests to this port

"traefik.http.services.frontend.loadbalancer.sticky.cookie=true"
  → Enable session affinity
  → Same client always routed to same backend replica
  → Useful for Next.js with in-memory sessions

"traefik.http.middlewares.rate-limit-frontend.ratelimit.average=50"
  → Allow average of 50 requests/second
  → burst=200 allows temporary spikes
  → Per source IP
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Missing traefik.docker.network Label

**Symptom**:
```
Traefik dashboard shows service discovered
Frontend not accessible (502 Bad Gateway)
Traefik logs: "dial tcp: lookup frontend on 127.0.0.11:53: no such host"
```

**What I did wrong**:
```yaml
# ❌ Wrong - Service on multiple networks, no explicit network specified
front-website:
  networks:
    - internal-backend
    - public-facing
  deploy:
    labels:
      - "traefik.enable=true"
      # Missing: traefik.docker.network label!
```

**Why wrong**: Traefik picks a network (often the wrong one). It tries to reach backend via `internal-backend` network, but Traefik itself is only on `public-facing`.

**Solution**:
```yaml
# ✅ Correct - Explicitly specify network
labels:
  - "traefik.enable=true"
  - "traefik.docker.network=glasck_public-facing"
```

**Time wasted**: 2h
**Lesson**: ALWAYS set `traefik.docker.network` when service connects to multiple networks.

---

### Pitfall 2: Port Mode ingress vs host

**Symptom**:
```
Client IP always shows as 10.0.X.X (internal Swarm IP)
Rate limiting doesn't work properly (all requests look like same IP)
Access logs show gateway IP instead of real client IP
```

**What I did wrong**:
```yaml
# ❌ Wrong - Default port mode is "ingress"
reverse-proxy:
  ports:
    - "80:80"
    - "443:443"
  # Default mode: ingress (routing mesh)
```

**Why wrong**: Ingress mode routes through Swarm routing mesh. All requests appear to come from the ingress network gateway IP, not the real client.

**Solution**:
```yaml
# ✅ Correct - Use host mode to preserve client IP
reverse-proxy:
  ports:
    - target: 80
      published: 80
      mode: host     # Preserve source IP
    - target: 443
      published: 443
      mode: host
  deploy:
    placement:
      constraints:
        - node.role == manager  # Host mode requires fixed node placement
```

**Time wasted**: 1h30
**Lesson**: Use `mode: host` for Traefik to preserve client IPs. Requires running on specific node (can't use full Swarm load balancing).

---

### Pitfall 3: TLS Certificate on HTTP Router

**Symptom**:
```
HTTPS works fine
HTTP requests get certificate errors instead of redirecting
Browser shows "This site can't provide a secure connection"
```

**What I did wrong**:
```yaml
# ❌ Wrong - TLS on HTTP router
labels:
  - "traefik.http.routers.frontend-http.rule=Host(`glasck.com`)"
  - "traefik.http.routers.frontend-http.entrypoints=web"  # Port 80
  - "traefik.http.routers.frontend-http.tls=true"  # WRONG! TLS on HTTP!
```

**Why wrong**: Port 80 is HTTP (no TLS). Adding `tls=true` makes Traefik try to negotiate TLS on plaintext port.

**Solution**:
```yaml
# ✅ Correct - Separate routers for HTTP and HTTPS
labels:
  # HTTP Router - Redirect only, no TLS
  - "traefik.http.routers.frontend-http.rule=Host(`glasck.com`)"
  - "traefik.http.routers.frontend-http.entrypoints=web"
  - "traefik.http.routers.frontend-http.middlewares=redirect-to-https"

  # HTTPS Router - TLS enabled
  - "traefik.http.routers.frontend-https.rule=Host(`glasck.com`)"
  - "traefik.http.routers.frontend-https.entrypoints=websecure"
  - "traefik.http.routers.frontend-https.tls=true"
```

**Time wasted**: 45min
**Lesson**: HTTP and HTTPS need SEPARATE routers. Only the `websecure` router should have `tls=true`.

---

### Pitfall 4: Traefik Not Running on Manager Node

**Symptom**:
```
Traefik container starts but doesn't discover services
Dashboard empty
Logs: "permission denied" accessing /var/run/docker.sock
```

**What I did wrong**:
```yaml
# ❌ Wrong - Traefik can run on any node
reverse-proxy:
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
  # No placement constraints
```

**Why wrong**: Docker socket only exists on manager nodes. If Traefik runs on worker node, it can't access the socket.

**Solution**:
```yaml
# ✅ Correct - Force Traefik to manager node
reverse-proxy:
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
  deploy:
    placement:
      constraints:
        - node.role == manager
```

**Time wasted**: 30min
**Lesson**: Traefik MUST run on manager node to access Docker socket. Add placement constraint.

---

## 🔧 Essential Commands

```bash
# View Traefik dashboard
http://localhost:8082/dashboard/

# Check if service discovered by Traefik
docker service logs glasck_reverse-proxy | grep frontend

# Debug routing
curl -v http://localhost -H "Host: glasck.com"

# View Traefik configuration in dashboard
# Dashboard → HTTP → Routers/Services/Middlewares

# Force Traefik to reload config (provider change)
docker service update --force glasck_reverse-proxy

# Check which network Traefik is using
docker service inspect glasck_reverse-proxy | jq '.[0].Spec.TaskTemplate.Networks'

# View service labels
docker service inspect glasck_front-website | jq '.[0].Spec.Labels'

# Test rate limiting
ab -n 1000 -c 10 https://glasck.com/
```

---

## 🧪 Tests Done

### Understanding
- [x] Explained out loud (3 min)
- [x] 3 use cases identified
- [x] Compared with [[nginx-reverse-proxy]]

### Practice
- [x] **Lab 1**: Glasck production Traefik setup - ✅ Success (5h)
- [x] **Lab 2**: Multi-network routing debugging - ✅ Success (2h)
- [x] **Lab 3**: Rate limiting and middlewares - ✅ Success (2h)

### Retention (Day+7)
- Date: TBD
- Result: TBD
- Score: TBD

---

## 📊 Learning Timeline

```
From Glasck (2025-12): Mastered Traefik Swarm integration
2025-12-17: Discovered multi-network issue (2h debugging)
2025-12-19: Perfected middleware chaining and rate limiting
2025-12-23: Documentation extraction
```

### Time Invested

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Discovery | 1h | 2h | 3h |
| Practice (Glasck setup) | 1h | 5h | 6h |
| Debugging (4 pitfalls) | - | 5h | 5h |
| Documentation | 1h | 1h30 | 2h30 |
| **TOTAL** | **3h (18%)** | **13h30 (82%)** | **16h30** |

**Ratio**: 18% ✅ (target <30%)

---

## 📝 Status

**Current status**: ✅ Mastered

---

## 🔗 Resources

### Official
- [Traefik Docs](https://doc.traefik.io/traefik/)
- [Traefik Docker Provider](https://doc.traefik.io/traefik/providers/docker/)
- [Traefik Swarm Provider](https://doc.traefik.io/traefik/providers/swarm/)

### Personal
- [[cheatsheet-traefik]]
- [[docker-swarm-overlay-networks]]
- Project: Glasck deployment
- Source files:
  - [Glasck/deployment/config/traefik/traefik.yml](/home/artberna/abGitHub/Glasck/deployment/config/traefik/traefik.yml)
  - [Glasck/deployment/docker/stack.vps.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.vps.yml)
  - [Glasck/deployment/docker/stack.dev.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.dev.yml)

---

## ✅ Mastery Checklist

### Understanding
- [x] Explain 3 min without notes
- [x] 3 concrete use cases
- [x] 4 common pitfalls
- [x] Compare with traditional reverse proxies

### Application
- [x] Production Swarm deployment
- [x] Multi-network routing configuration
- [x] Middleware chaining (redirects, rate limiting)
- [x] Debug routing issues

### Solidification
- [x] Complete note with real examples
- [x] Extract to cheatsheet
- [x] Production deployment (Glasck)
- [ ] Retention test Day+7: TBD

**Validation date**: 2025-12-23
**Total time**: 16h30

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
