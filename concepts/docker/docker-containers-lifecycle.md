# Docker Containers - Lifecycle & Runtime

## 📋 Metadata

```yaml
tags: [concept, docker, containers, lifecycle, runtime, status/learned]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐ (2/5)
time-to-master: 3h
```

**Prerequisites**: [[docker-images-layers]]
**Related to**: None

---

## 🎯 TL;DR (30 seconds)

A Docker container is a running instance of an image - a lightweight, isolated process with its own filesystem, network, and process tree. Containers are ephemeral: writable layer is lost on removal. Understanding container lifecycle (create → start → run → stop → remove) is essential for proper resource management and debugging.

**Analogy**: Image is like a class definition, container is an instance of that class. You can create multiple containers from one image, just like multiple objects from one class.

---

## 🤔 When to Use?

### ✅ Good for
1. **Stateless applications**: Ephemeral containers, data in volumes
2. **Microservices**: Isolated processes, independent scaling
3. **Dev/Test environments**: Quick spin up/down, reproducible

### ❌ Bad for
- **Stateful apps without volumes** → Data lost on container removal
- **Long-lived mutable state in container** → Use volumes or databases
- **Kernel-level operations** → Containers share host kernel

---

## 📚 Key Concepts

### 1. Container Lifecycle States

**My understanding**:
A container goes through several states:
```
Created → Running → Paused → Stopped → Removed
   ↓         ↓        ↓         ↓
 (start)  (pause)  (stop)    (rm)
```

- **Created**: Container exists but not started
- **Running**: Process executing, consuming resources
- **Paused**: Process frozen (SIGSTOP), RAM preserved
- **Stopped**: Process exited (gracefully or killed)
- **Removed**: Container deleted, writable layer gone

**Why important**:
Stopped containers still consume disk (writable layer). Always clean up stopped containers to free space.

**Mental model**:
- Image = CD/DVD (read-only)
- Container = CD player with disc loaded (writable layer on top)
- Stop = press pause, data in RAM
- Remove = eject disc, lose any notes you wrote

---

### 2. Container Writable Layer

**My understanding**:
When you run a container, Docker creates a thin writable layer on top of read-only image layers. All file modifications (logs, temp files, app data) go here. This layer is LOST when container is removed.

**Why important**:
Explains why data disappears when container is recreated. Solution: volumes for persistent data.

**Real example**:
```bash
# Start container
docker run -d --name web nginx

# Modify file inside container
docker exec web sh -c "echo 'test' > /usr/share/nginx/html/test.txt"

# File exists
docker exec web cat /usr/share/nginx/html/test.txt
# Output: test

# Remove container
docker rm -f web

# Start new container from same image
docker run -d --name web nginx

# File GONE
docker exec web cat /usr/share/nginx/html/test.txt
# Error: No such file
```

**Solution**: Use volumes for data that must persist.

---

### 3. Container vs Process

**My understanding**:
Container is NOT a VM - it's an isolated process (or process tree) using Linux namespaces and cgroups. No separate kernel, minimal overhead.

**Namespace isolation**:
- **PID**: Container sees only its processes (PID 1 = container's main process)
- **Network**: Separate IP, ports, routing table
- **Mount**: Separate filesystem view
- **UTS**: Separate hostname
- **IPC**: Separate inter-process communication
- **User**: Can map container UID to different host UID

**Cgroups** (resource limits):
- CPU quota/shares
- Memory limits
- Disk I/O limits
- Network bandwidth (with additional tools)

**Why important**:
Understanding that containers are processes explains why they start fast (<1s), why they're lightweight, and why they share host kernel.

---

### 4. Container Exit Codes

**My understanding**:
When container stops, it has an exit code (like any process):
- **0**: Success (clean exit)
- **1**: Application error (crash, exception)
- **137**: Killed by SIGKILL (often OOM - Out Of Memory)
- **139**: Segmentation fault
- **143**: Terminated by SIGTERM (graceful shutdown)

**Why important**:
Exit code reveals WHY container stopped. Essential for debugging crashes.

**How to check**:
```bash
# View exit code
docker inspect --format='{{.State.ExitCode}}' myapp

# View reason
docker inspect --format='{{.State.Status}}' myapp

# If OOMKilled
docker inspect --format='{{.State.OOMKilled}}' myapp
```

**From experience**:
```
137 = Usually OOM (memory limit exceeded)
143 = Normal shutdown (docker stop)
0   = App exited cleanly
1   = App crashed (check logs)
```

---

### 5. Container Restart Policies

**My understanding**:
Restart policies control what happens when container exits:
- **no**: Never restart (default)
- **on-failure[:max-retries]**: Restart only if exit code != 0
- **always**: Always restart, even after reboot
- **unless-stopped**: Always restart, except if manually stopped

**Why important**:
Production services need `unless-stopped` or `always`. Development might use `on-failure` or `no`.

**Real example**:
```yaml
services:
  # Production API - always running
  api:
    image: myapi:latest
    restart: unless-stopped  # Restart after reboot, unless I stopped it

  # Database - critical
  db:
    image: postgres:15
    restart: always  # Always restart, even if I stopped it

  # Development worker - retry on crash
  worker:
    image: myworker:dev
    restart: on-failure:3  # Retry up to 3 times if crash

  # One-time migration - never restart
  migration:
    image: myapp:latest
    command: npm run migrate
    restart: "no"  # Run once, don't restart
```

---

## 💻 Minimal Example

### Context
Managing a web application container through its full lifecycle, demonstrating ephemeral nature and proper volume usage.

### Code

**Without Volume (data lost)**:
```bash
# Start container
docker run -d --name webapp \
  -p 8080:80 \
  nginx:alpine

# Create data inside container
docker exec webapp sh -c "echo 'Important Data' > /usr/share/nginx/html/data.txt"

# Verify data exists
docker exec webapp cat /usr/share/nginx/html/data.txt
# Output: Important Data

# Stop and remove container
docker rm -f webapp

# Start new container
docker run -d --name webapp \
  -p 8080:80 \
  nginx:alpine

# Data is GONE!
docker exec webapp cat /usr/share/nginx/html/data.txt
# Error: No such file or directory
```

**With Volume (data persists)**:
```bash
# Create named volume
docker volume create webapp-data

# Start container with volume
docker run -d --name webapp \
  -p 8080:80 \
  -v webapp-data:/usr/share/nginx/html \
  nginx:alpine

# Create data in volume
docker exec webapp sh -c "echo 'Important Data' > /usr/share/nginx/html/data.txt"

# Remove container
docker rm -f webapp

# Start NEW container with SAME volume
docker run -d --name webapp-new \
  -p 8080:80 \
  -v webapp-data:/usr/share/nginx/html \
  nginx:alpine

# Data PERSISTS!
docker exec webapp-new cat /usr/share/nginx/html/data.txt
# Output: Important Data
```

**Lifecycle Management**:
```bash
# Create container (doesn't start it)
docker create --name myapp \
  -p 3000:3000 \
  -e NODE_ENV=production \
  myapp:latest

# Start created container
docker start myapp

# Pause container (freeze process, keep in RAM)
docker pause myapp

# Unpause
docker unpause myapp

# Stop gracefully (SIGTERM, then SIGKILL after 10s)
docker stop -t 10 myapp

# Start again
docker start myapp

# Kill immediately (SIGKILL)
docker kill myapp

# Remove container
docker rm myapp

# One command: run, stop, remove
docker run --rm --name temp-app myapp:latest
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Data Loss on Container Recreate

**Symptom**:
```
Uploaded files disappear after container restart
User sessions lost
Application state reset
```

**What I did wrong**:
```bash
# ❌ Wrong - No volume for data
docker run -d --name app \
  -p 3000:3000 \
  myapp:latest

# Users upload files to /app/uploads inside container
# Deploy new version:
docker rm -f app
docker run -d --name app -p 3000:3000 myapp:v2
# All uploads GONE!
```

**Why wrong**: Container writable layer is ephemeral. Removed with container.

**Solution**:
```bash
# ✅ Correct - Volume for persistent data
docker volume create app-uploads

docker run -d --name app \
  -p 3000:3000 \
  -v app-uploads:/app/uploads \
  myapp:latest

# Now uploads survive container recreation
```

**Time wasted**: 2h debugging + data recovery attempt
**Lesson**: Always use volumes for data that must persist. Containers are ephemeral.

---

### Pitfall 2: Container Exit Code 137 (OOM)

**Symptom**:
```
Container randomly exits
Exit code: 137
Logs show no errors before crash
```

**What I did wrong**:
```bash
# ❌ Wrong - No memory limit, or limit too low
docker run -d --name api \
  --memory="256m" \  # Too low for this app!
  myapi:latest

# App uses 300MB → OOM Killer → Exit 137
```

**Why wrong**: Linux OOM (Out Of Memory) Killer terminates process when memory limit exceeded. Exit code 137 = SIGKILL = OOM.

**Solution**:
```bash
# ✅ Option 1: Increase memory limit
docker run -d --name api \
  --memory="1g" \
  --memory-swap="1.5g" \
  myapi:latest

# ✅ Option 2: Check memory usage first
docker stats api

# ✅ Option 3: Investigate memory leak
docker exec api node --trace-gc app.js

# Check if OOM killed
docker inspect --format='{{.State.OOMKilled}}' api
# true = memory limit was issue
```

**Time wasted**: 3h debugging
**Lesson**: Exit code 137 usually means OOM. Check memory limits and app memory usage.

---

### Pitfall 3: Stopped Containers Consuming Disk

**Symptom**:
```
Disk space filling up
docker system df shows large "Containers" usage
No running containers using space
```

**What I did wrong**:
```bash
# ❌ Wrong - Containers accumulate over time
docker run --name test1 myapp
docker run --name test2 myapp
# ...hundreds of stopped containers

docker ps -a | wc -l
# 237 stopped containers!

docker system df
# Containers: 15GB
```

**Why wrong**: Stopped containers retain their writable layer (logs, temp files). Disk space not freed until removed.

**Solution**:
```bash
# ✅ Option 1: Use --rm flag for temporary containers
docker run --rm --name temp myapp

# ✅ Option 2: Regular cleanup
docker container prune -f

# ✅ Option 3: Automatic cleanup script
# Cron: 0 2 * * * docker container prune -f

# ✅ Option 4: Remove old containers
docker ps -a --filter "status=exited" --filter "created<24h" -q | xargs docker rm
```

**Time wasted**: 1h cleanup
**Lesson**: Stopped containers consume disk. Use `--rm` for temp containers, regular pruning for long-running systems.

---

### Pitfall 4: Container Doesn't Stop (Ignoring SIGTERM)

**Symptom**:
```
docker stop myapp takes 10+ seconds
Container killed forcefully (SIGKILL)
Ungraceful shutdown (connections dropped)
```

**What I did wrong**:
```dockerfile
# ❌ Wrong - Shell form of CMD
FROM node:18
CMD npm start

# PID 1 = /bin/sh -c "npm start"
# SIGTERM sent to sh, not node
# sh doesn't forward signal → timeout → SIGKILL
```

**Why wrong**: Shell form wraps command in `/bin/sh -c`. SIGTERM goes to shell (PID 1), not application. Shell doesn't forward signals.

**Solution**:
```dockerfile
# ✅ Option 1: Exec form
FROM node:18
CMD ["node", "server.js"]
# PID 1 = node process directly
# SIGTERM goes to node → graceful shutdown

# ✅ Option 2: Handle SIGTERM in app
// server.js
process.on('SIGTERM', () => {
  console.log('SIGTERM received, closing server...');
  server.close(() => {
    console.log('Server closed gracefully');
    process.exit(0);
  });
});

# ✅ Option 3: Use tini (init system)
FROM node:18
RUN apk add --no-cache tini
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["node", "server.js"]
```

**Time wasted**: 2h debugging slow shutdowns
**Lesson**: Use exec form for CMD/ENTRYPOINT. PID 1 must handle SIGTERM. Consider tini for complex cases.

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why do containers lose data when removed, and what's the solution?</summary>

**Answer**: Container writable layer is ephemeral - all file modifications (logs, uploads, app data) exist only in this layer and are lost when container is removed. This is by design for stateless applications. Solution: use volumes for data that must persist across container recreations.
</details>

<details>
<summary><strong>Q2:</strong> What does exit code 137 mean, and how do you diagnose it?</summary>

**Answer**: Exit code 137 = SIGKILL, usually from OOM (Out Of Memory) Killer when container exceeds memory limit. Diagnose with `docker inspect --format='{{.State.OOMKilled}}' container` (returns true if OOM). Fix by increasing memory limit, checking for memory leaks, or investigating actual memory usage with `docker stats`.
</details>

<details>
<summary><strong>Q3:</strong> Why do stopped containers still consume disk space, and what should you do?</summary>

**Answer**: Stopped containers retain their writable layer (logs, temp files, changes) on disk until explicitly removed. They accumulate over time. Solutions: use `--rm` flag for temporary containers, run `docker container prune -f` regularly, or set up automated cleanup cron jobs.
</details>

<details>
<summary><strong>Q4:</strong> Why might `docker stop` take 10+ seconds and kill containers forcefully?</summary>

**Answer**: Shell form CMD (`CMD npm start`) wraps command in `/bin/sh -c`, making shell PID 1. SIGTERM goes to shell which doesn't forward to app, causing timeout then SIGKILL. Fix with exec form (`CMD ["node", "server.js"]`) so app is PID 1 and receives signals directly, or handle SIGTERM in app code.
</details>

---

## 📊 Stats

```yaml
Total time: 7h (40% assisted / 60% autonomous)
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
