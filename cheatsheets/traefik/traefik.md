# Traefik - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, traefik, reverse-proxy, swarm]
created: 2025-12-23
updated: 2025-12-23
```

**Related**: [[concepts/traefik/traefik-swarm-integration]], [[docker-swarm]]

---

## 🚀 Quick Start

```yaml
# traefik.yml (static config)
api:
  dashboard: true

entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

providers:
  swarm:
    endpoint: "unix:///var/run/docker.sock"
    exposedByDefault: false
    watch: true
```

```yaml
# Service labels (dynamic config)
deploy:
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.myapp.rule=Host(`example.com`)"
    - "traefik.http.routers.myapp.entrypoints=websecure"
    - "traefik.http.services.myapp.loadbalancer.server.port=3000"
```

---

## 🎯 Router Configuration

### Basic Router
```yaml
labels:
  # Enable Traefik
  - "traefik.enable=true"

  # Router rule
  - "traefik.http.routers.myapp.rule=Host(`example.com`)"

  # Entry point
  - "traefik.http.routers.myapp.entrypoints=websecure"

  # TLS
  - "traefik.http.routers.myapp.tls=true"

  # Link to service
  - "traefik.http.routers.myapp.service=myapp"
```

### Multiple Hosts
```yaml
- "traefik.http.routers.myapp.rule=Host(`example.com`) || Host(`www.example.com`)"
```

### Path-Based Routing
```yaml
- "traefik.http.routers.api.rule=Host(`example.com`) && PathPrefix(`/api`)"
- "traefik.http.routers.admin.rule=Host(`example.com`) && Path(`/admin`)"
```

### Header-Based Routing
```yaml
- "traefik.http.routers.myapp.rule=Host(`example.com`) && Headers(`X-Custom-Header`, `value`)"
```

---

## 🔧 Middleware Configuration

### HTTP → HTTPS Redirect
```yaml
labels:
  # Middleware definition
  - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
  - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"

  # Apply to router
  - "traefik.http.routers.myapp-http.middlewares=redirect-https"
```

### www → non-www Redirect
```yaml
- "traefik.http.middlewares.www-redirect.redirectregex.regex=^https?://www\\.(.+)"
- "traefik.http.middlewares.www-redirect.redirectregex.replacement=https://$${1}"
- "traefik.http.middlewares.www-redirect.redirectregex.permanent=true"
```

### Rate Limiting
```yaml
- "traefik.http.middlewares.rate-limit.ratelimit.average=100"
- "traefik.http.middlewares.rate-limit.ratelimit.period=1s"
- "traefik.http.middlewares.rate-limit.ratelimit.burst=200"
```

### InFlight Requests Limit
```yaml
- "traefik.http.middlewares.inflight.inflightreq.amount=1000"
```

### Headers Modification
```yaml
# Add security headers
- "traefik.http.middlewares.secure-headers.headers.sslRedirect=true"
- "traefik.http.middlewares.secure-headers.headers.stsSeconds=31536000"
- "traefik.http.middlewares.secure-headers.headers.frameDeny=true"
- "traefik.http.middlewares.secure-headers.headers.contentTypeNosniff=true"
```

### Middleware Chaining
```yaml
- "traefik.http.routers.myapp.middlewares=www-redirect,redirect-https,rate-limit"
```

---

## ⚖️ Service (Load Balancer) Configuration

### Basic Service
```yaml
- "traefik.http.services.myapp.loadbalancer.server.port=3000"
```

### Sticky Sessions
```yaml
- "traefik.http.services.myapp.loadbalancer.sticky.cookie=true"
- "traefik.http.services.myapp.loadbalancer.sticky.cookie.name=lb_cookie"
- "traefik.http.services.myapp.loadbalancer.sticky.cookie.httpOnly=true"
- "traefik.http.services.myapp.loadbalancer.sticky.cookie.secure=true"
- "traefik.http.services.myapp.loadbalancer.sticky.cookie.sameSite=lax"
```

### Health Checks
```yaml
- "traefik.http.services.myapp.loadbalancer.healthcheck.path=/health"
- "traefik.http.services.myapp.loadbalancer.healthcheck.interval=10s"
- "traefik.http.services.myapp.loadbalancer.healthcheck.timeout=5s"
- "traefik.http.services.myapp.loadbalancer.healthcheck.scheme=http"
```

### Multi-Network Services
```yaml
# CRITICAL: Specify which network to use
- "traefik.docker.network=mystack_public-facing"
```

---

## 🔐 TLS / HTTPS Configuration

### Basic TLS
```yaml
- "traefik.http.routers.myapp.tls=true"
```

### Let's Encrypt (Static Config)
```yaml
# In traefik.yml
certificatesResolvers:
  letsencrypt:
    acme:
      email: "admin@example.com"
      storage: "/acme.json"
      httpChallenge:
        entryPoint: web
```

```yaml
# Service label
- "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
```

---

## 📝 Complete Examples

### Frontend with HTTPS
```yaml
services:
  frontend:
    image: myapp:latest
    networks:
      - public
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=mystack_public"

        # HTTP Router (redirect)
        - "traefik.http.routers.frontend-http.rule=Host(`example.com`)"
        - "traefik.http.routers.frontend-http.entrypoints=web"
        - "traefik.http.routers.frontend-http.middlewares=redirect-https"

        # HTTPS Router
        - "traefik.http.routers.frontend-https.rule=Host(`example.com`)"
        - "traefik.http.routers.frontend-https.entrypoints=websecure"
        - "traefik.http.routers.frontend-https.tls=true"
        - "traefik.http.routers.frontend-https.middlewares=rate-limit"

        # Middlewares
        - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"
        - "traefik.http.middlewares.rate-limit.ratelimit.average=50"
        - "traefik.http.middlewares.rate-limit.ratelimit.period=1s"

        # Service
        - "traefik.http.services.frontend.loadbalancer.server.port=3000"
        - "traefik.http.services.frontend.loadbalancer.sticky.cookie=true"

networks:
  public:
    driver: overlay
```

### API with Path Prefix
```yaml
deploy:
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.api.rule=Host(`example.com`) && PathPrefix(`/api`)"
    - "traefik.http.routers.api.entrypoints=websecure"
    - "traefik.http.routers.api.tls=true"
    - "traefik.http.services.api.loadbalancer.server.port=8080"

    # Strip /api prefix before forwarding
    - "traefik.http.middlewares.api-strip.stripprefix.prefixes=/api"
    - "traefik.http.routers.api.middlewares=api-strip"
```

---

## 🐳 Traefik Container Configuration

### Docker Swarm Deployment
```yaml
services:
  traefik:
    image: traefik:v3.0
    ports:
      - target: 80
        published: 80
        mode: host  # Preserve client IP
      - target: 443
        published: 443
        mode: host
      - target: 8080
        published: 8082  # Dashboard
        mode: host
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./traefik.yml:/etc/traefik/traefik.yml:ro
    networks:
      - public
    deploy:
      placement:
        constraints:
          - node.role == manager  # Access to Docker socket
      labels:
        - "traefik.enable=false"  # Don't route to itself
```

---

## 🔍 Debugging & Testing

### Dashboard
```bash
# Access dashboard
http://localhost:8080/dashboard/

# Enable in traefik.yml
api:
  dashboard: true
  insecure: true  # Remove in production
```

### Testing Routing
```bash
# Test with custom Host header
curl -H "Host: example.com" http://localhost

# Verbose output
curl -v https://example.com

# Check specific header
curl -I https://example.com

# Test rate limiting
ab -n 1000 -c 10 https://example.com/
```

### View Logs
```bash
# Docker Swarm
docker service logs traefik_reverse-proxy

# Follow logs
docker service logs -f traefik_reverse-proxy

# Filter logs
docker service logs traefik_reverse-proxy 2>&1 | grep ERROR
```

### Check Configuration
```bash
# Force reload
docker service update --force traefik_reverse-proxy

# View Traefik's detected services
# Navigate to: Dashboard → HTTP → Services

# Check which network Traefik uses
docker service inspect traefik_reverse-proxy | jq '.[0].Spec.TaskTemplate.Networks'
```

---

## 🚨 Common Issues & Solutions

### 502 Bad Gateway
```bash
# Check:
# 1. Service is running
docker service ps myapp

# 2. Correct network specified
docker service inspect myapp | jq '.[0].Spec.Labels."traefik.docker.network"'

# 3. Service port correct
docker service inspect myapp | jq '.[0].Spec.Labels."traefik.http.services.myapp.loadbalancer.server.port"'

# 4. Traefik can reach service
docker exec $(docker ps -qf "name=traefik") ping myapp
```

### Service Not Discovered
```bash
# Check:
# 1. traefik.enable=true
docker service inspect myapp | jq '.[0].Spec.Labels."traefik.enable"'

# 2. exposedByDefault setting
# In traefik.yml: exposedByDefault: false (recommended)

# 3. Service on correct network
docker service inspect myapp | jq '.[0].Spec.TaskTemplate.Networks'

# 4. Traefik logs
docker service logs traefik | grep myapp
```

### TLS Certificate Errors
```bash
# Verify certificate resolver
docker service inspect myapp | jq '.[0].Spec.Labels."traefik.http.routers.myapp-https.tls.certresolver"'

# Check ACME storage
docker exec $(docker ps -qf "name=traefik") cat /acme.json

# Force cert renewal
docker service update --force traefik
```

### Client IP Shows Internal IP
```bash
# Solution: Use mode: host for ports
ports:
  - target: 80
    published: 80
    mode: host  # Not mode: ingress
```

---

## 📚 Best Practices

1. **Network**: Always specify `traefik.docker.network` for multi-network services
2. **Ports**: Use `mode: host` to preserve client IPs
3. **Security**: Set `exposedByDefault: false` and opt-in with `traefik.enable=true`
4. **Separation**: Use separate routers for HTTP (redirect) and HTTPS (main)
5. **TLS**: Never set `tls=true` on HTTP router (port 80)
6. **Placement**: Run Traefik on manager node for Swarm provider
7. **Monitoring**: Enable dashboard for debugging (disable in production or protect)
8. **Healthchecks**: Configure both Docker healthcheck AND Traefik healthcheck

---

**Last updated**: 2025-12-23
**Source**: Extracted from Glasck deployment
