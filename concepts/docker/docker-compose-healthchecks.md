# Docker Compose - Healthchecks

## 📋 Metadata

```yaml
tags: [concept, docker, docker-compose, healthcheck, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐ (2/5)
time-to-master: 2h
```

**Prerequisites**: [[docker-basics]], [[docker-compose-basics]]
**Related to**: [[docker-swarm-healthchecks]], [[container-orchestration]]

---

## 🎯 TL;DR (30 seconds)

Healthchecks are automated tests that Docker runs periodically to verify if a container is actually working, not just running. They define how to check if your service is healthy and ready to handle traffic.

**Analogy**: Like a nurse checking your pulse every few minutes - the machine (container) might be on, but the healthcheck tells you if it's actually functioning properly.

---

## 🤔 When to Use?

### ✅ Good for
1. **Database containers**: Ensure DB is ready before app connects
2. **API services**: Verify endpoints respond before routing traffic
3. **Dependent services**: Coordinate startup order (with depends_on conditions)

### ❌ Bad for
- **Stateless workers** without HTTP endpoints → Resource waste
- **One-time tasks** → Use restart: "no" instead

---

## 📚 Key Concepts (In My Own Words)

### 1. Healthcheck vs Container Status

**My understanding**:
Docker has two states: container status (running/stopped) and health status (healthy/unhealthy/starting). A container can be "running" but "unhealthy" - meaning the process is alive but not functioning correctly.

**Why important**:
Without healthchecks, orchestrators like Swarm or load balancers send traffic to broken containers that are technically "running" but unable to serve requests.

**Mental model**:
- Container status = "Is the car engine on?"
- Health status = "Can the car actually drive?"

---

### 2. Healthcheck Parameters

**Test command**: The actual check to run (CMD-SHELL or CMD)
**Interval**: Time between checks (default: 30s)
**Timeout**: Max time for check to complete (default: 30s)
**Retries**: Consecutive failures before marking unhealthy (default: 3)
**Start period**: Grace period before first check (default: 0s)

**Difference with start_period vs retries**:
- `start_period`: Gives container time to boot (failures during this don't count)
- `retries`: Number of failures needed to mark as unhealthy (after start period)

---

## 💻 Minimal Example

### Context
From Transcendence project - Ensuring Elasticsearch is ready before Kibana starts, and Redis is healthy in Glasck Swarm deployment.

### Code
```yaml
# Redis with healthcheck (from Glasck stack.vps.yml)
redis-dev:
  image: redis:7-alpine
  healthcheck:
    test: ["CMD", "redis-cli", "ping"]
    interval: 5s
    timeout: 3s
    retries: 5
    start_period: 10s
  deploy:
    restart_policy:
      condition: on-failure
      delay: 5s
      max_attempts: 5

# Elasticsearch with healthcheck (from Transcendence)
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
  healthcheck:
    test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s

# API with HTTP healthcheck (from Glasck stack.dev.yml)
api-fastify-dev:
  image: glasck-api-fastify:latest
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3001/api/health"]
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 180s  # 3 minutes for app startup
```

### Line-by-Line Explanation
```
test: ["CMD", "redis-cli", "ping"]
  → Runs redis-cli ping command directly (no shell overhead)
  → Returns "PONG" if healthy, exit code 0

interval: 5s
  → Check every 5 seconds (aggressive for critical service)

timeout: 3s
  → If check takes >3s, consider it failed

retries: 5
  → Need 5 consecutive failures to mark unhealthy

start_period: 10s
  → Give Redis 10s to start before counting failures
  → Failures during this grace period don't count toward retries
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: start_period Too Short

**Symptom**:
```
Container marked unhealthy immediately after start
Service constantly restarting in Swarm
```

**What I did wrong**:
```yaml
# ❌ Wrong - Elasticsearch needs time to start
elasticsearch:
  healthcheck:
    test: ["CMD-SHELL", "curl http://localhost:9200"]
    interval: 10s
    timeout: 5s
    retries: 3
    start_period: 10s  # Too short for Elasticsearch!
```

**Why wrong**: Elasticsearch takes 40-60s to start, but healthcheck starts counting failures after only 10s. After 3 failures (~30s), container marked unhealthy and restarted.

**Solution**:
```yaml
# ✅ Correct - Give adequate startup time
elasticsearch:
  healthcheck:
    test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
    interval: 30s
    timeout: 10s
    retries: 5
    start_period: 60s  # Sufficient time for Elasticsearch boot
```

**Time wasted**: 45 min debugging restart loops
**Lesson**: Set `start_period` to 2-3x normal startup time, especially for Java apps

---

### Pitfall 2: Missing curl in Container

**Symptom**:
```
Healthcheck always fails: "curl: not found"
Container keeps restarting despite service working
```

**What I did wrong**:
```yaml
# ❌ Wrong - Alpine images don't include curl
api-fastify:
  image: node:20-alpine
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3001/health"]
```

**Why wrong**: Alpine-based images are minimal and don't include curl by default.

**Solution**:
```yaml
# ✅ Option 1: Use wget (available in Alpine)
healthcheck:
  test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3001/api/health"]

# ✅ Option 2: Install curl in Dockerfile
RUN apk add --no-cache curl

# ✅ Option 3: Use application-specific tool
healthcheck:
  test: ["CMD", "node", "healthcheck.js"]
```

**Time wasted**: 20 min
**Lesson**: Always verify health check dependencies exist in image. For Alpine, use `wget` or install `curl`.

---

### Pitfall 3: Healthcheck Too Aggressive

**Symptom**:
```
Service marked unhealthy during normal load spikes
Unnecessary restarts causing downtime
```

**What I did wrong**:
```yaml
# ❌ Wrong - Too sensitive for production
front-website:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
    interval: 5s   # Too frequent
    timeout: 2s    # Too short
    retries: 2     # Too few
```

**Why wrong**: During traffic spikes, response time can exceed 2s, causing false negatives. With only 2 retries, service marked unhealthy after just 10s of slow responses.

**Solution**:
```yaml
# ✅ Correct - Balanced for production
front-website:
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
    interval: 10s    # Check every 10s
    timeout: 5s      # Allow 5s for response
    retries: 10      # Need 10 consecutive failures (~100s)
    start_period: 120s
```

**Time wasted**: 1h debugging false positive restarts
**Lesson**: Production healthchecks should tolerate temporary slowness. Balance between quick detection and false positives.

---

## 🔧 Essential Commands

```bash
# Check container health status
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Health}}"

# View healthcheck logs for specific container
docker inspect --format='{{json .State.Health}}' <container> | jq

# See last 5 healthcheck results
docker inspect <container> | jq '.[0].State.Health.Log[-5:]'

# Force healthcheck now (useful for debugging)
docker exec <container> <healthcheck-command>

# Swarm: View service health
docker service ps <service> --format "table {{.Name}}\t{{.CurrentState}}\t{{.Error}}"
```

---

## 🧪 Tests Done

### Understanding
- [x] Explained out loud (2 min)
- [x] 3 use cases identified (DB, API, dependent services)
- [x] Compared with [[container-restart-policies]]

### Practice
- [x] **Lab 1**: Transcendence monitoring stack - ✅ Success (2h)
- [x] **Lab 2**: Glasck Swarm deployment - ✅ Success (1h30)
- [x] **Lab 3**: Debug unhealthy containers - ✅ Success (45min)

### Retention (Day+7)
- Date: TBD
- Result: TBD
- Score: TBD

---

## ❓ Unresolved Questions

1. **Best practice for healthcheck frequency in Swarm** - To investigate: [[docker-swarm-best-practices]]
2. **Healthcheck impact on performance** - Blocking: No, works well with current settings

---

## 📊 Learning Timeline

```
From Transcendence (2024): Discovered healthchecks through ELK stack
From Glasck (2025-12): Mastered Swarm healthchecks with Redis/Traefik
2025-12-23: Documentation extraction
```

### Time Invested

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Discovery (Transcendence) | 30min | 1h | 1h30 |
| Practice (Glasck) | - | 2h | 2h |
| Debugging pitfalls | - | 2h | 2h |
| Documentation | 30min | 30min | 1h |
| **TOTAL** | **1h (17%)** | **5h30 (83%)** | **6h30** |

**Ratio**: 17% ✅ (target <30%)

---

## 📝 Status

```
⏳ Discovering   - Initial exploration
🟡 Learning      - Partial understanding
🟠 Practicing    - Practice in progress
✅ Mastered      - Fully understood
🔄 Review        - Needs refresh
```

**Current status**: ✅ Mastered

---

## 🔗 Resources

### Official
- [Docker Compose healthcheck](https://docs.docker.com/compose/compose-file/compose-file-v3/#healthcheck)
- [Dockerfile HEALTHCHECK](https://docs.docker.com/engine/reference/builder/#healthcheck)

### Personal
- [[cheatsheet-docker-compose]]
- Projects: Transcendence (ELK), Glasck (Swarm)
- Source files:
  - [Transcendence/.devcontainer/docker-compose.yml](/home/artberna/abGitHub/Transcendence/.devcontainer/docker-compose.yml)
  - [Glasck/deployment/docker/stack.vps.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.vps.yml)

---

## ✅ Mastery Checklist

### Understanding
- [x] Explain 2 min without notes
- [x] 3 concrete use cases (DB, API, orchestration)
- [x] 3 common pitfalls (start_period, missing curl, aggressive timing)
- [x] Compare with restart policies

### Application
- [x] Example code from real projects
- [x] Debug unhealthy containers
- [x] Adapt to new use case (Swarm)
- [x] Explain each parameter

### Solidification
- [x] Complete note with links
- [x] Extract to cheatsheet
- [x] Real production deployments
- [ ] Retention test Day+7: TBD

**Validation date**: 2025-12-23
**Total time**: 6h30

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
