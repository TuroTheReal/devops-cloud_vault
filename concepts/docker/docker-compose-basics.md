# Docker Compose Basics

## 📋 Metadata

```yaml
tags: [concept, docker, docker-compose, multi-container, orchestration, status/learned]
created: 2025-12-26
difficulty: ⭐⭐ (2/5)
time-to-master: 1h30
```

**Prerequisites**: [[docker-containers-lifecycle]], [[docker-volumes-persistence]]
**Related to**: [[docker-compose-healthchecks]], [[docker-network-isolation]]

---

## 🎯 TL;DR (30 seconds)

> Docker Compose defines multi-container apps in YAML file. One command starts all services (db, web, redis) with networks, volumes. Replaces long `docker run` commands with declarative config.
>
> **Analogy**: docker run = cooking one dish. docker-compose = full meal recipe (appetizer, main, dessert - all coordinated).

---

## 🤔 When to Use?

### ✅ Good for
- **Multi-container apps**: Web app + database + cache (all connected)
- **Development environments**: Reproducible local stack
- **Integration testing**: Spin up test env with all dependencies

### ❌ Bad for
- **Production orchestration**: Use Docker Swarm or Kubernetes (scaling, HA)
- **Single container**: Just use `docker run` (simpler)
- **Complex networking**: Compose networking is basic (advanced needs Swarm)

---

## 📚 Key Concepts

### 1. docker-compose.yml - Declarative Configuration

**My understanding**:
> **YAML file describes entire stack - services, networks, volumes. Compose reads it and creates everything**
>
> Structure:
> ```yaml
> version: '3.8'  # Compose file version
> services:       # Container definitions
>   web:          # Service name
>     image: nginx
>     ports:
>       - "80:80"
> volumes:        # Persistent data
>   db-data:
> networks:       # Custom networks
>   app-net:
> ```
>
> Advantages:
> - Replace 10 docker run commands with 1 file
> - Version controlled (Git tracks changes)
> - Self-documenting (team sees full stack)

**Why important**:
> **Infrastructure as code - reproducible environments, no manual steps**

---

### 2. Services (Container Definitions)

**My understanding**:
> **Service = containerized component of your app (web, db, cache). Compose starts them together, handles dependencies**
>
> Service definition:
> ```yaml
> services:
>   web:
>     image: nginx:alpine         # Which image
>     ports:
>       - "80:80"                  # Port mapping
>     volumes:
>       - ./html:/usr/share/nginx/html  # Bind mount
>     environment:
>       - NGINX_HOST=example.com   # Environment variables
>     depends_on:
>       - db                       # Start db before web
>
>   db:
>     image: postgres:14
>     volumes:
>       - pgdata:/var/lib/postgresql/data
>     environment:
>       POSTGRES_PASSWORD: secret
> ```

**Why important**:
> **Defines how containers work together - dependencies, networking, config**

**Mental model**:
```
docker-compose.yml
    ↓
Compose reads file
    ↓
Creates:
  - web container (depends on db)
  - db container (starts first)
  - Shared network (app_default)
  - Volumes (pgdata)
    ↓
Services communicate via service names!
(web can reach db at hostname "db")
```

---

### 3. Networks (Service Communication)

**My understanding**:
> **Compose creates default network, all services join it. Services reach each other by service name (DNS)**
>
> How it works:
> - Compose creates network: `myapp_default`
> - All services join automatically
> - Service name = hostname (`db` → `postgresql://db:5432`)
> - Isolated from other Compose apps
>
> Example:
> ```yaml
> services:
>   web:
>     image: myapp
>     environment:
>       - DATABASE_URL=postgresql://db:5432/mydb
>       # ↑ "db" resolves to db service container!
>
>   db:
>     image: postgres:14
> ```

**Why important**:
> **Zero network configuration - services auto-discover each other**

---

### 4. depends_on (Startup Order)

**My understanding**:
> **Controls startup order - Compose starts dependencies first, but doesn't wait for "ready"**
>
> How it works:
> ```yaml
> services:
>   web:
>     image: myapp
>     depends_on:
>       - db       # Start db before web
>       - redis    # Start redis before web
>
>   db:
>     image: postgres:14
>
>   redis:
>     image: redis:7
> ```
>
> **Important**: `depends_on` only controls START order, not READY state!
> - Compose starts db container
> - Immediately starts web (doesn't wait for PostgreSQL to accept connections)
> - Web may fail if db not ready yet

**Why important**:
> **Prevents "connection refused" errors - but needs healthchecks for full readiness** (see [[docker-compose-healthchecks]])

**Startup sequence**:
```
1. docker-compose up
     ↓
2. Start db, redis (no dependencies)
     ↓
3. Wait for db, redis containers to START (not be ready!)
     ↓
4. Start web (depends_on satisfied)
     ↓
5. All containers running (but db might not accept connections yet!)
```

---

### 5. Volumes in Compose

**My understanding**:
> **Define volumes at top-level, reference in services - Compose creates named volumes automatically**
>
> Two types:
> ```yaml
> services:
>   db:
>     volumes:
>       # Named volume (Compose creates it)
>       - db-data:/var/lib/postgresql/data
>       # Bind mount (maps host path)
>       - ./init.sql:/docker-entrypoint-initdb.d/init.sql
>
> volumes:
>   db-data:  # Declare named volume here
> ```

**Why important**:
> **Data persists across `docker-compose down` (unless you use `-v` flag)**

---

## 💻 Minimal Example

### Context
Full-stack app: React frontend, Node.js API, PostgreSQL database. Demonstrate service communication and volumes.

### Code

**Project structure**:
```
myapp/
├── docker-compose.yml
├── frontend/
│   └── Dockerfile
├── backend/
│   └── Dockerfile
└── init.sql
```

**docker-compose.yml**:
```yaml
version: '3.8'

services:
  # PostgreSQL database
  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"  # Expose for debugging (optional)

  # Node.js API backend
  backend:
    build: ./backend
    environment:
      # ↓ Service name "db" resolves to db container!
      DATABASE_URL: postgresql://user:secret@db:5432/myapp
      PORT: 3000
    ports:
      - "3000:3000"
    depends_on:
      - db  # Start db first
    volumes:
      - ./backend:/app  # Bind mount for dev (live reload)
      - /app/node_modules  # Prevent overwriting node_modules

  # React frontend
  frontend:
    build: ./frontend
    environment:
      # ↓ Service name "backend" resolves to backend container!
      REACT_APP_API_URL: http://backend:3000
    ports:
      - "8080:80"
    depends_on:
      - backend

volumes:
  db-data:  # Named volume for database persistence

# Default network created automatically:
# myapp_default (all services join it)
```

**Usage**:
```bash
# Start entire stack
docker-compose up -d

# Check all services running
docker-compose ps
# Output:
#   myapp_db_1        running  5432/tcp
#   myapp_backend_1   running  3000/tcp
#   myapp_frontend_1  running  8080/tcp

# View logs (all services combined)
docker-compose logs -f

# View specific service logs
docker-compose logs backend

# Stop everything (volumes persist!)
docker-compose down

# Stop and DELETE volumes (data loss!)
docker-compose down -v

# Rebuild after code changes
docker-compose up -d --build
```

**Service communication example**:
```javascript
// backend/app.js
const express = require('express');
const { Pool } = require('pg');

// Connect to "db" service (hostname = service name!)
const pool = new Pool({
  connectionString: process.env.DATABASE_URL
  // postgresql://user:secret@db:5432/myapp
  //                          ↑ "db" resolves to db container IP
});

app.get('/api/users', async (req, res) => {
  const result = await pool.query('SELECT * FROM users');
  res.json(result.rows);
});
```

**Output**: Frontend (port 8080) → Backend (port 3000) → Database (port 5432), all communicating by service name.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: depends_on Doesn't Wait for Service Ready

**Symptom**: Backend starts before database is ready, "connection refused" error.

**What I did wrong**:
```yaml
# ❌ Wrong - assumes db is READY when started
services:
  backend:
    depends_on:
      - db  # Only waits for container START, not PostgreSQL READY
```

**Why wrong**:
> `depends_on` controls start order, not readiness. PostgreSQL container starts, but takes 5s to accept connections. Backend tries to connect immediately, fails.

**Solution**:
```yaml
# ✅ Option 1: Add healthcheck + depends_on condition
services:
  db:
    image: postgres:14
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "user"]
      interval: 5s
      timeout: 3s
      retries: 5

  backend:
    depends_on:
      db:
        condition: service_healthy  # Wait until healthcheck passes!
```

**Lesson**: Use healthchecks with `depends_on` conditions for true readiness (see [[docker-compose-healthchecks]])

---

### Pitfall 2: Hardcoded Hostnames Instead of Service Names

**Symptom**: Backend can't connect to database, "host not found" error.

**What I did wrong**:
```javascript
// ❌ Wrong - hardcoded localhost (wrong in container!)
const pool = new Pool({
  host: 'localhost',  // localhost = backend container itself, NOT db!
  port: 5432
});
```

**Why wrong**:
> `localhost` in container = container itself, not host machine or other containers. Services are separate containers with separate IPs.

**Solution**:
```javascript
// ✅ Correct - use service name (Compose DNS resolves it)
const pool = new Pool({
  host: 'db',  // "db" = service name from docker-compose.yml
  port: 5432
});
```

**Lesson**: In Compose, service name = hostname. Never use `localhost` for inter-service communication.

---

### Pitfall 3: Lost Data After docker-compose down

**Symptom**: Ran `docker-compose down`, all database data disappeared.

**What I did wrong**:
```bash
# ❌ Wrong - down -v deletes volumes!
docker-compose down -v
# Removes containers + networks + VOLUMES (data loss!)
```

**Why wrong**:
> `-v` flag tells Compose to remove volumes. Database data stored in volume = data deleted.

**Solution**:
```bash
# ✅ Correct - down WITHOUT -v (volumes persist)
docker-compose down
# Removes containers + networks, KEEPS volumes

# Only use -v when you WANT to delete data (clean slate)
docker-compose down -v  # Dev: reset database to clean state
```

**Lesson**: `docker-compose down` is safe (preserves data). `docker-compose down -v` deletes volumes (data loss)!

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> How do services in docker-compose.yml communicate with each other, and what hostname do they use?</summary>

**Answer**: Compose creates a default network that all services join. Services communicate using service name as hostname - example: `backend` service connects to `db` service at hostname "db". Compose DNS resolves "db" to db container's IP. Never use localhost for inter-service communication.
</details>

<details>
<summary><strong>Q2:</strong> What's the difference between depends_on and healthcheck for startup ordering?</summary>

**Answer**: `depends_on` only controls container START order (db starts before backend), but doesn't wait for service READY. Backend might try connecting before PostgreSQL accepts connections. Healthcheck + `condition: service_healthy` makes Compose wait until service is actually ready to receive traffic.
</details>

<details>
<summary><strong>Q3:</strong> What happens to volumes when you run docker-compose down vs docker-compose down -v?</summary>

**Answer**: `docker-compose down` removes containers and networks but PRESERVES volumes (data persists). `docker-compose down -v` removes volumes too (data loss!). Use `-v` only when you want to delete data (clean slate for dev), never in production.
</details>

<details>
<summary><strong>Q4:</strong> Why can't you use localhost to connect from one service to another in Compose?</summary>

**Answer**: Each container has its own network namespace - `localhost` in a container refers to that container itself, not the host machine or other containers. Services are isolated. Use service names (DNS) to communicate: backend connects to db at hostname "db", not "localhost".
</details>

---

## 📊 Stats

```yaml
Total time: 1h30 (35% assisted / 65% autonomous)
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
