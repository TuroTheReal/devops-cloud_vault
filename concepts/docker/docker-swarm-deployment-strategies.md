# Docker Swarm - Deployment Strategies (Update & Rollback)

## 📋 Metadata

```yaml
tags: [concept, docker, swarm, deployment, zero-downtime, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 4h
```

**Prerequisites**: [[docker-swarm-basics]], [[docker-swarm-healthchecks]]
**Related to**: [[kubernetes-rolling-updates]], [[blue-green-deployment]]

---

## 🎯 TL;DR (30 seconds)

Swarm deployment strategies control HOW service updates happen - rolling updates replace containers gradually to avoid downtime, while automatic rollback reverts to previous version if health checks fail. You configure update speed, failure tolerance, and rollback behavior to balance deployment safety with speed.

**Analogy**: Like replacing airplane engines mid-flight - you replace one engine at a time, verify it works, then move to the next. If something fails, you switch back to the old engine immediately.

---

## 🤔 When to Use?

### ✅ Good for
1. **Production deployments**: Zero-downtime updates for user-facing services
2. **Stateless applications**: Easy to replace replicas without data loss
3. **High-availability services**: Need continuous availability during updates

### ❌ Bad for
- **Database migrations** → May require downtime for schema changes
- **Stateful services with local data** → Use StatefulSets or manual coordination
- **Breaking API changes** → Need blue-green or canary deployment

---

## 📚 Key Concepts (In My Own Words)

### 1. Rolling Update Workflow

**My understanding**:
Swarm updates services by gradually replacing old containers with new ones:
1. Start new container with new image
2. Wait for it to pass healthcheck
3. Route traffic to new container
4. Stop old container
5. Repeat for next replica

**Why important**:
Without rolling updates, you'd have to stop ALL containers, deploy new ones, causing downtime. Rolling updates maintain service availability.

**Mental model**:
- `parallelism: 1` = Replace ONE container at a time (safest, slowest)
- `parallelism: 2` = Replace TWO containers simultaneously (faster, riskier)

---

### 2. Update Config Parameters

From Glasck stack.dev.yml:
```yaml
deploy:
  update_config:
    parallelism: 1          # Update 1 replica at a time
    delay: 10s              # Wait 10s between each update
    failure_action: rollback # Auto-rollback on failure
    monitor: 45s            # Monitor for 45s after update
    max_failure_ratio: 0    # Any failure triggers rollback
    order: start-first      # Start new before stopping old
```

**Parameter breakdown**:

- **parallelism**: How many replicas to update simultaneously
- **delay**: Pause between batches (allows monitoring)
- **failure_action**: What to do on failure (rollback/pause/continue)
- **monitor**: How long to watch new containers for failures
- **max_failure_ratio**: % of failures allowed before triggering failure_action
- **order**: start-first (new+old coexist) vs stop-first (stop old, start new)

---

### 3. Order: start-first vs stop-first

**start-first** (Recommended for most cases):
```yaml
update_config:
  order: start-first
```
- Starts new replica BEFORE stopping old one
- Temporarily have N+1 replicas running
- Zero downtime (old serves traffic while new starts)
- Requires extra resources

**stop-first**:
```yaml
update_config:
  order: stop-first
```
- Stops old replica BEFORE starting new one
- Always exactly N replicas
- Brief capacity reduction (N-1 during update)
- Saves resources

**My choice from Glasck**: `start-first` because avoiding downtime > saving resources

---

### 4. Automatic Rollback

**From Glasck deployment**:
```yaml
deploy:
  update_config:
    failure_action: rollback
    monitor: 45s
    max_failure_ratio: 0

  rollback_config:
    parallelism: 1
    delay: 5s
    order: start-first
```

**My understanding**:
If ANY replica fails healthcheck within 45s of starting, Swarm automatically:
1. Stops the failed update
2. Reverts to previous image version
3. Uses rollback_config settings to restore old version
4. Logs the failure for investigation

**Why important**:
Prevents bad deployments from cascading. If new code has a bug, it auto-reverts before impacting users.

**max_failure_ratio: 0** means "zero tolerance" - even 1 failure triggers rollback. For less critical services, you might use `0.3` (30% failures tolerated).

---

## 💻 Minimal Example

### Context
From Glasck - Frontend service with aggressive safety: start-first deployment, long healthcheck grace period, automatic rollback on any failure.

### Code
```yaml
# Frontend service with safe deployment strategy
front-website-dev:
  image: glasck-front-website:${FRONT_VERSION:-latest}
  networks:
    - internal-backend
    - public-facing
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3000/api/health"]
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 120s  # 2 minutes grace period for Next.js startup
  stop_grace_period: 30s  # Allow 30s for graceful shutdown
  deploy:
    replicas: 1

    # UPDATE STRATEGY
    update_config:
      parallelism: 1          # One replica at a time
      delay: 15s              # Wait 15s between updates
      failure_action: rollback # Auto-rollback on failure
      monitor: 60s            # Watch for 60s after starting
      max_failure_ratio: 0    # Zero failures tolerated
      order: start-first      # New starts before old stops

    # ROLLBACK STRATEGY
    rollback_config:
      parallelism: 1          # Rollback one at a time
      delay: 5s               # Faster rollback (5s vs 15s)
      order: start-first      # Same strategy as update

    # RESTART POLICY
    restart_policy:
      condition: on-failure   # Restart on crashes
      delay: 10s              # Wait 10s before restart
      max_attempts: 5         # Try 5 times max
      window: 60s             # Within 60s window

    # RESOURCE LIMITS
    resources:
      limits:
        cpus: '2'
        memory: 6G
      reservations:
        cpus: '1'
        memory: 2G

# API service with similar strategy
api-fastify-dev:
  image: glasck-api-fastify:${API_VERSION:-latest}
  networks:
    - internal-backend
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:3001/api/health"]
    interval: 10s
    timeout: 5s
    retries: 10
    start_period: 180s  # 3 minutes for API startup (longer than frontend)
  deploy:
    replicas: 1
    update_config:
      parallelism: 1
      delay: 10s
      failure_action: rollback
      monitor: 45s
      max_failure_ratio: 0
      order: start-first
    rollback_config:
      parallelism: 1
      delay: 5s
      order: start-first
    restart_policy:
      condition: on-failure
      delay: 10s
      max_attempts: 5
      window: 60s
```

### Line-by-Line Explanation
```
update_config:
  parallelism: 1
    → Update ONE replica at a time
    → With replicas: 1, this is entire service
    → For replicas: 3, would update in 3 batches

  delay: 15s
    → After updating 1 replica, wait 15s before next
    → Allows monitoring for issues before continuing
    → Gives time to catch errors in logs

  failure_action: rollback
    → If update fails, automatically revert to previous version
    → Alternatives: "pause" (stop deployment) or "continue" (ignore failures)

  monitor: 60s
    → After new container starts, watch it for 60s
    → If healthcheck fails during this window, triggers failure_action
    → Must be > start_period for meaningful monitoring

  max_failure_ratio: 0
    → ANY failure (0%) triggers rollback
    → Alternative: 0.3 = tolerate 30% failures
    → Strict setting for critical services

  order: start-first
    → Start new replica BEFORE stopping old
    → Temporarily have 2 replicas during update
    → Zero downtime at cost of extra resources
    → Alternative: stop-first (saves resources, brief downtime)

rollback_config:
  delay: 5s
    → Faster rollback than update (5s vs 15s)
    → Want to restore service quickly if update failed

stop_grace_period: 30s
  → Give container 30s to finish existing requests
  → Swarm sends SIGTERM, waits 30s, then SIGKILL
  → Important for Next.js to close connections gracefully
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: monitor Period < start_period

**Symptom**:
```
Service updates successfully every time
Even when new version is broken!
Healthcheck shows "starting" during entire monitor window
```

**What I did wrong**:
```yaml
# ❌ Wrong - monitor ends before healthcheck passes
healthcheck:
  start_period: 120s  # 2 minutes grace period
deploy:
  update_config:
    monitor: 30s      # Only monitor for 30s!
```

**Why wrong**: Healthcheck has 120s grace period before failures count. But monitor window is only 30s. So Swarm finishes monitoring while container is still in "starting" state, declares success, and moves on.

**Solution**:
```yaml
# ✅ Correct - monitor LONGER than start_period
healthcheck:
  start_period: 120s
deploy:
  update_config:
    monitor: 180s     # Monitor for 3 minutes (2min grace + 1min actual checks)
    # Or at minimum: monitor > start_period
```

**Time wasted**: 2h deploying broken versions
**Lesson**: `monitor` must be LONGER than `start_period` for automatic rollback to work. Otherwise, monitoring ends before healthchecks start counting failures.

---

### Pitfall 2: Insufficient Resources for start-first

**Symptom**:
```
Update hangs indefinitely
Old container running
New container stuck in "Pending" state
Error: "insufficient resources on the cluster"
```

**What I did wrong**:
```yaml
# ❌ Wrong - Not enough resources for N+1 replicas
deploy:
  replicas: 3
  update_config:
    order: start-first  # Need 4 replicas temporarily!
  resources:
    reservations:
      memory: 2G  # Each replica needs 2GB
# Cluster has 6GB total: Can run 3 replicas, but not 4!
```

**Why wrong**: `start-first` temporarily needs N+1 replicas (4 in this case), but cluster only has room for 3.

**Solution**:
```yaml
# ✅ Option 1: Use stop-first (saves resources)
deploy:
  update_config:
    order: stop-first  # Only need N replicas max

# ✅ Option 2: Lower resource reservations
deploy:
  resources:
    reservations:
      memory: 1G  # Cluster can handle 6 replicas now

# ✅ Option 3: Add more cluster capacity
# Scale cluster to handle N+1 replicas
```

**Time wasted**: 1h
**Lesson**: With `start-first`, ensure cluster has resources for N+1 replicas. Otherwise, use `stop-first`.

---

### Pitfall 3: Rollback to Broken Image

**Symptom**:
```
Deploy new version (v2) → fails healthcheck → auto-rollback
But rollback also fails!
Service stuck in update loop
```

**What I did wrong**:
```bash
# ❌ Wrong - Rollback to previous image, which was ALSO broken
# Deployment history:
# v1.0.0 → working
# v1.1.0 → broken (pushed to registry)
# v1.2.0 → also broken (current)

# Rollback goes from v1.2.0 to v1.1.0 (both broken!)
```

**Why wrong**: Swarm rolls back to "previous image", not "last working image". If you deployed multiple broken versions, rollback might still be broken.

**Solution**:
```bash
# ✅ Option 1: Manual rollback to known-good version
docker service update --image glasck-front-website:v1.0.0 glasck_front-website

# ✅ Option 2: Keep versioned backups (from Glasck Taskfile)
task backup  # Saves current working images before deployment

# ✅ Option 3: Deployment tagging strategy
# Always tag working versions as "stable"
docker tag glasck-front-website:v1.0.0 glasck-front-website:stable

# Rollback to stable
docker service update --image glasck-front-website:stable glasck_front-website
```

**Time wasted**: 45min
**Lesson**: Test thoroughly before deploying. Rollback only helps if previous version was working. Keep manual rollback commands ready.

---

### Pitfall 4: Ignoring stop_grace_period

**Symptom**:
```
During update, see errors in logs:
"ECONNRESET" - connections forcefully closed
"502 Bad Gateway" spikes during deployment
Some requests lost during rolling update
```

**What I did wrong**:
```yaml
# ❌ Wrong - No grace period configured
deploy:
  update_config:
    order: start-first
# Swarm immediately kills old container (default 10s)
# Existing HTTP requests abruptly terminated
```

**Why wrong**: When Swarm updates, it sends SIGTERM to old container and waits `stop_grace_period` (default 10s) before SIGKILL. But Next.js needs time to finish handling existing requests.

**Solution**:
```yaml
# ✅ Correct - Give adequate time for graceful shutdown
services:
  front-website:
    stop_grace_period: 30s  # 30 seconds for Next.js shutdown
    deploy:
      update_config:
        order: start-first

# In Next.js app, handle SIGTERM gracefully:
// server.js
process.on('SIGTERM', () => {
  console.log('SIGTERM received, closing server gracefully');
  server.close(() => {
    console.log('Server closed');
    process.exit(0);
  });
});
```

**Time wasted**: 1h debugging request errors
**Lesson**: Always set `stop_grace_period` to allow graceful shutdown. For web servers, 30s is reasonable. Monitor logs during updates to verify clean shutdowns.

---

## 📊 Stats

```yaml
Total time: 10h (55% assisted / 45% autonomous)
Status: ✅ Mastered
Used in: [[2025-12-glasck-swarm-deployment]]
```

---

**Validation date**: 2025-12-23
**Total time**: 11h30

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
