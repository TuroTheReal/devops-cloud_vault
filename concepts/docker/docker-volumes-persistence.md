# Docker Volumes & Data Persistence

## 📋 Metadata

```yaml
tags: [concept, docker, volumes, persistence, bind-mounts, status/learned]
created: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 1h
```

**Prerequisites**: [[docker-containers-lifecycle]], [[docker-why-containers]]
**Related to**: [[docker-compose-healthchecks]]

---

## 🎯 TL;DR (30 seconds)

> Containers are ephemeral (restart = data loss). Volumes persist data outside container. 3 types: Named volumes (Docker-managed), bind mounts (host path), tmpfs (RAM-only).
>
> **Analogy**: Container = hotel room (temporary). Volume = storage unit (permanent, survives checkout).

---

## 🤔 When to Use?

### ✅ Good for
- **Databases**: PostgreSQL, MySQL data must survive restarts
- **User uploads**: Files uploaded by users
- **Config files**: Share host config with container

### ❌ Bad for
- **Application code**: Use image layers, not volumes (immutability)
- **Temporary data**: Use tmpfs or container filesystem
- **Secrets**: Use Docker secrets, not volumes (security)

---

## 📚 ccc

### 1. Container Filesystem is Ephemeral

**My understanding**:
> **Data written to container filesystem disappears on restart - containers are immutable by design**
>
> Why containers lose data:
> - Container = read-only image layers + writable top layer
> - Restart = new writable layer, old one discarded
> - Update image = new container, old data gone
>
> Example problem:
> ```bash
> docker run postgres  # Start database
> # Write data to /var/lib/postgresql/data
> docker restart postgres  # Data LOST!
> ```

**Why important**:
> **Understanding this prevents accidental data loss - databases, uploads MUST use volumes**

---

### 2. Named Volumes (Docker-Managed)

**My understanding**:
> **Docker creates and manages volume in `/var/lib/docker/volumes/` - you reference by name, Docker handles storage location**
>
> How it works:
> - Create volume: `docker volume create mydata`
> - Mount in container: `docker run -v mydata:/data app`
> - Docker stores in: `/var/lib/docker/volumes/mydata/_data`
> - Survives container restarts, removals
> - Can be shared between containers
>
> Advantages:
> - Docker manages location (portable across hosts)
> - Backup with `docker volume` commands
> - Works on Windows/Mac (no path issues)

**Why important**:
> **This is the recommended way for production data - Docker handles complexity**

**Mental model**:
```
Container 1           Volume "mydata"           Host
┌─────────────┐       ┌──────────────┐       ┌─────────────────────────┐
│ /app/data ──┼──────→│  mydata      │←──────┤ /var/lib/docker/volumes/│
└─────────────┘       │  (persists)  │       │ mydata/_data/           │
                      └──────────────┘       └─────────────────────────┘
Container 2                ↑
┌─────────────┐            │
│ /app/data ──┼────────────┘
└─────────────┘
(Both containers share same data!)
```

---

### 3. Bind Mounts (Host Path Mapping)

**My understanding**:
> **Map host directory directly to container - you control exact path, Docker just mounts it**
>
> How it works:
> - Specify host path: `docker run -v /host/path:/container/path app`
> - Container sees host files directly
> - Changes in container = changes on host (bidirectional)
> - Path must exist on host
>
> Use cases:
> - Development: Mount source code for live reload
> - Config files: Mount `/etc/myapp/config.yml` to container
> - Logs: Write container logs to `/var/log/myapp/`

**Why important**:
> **Essential for dev workflow (code changes without rebuild) and host integration (logs, configs)**

**Difference from volumes**:
```bash
# Named volume (Docker manages location)
docker run -v mydata:/data postgres
# Data in /var/lib/docker/volumes/mydata/_data

# Bind mount (you specify location)
docker run -v /home/user/pgdata:/data postgres
# Data in /home/user/pgdata (exact path you chose)
```

---

### 4. tmpfs Mounts (RAM Storage)

**My understanding**:
> **Store data in RAM, not disk - fast but disappears on container stop (even before restart)**
>
> How it works:
> - Mount tmpfs: `docker run --tmpfs /tmp app`
> - Data stored in host RAM
> - Never written to disk (secure for sensitive data)
> - Lost on container stop (not just restart!)
>
> Use cases:
> - Temporary processing files (fast I/O)
> - Sensitive data (passwords, tokens - never touch disk)
> - Cache (speed, doesn't need persistence)

**Why important**:
> **Security + performance - sensitive temp data never hits disk (no forensic recovery)**

---

## 💻 Minimal Example

### Context
Database container with persistent data - demonstrate data survives container restart and removal.

### Code

**1. Without volume (data loss)**:
```bash
# Start PostgreSQL without volume
docker run -d --name db1 \
  -e POSTGRES_PASSWORD=secret \
  postgres

# Create table, insert data
docker exec -it db1 psql -U postgres -c "
  CREATE TABLE users (id SERIAL, name TEXT);
  INSERT INTO users (name) VALUES ('Alice');
"

# Check data exists
docker exec -it db1 psql -U postgres -c "SELECT * FROM users;"
# Output: Alice

# Restart container
docker restart db1

# Data LOST!
docker exec -it db1 psql -U postgres -c "SELECT * FROM users;"
# ERROR: relation "users" does not exist
```

**2. With named volume (data persists)**:
```bash
# Create volume
docker volume create pgdata

# Start PostgreSQL with volume
docker run -d --name db2 \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres

# Create data
docker exec -it db2 psql -U postgres -c "
  CREATE TABLE users (id SERIAL, name TEXT);
  INSERT INTO users (name) VALUES ('Alice');
"

# Remove container completely!
docker rm -f db2

# Create new container with SAME volume
docker run -d --name db3 \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres

# Data SURVIVES!
docker exec -it db3 psql -U postgres -c "SELECT * FROM users;"
# Output: Alice (data persisted across container removal!)
```

**3. Bind mount for development**:
```bash
# Mount source code for live reload
docker run -d --name app \
  -v $(pwd)/src:/app/src \
  -p 3000:3000 \
  node:18 \
  sh -c "cd /app && npm run dev"

# Edit src/index.js on host → changes immediately in container!
# No rebuild needed (dev workflow)
```

**Output**: Named volume preserves database even after container removal. Bind mount enables live code reload.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Lost Database Data on Container Update

**Symptom**: Updated PostgreSQL image version, all database data disappeared.

**What I did wrong**:
```bash
# ❌ Wrong - no volume, data in container filesystem
docker run -d --name db postgres:14
# Insert data...
docker rm -f db
docker run -d --name db postgres:15  # Upgrade
# Data LOST!
```

**Why wrong**:
> Database stored data in container filesystem (`/var/lib/postgresql/data`). Removing container = losing data.

**Solution**:
```bash
# ✅ Correct - use named volume
docker volume create pgdata
docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data \
  postgres:14

# Upgrade (data survives!)
docker rm -f db
docker run -d --name db \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15  # Same volume, data intact!
```

**Time wasted**: 2 hours (had to restore from backup)
**Lesson**: ALWAYS use volumes for databases - container filesystem is ephemeral

---

### Pitfall 2: Bind Mount Path Doesn't Exist

**Symptom**: Container starts but directory is empty, expected files missing.

**What I did wrong**:
```bash
# ❌ Wrong - typo in host path
docker run -v /hom/user/data:/data app  # Typo: /hom instead of /home
# Docker creates /hom/user/data (empty directory!)
# Container sees empty /data
```

**Why wrong**:
> Docker creates host directory if it doesn't exist (silently!). Typo = wrong path, empty mount.

**Solution**:
```bash
# ✅ Verify path exists first
ls /home/user/data  # Check path before mounting

# ✅ Use absolute paths (avoid confusion)
docker run -v /home/user/data:/data app

# ✅ Or use $(pwd) for relative
docker run -v $(pwd)/data:/data app
```

**Lesson**: Always verify host paths exist before bind mounting - Docker won't warn about typos

---

### Pitfall 3: Permission Issues with Bind Mounts

**Symptom**: Container can't write to bind-mounted directory, "Permission denied" errors.

**What I did wrong**:
> Mounted host directory owned by root, container runs as `node` user (UID 1000). User mismatch = permission denied.

**Solution**:
```bash
# ✅ Option 1: Change host directory ownership
sudo chown -R 1000:1000 /host/data  # Match container user UID
docker run -v /host/data:/data app

# ✅ Option 2: Run container as root (not recommended)
docker run --user root -v /host/data:/data app

# ✅ Option 3: Use named volume (Docker handles permissions)
docker volume create appdata
docker run -v appdata:/data app  # No permission issues!
```

**Lesson**: Bind mounts inherit host permissions - match UIDs or use named volumes for simplicity

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why does data disappear when you restart a container, and how do volumes solve this?</summary>

**Answer**: Containers are ephemeral - filesystem changes live in writable layer that's discarded on restart. Volumes store data outside container (on host or Docker-managed location), persisting across restarts and removals. Database in container = data loss. Database with volume = data survives.
</details>

<details>
<summary><strong>Q2:</strong> What's the difference between named volumes and bind mounts, and when would you use each?</summary>

**Answer**: Named volumes are Docker-managed (Docker chooses location like `/var/lib/docker/volumes/`), portable, recommended for production data. Bind mounts map specific host path (you control location), used for dev (mount source code), configs, logs. Use volumes for data, bind mounts for host integration.
</details>

<details>
<summary><strong>Q3:</strong> Why might a bind mount appear empty even though the host directory has files?</summary>

**Answer**: Typo in host path - Docker creates the directory if it doesn't exist (silently). Example: `-v /hom/user/data:/data` (typo: `/hom`) creates empty `/hom/user/data/`, container sees empty mount. Always verify host path exists before mounting.
</details>

<details>
<summary><strong>Q4:</strong> When would you use tmpfs instead of a regular volume?</summary>

**Answer**: tmpfs stores data in RAM (never disk) - use for sensitive temp data (passwords, tokens that shouldn't touch disk), fast temporary processing, or cache. Data lost on container stop (even before restart). Security + speed, but not persistent.
</details>

---

## 📊 Stats

```yaml
Total time: 1h (30% assisted / 70% autonomous)
Status: 🟡 Learning
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
