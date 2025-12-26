# Why Containers? - Docker

## 📋 Metadata

```yaml
tags: [concept, docker, containers, virtualization, status/learned]
created: 2025-12-26
difficulty: ⭐ (1/5)
time-to-master: 30min
```

**Prerequisites**: None (foundational)
**Related to**: [[docker-containers-lifecycle]], [[docker-images-layers]]

---

## 🎯 TL;DR (30 seconds)

> Containers package application + dependencies in isolated environment. Lighter than VMs (no full OS), consistent across environments (dev = prod), faster to start.
>
> **Analogy**: VM = entire house (walls, plumbing, electricity). Container = apartment in building (shares infrastructure, isolated space)

---

## 🤔 When to Use?

### ✅ Good for
- **Microservices**: Each service in isolated container
- **Dev/Prod parity**: "Works on my machine" → works everywhere
- **CI/CD pipelines**: Fast, reproducible builds

### ❌ Bad for
- **GUI applications**: Containers designed for server apps
- **Kernel-level access**: Need full OS control → VM better
- **Windows apps on Linux**: Containers share host kernel

---

## 📚 Key Concepts

### 1. VMs vs Containers

**My understanding**:
> **VMs virtualize hardware, containers virtualize OS - containers share kernel, VMs don't**
>
> **Virtual Machine**:
> - Full OS per app (Windows, Linux)
> - Hypervisor (VMware, VirtualBox)
> - Heavy (GB of RAM, minutes to boot)
> - Strong isolation (separate kernel)
>
> **Container**:
> - Share host OS kernel
> - Container runtime (Docker, containerd)
> - Lightweight (MB of RAM, seconds to boot)
> - Process-level isolation (namespaces, cgroups)

**Why important**:
> **Understanding this explains why containers are faster but less isolated than VMs**

**Mental model**:
```
VM Architecture:
┌─────────────────────────────────────┐
│  App A  │  App B  │  App C          │
│  Bins   │  Bins   │  Bins           │
│  Libs   │  Libs   │  Libs           │
├─────────┼─────────┼─────────────────┤
│ Guest OS│ Guest OS│ Guest OS        │
├─────────┴─────────┴─────────────────┤
│        Hypervisor (ESXi, KVM)       │
├─────────────────────────────────────┤
│          Host OS (Linux)            │
└─────────────────────────────────────┘

Container Architecture:
┌─────────────────────────────────────┐
│  App A  │  App B  │  App C          │
│  Bins   │  Bins   │  Bins           │
│  Libs   │  Libs   │  Libs           │
├─────────┴─────────┴─────────────────┤
│    Docker Engine (containerd)       │
├─────────────────────────────────────┤
│          Host OS (Linux)            │
└─────────────────────────────────────┘
```

---

### 2. The "Works on My Machine" Problem

**My understanding**:
> **Before containers: Dev has Python 3.8, prod has 3.7 → app breaks. Container includes exact Python version, same everywhere**
>
> Traditional deployment problems:
> - Different OS versions (Ubuntu 20.04 vs 22.04)
> - Missing dependencies (forgot to install libssl)
> - Config drift (dev uses SQLite, prod uses PostgreSQL)
>
> Container solution:
> - Package app + exact dependencies
> - Same image runs on laptop, CI, production
> - Reproducible builds (today = 1 year later)

**Why important**:
> **This is THE killer feature - eliminates environment inconsistencies**

---

### 3. Resource Efficiency

**My understanding**:
> **Run 100 containers on server that handles 10 VMs - containers share kernel, no OS duplication**
>
> Why containers are lighter:
> - No guest OS overhead (1 kernel for all containers)
> - Shared base layers (10 containers using `ubuntu:22.04` = stored once)
> - Fast startup (no OS boot, just process start)
> - Lower memory (app memory only, no OS memory per instance)
>
> Real-world impact:
> - VM: 2GB RAM minimum (1GB for OS, 1GB for app)
> - Container: 50MB RAM (just app, share host kernel)

**Why important**:
> **Allows dense server utilization - more apps per host = lower infrastructure cost**

---

### 4. Portability

**My understanding**:
> **Build once, run anywhere - laptop, AWS, Azure, on-premise - same container image**
>
> How portability works:
> - Container image = standardized package format
> - Docker Registry = hub for sharing images (Docker Hub, private registry)
> - Any Docker-compatible runtime can run image (Docker, Podman, Kubernetes)
> - No vendor lock-in (move between cloud providers easily)

**Why important**:
> **Deploy to any cloud without rewriting app - huge for multi-cloud strategies**

---

## 💻 Minimal Example

### Context
Demonstrate same container running on different machines - illustrates portability and consistency.

### Code

**1. Create simple app** (`app.py`):
```python
# Simple Python web app
import sys
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        message = f"Python {sys.version} running in container!"
        self.wfile.write(message.encode())

HTTPServer(('', 8000), Handler).serve_forever()
```

**2. Create Dockerfile**:
```dockerfile
# Specify exact Python version (consistency)
FROM python:3.11-slim

# Copy app
COPY app.py /app/app.py

# Run app
CMD ["python", "/app/app.py"]
```

**3. Build and run**:
```bash
# Build image (packages app + Python 3.11)
docker build -t myapp:1.0 .

# Run on laptop
docker run -p 8000:8000 myapp:1.0
# Visit http://localhost:8000 → "Python 3.11 running in container!"

# Push to registry
docker push myregistry/myapp:1.0

# Pull and run on production server (identical environment!)
docker pull myregistry/myapp:1.0
docker run -p 8000:8000 myapp:1.0
# Same Python 3.11, same app, guaranteed!
```

**Output**: App runs identically on dev laptop and production server - same Python version, same dependencies.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Assuming Containers = VMs

**Symptom**: Container exits immediately after start, expecting it to "keep running" like a VM.

**What I did wrong**:
```bash
# ❌ Wrong - expecting container to stay alive like VM
docker run ubuntu:22.04
# Container starts, nothing to run, exits immediately
```

**Why wrong**:
> Containers run single process - when process ends, container stops. VMs run init system (systemd), stay alive. Container is NOT a mini-VM.

**Solution**:
```bash
# ✅ Correct - container runs process, stays alive while process runs
docker run -it ubuntu:22.04 bash
# Bash process runs → container stays alive
# Exit bash → container stops

# ✅ For services - process must run in foreground
docker run -d nginx
# nginx runs in foreground → container stays alive
```

**Time wasted**: 20 minutes
**Lesson**: Containers run processes, not operating systems - if process exits, container exits

---

### Pitfall 2: Storing Data in Container

**Symptom**: Wrote data to container filesystem, container restarts, data gone.

**What I did wrong**:
> Stored database files inside container, didn't use volumes. Container is ephemeral - data lost on restart.

**Solution**:
```bash
# ✅ Use volumes for persistent data
docker run -v mydata:/var/lib/postgresql/data postgres
# Data persists across container restarts
```

**Lesson**: Containers are immutable - use volumes for data that must survive restarts (see [[docker-volumes-persistence]])

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> What's the fundamental difference between VMs and containers?</summary>

**Answer**: VMs virtualize hardware (each VM has full OS + kernel), containers virtualize OS (share host kernel, isolate processes). This makes containers lighter (MB vs GB), faster to start (seconds vs minutes), but less isolated (share kernel with host).
</details>

<details>
<summary><strong>Q2:</strong> Why do containers solve the "works on my machine" problem?</summary>

**Answer**: Container image packages app + exact dependencies (specific Python version, libraries, OS packages). Same image runs identically on dev laptop, CI, and production - no environment drift. Dev using Python 3.8, prod using 3.7 → both use same containerized Python version.
</details>

<details>
<summary><strong>Q3:</strong> Why does a container exit immediately after starting, and how do you keep it running?</summary>

**Answer**: Containers run a single process - when that process exits, container stops. Unlike VMs (run init system, stay alive), containers are process-focused. To keep running: provide long-running process (web server, bash interactive session). If main process exits, container exits.
</details>

---

## 📊 Stats

```yaml
Total time: 30min (20% assisted / 80% autonomous)
Status: 🟡 Learning
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
