# Docker Swarm Basics

## 📋 Metadata

```yaml
tags: [concept, docker, swarm, orchestration, clustering, status/learning]
created: 2025-12-26
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 2h
```

**Prerequisites**: [[docker-compose-basics]], [[docker-containers-lifecycle]]
**Related to**: [[docker-swarm-overlay-networks]], [[docker-swarm-deployment-strategies]]

---

## 🎯 TL;DR (30 seconds)

> Docker Swarm clusters multiple servers into one orchestrated system. Deploy services (not containers), Swarm distributes replicas across nodes, handles failures automatically. Compose for dev, Swarm for production.
>
> **Analogy**: Compose = one chef cooking. Swarm = restaurant kitchen (multiple chefs, manager assigns dishes, replaces sick chefs).

---

## 🤔 When to Use?

### ✅ Good for
- **Production deployments**: HA, auto-recovery, rolling updates
- **Scaling**: Run 10 replicas across 5 servers
- **Load balancing**: Traffic distributed across replicas automatically

### ❌ Bad for
- **Development**: Compose is simpler (single machine)
- **Complex orchestration**: Kubernetes more powerful (but complex)
- **Small apps**: Single container on one server = overkill

---

## 📚 Key Concepts

### 1. Swarm Mode - Cluster of Docker Hosts

**My understanding**:
> **Turn multiple Docker hosts into cluster - 1 manager, N workers. Deploy once, Swarm handles distribution across nodes**
>
> Key components:
> - **Manager node**: Orchestrates cluster (receives commands, assigns tasks)
> - **Worker nodes**: Run containers (execute tasks from manager)
> - **Swarm**: Cluster of all nodes working together
> - **Raft consensus**: Managers stay in sync (3-5 managers for HA)
>
> Single node vs Swarm:
> ```
> Single Docker Host:
> docker run nginx  → runs on THIS machine
>
> Swarm Cluster (3 nodes):
> docker service create nginx --replicas 3
> → Manager distributes 1 replica to each node
> → Automatic load balancing
> → If node fails, replicas rescheduled
> ```

**Why important**:
> **Transforms Docker from single-host to multi-host orchestration - foundation for HA**

**Mental model**:
```
Swarm Cluster
┌─────────────────────────────────────────────┐
│ Manager Node (leader)                       │
│  - Receives service definitions             │
│  - Schedules tasks to workers               │
│  - Monitors node health                     │
│  - Maintains desired state                  │
└─────────────────────────────────────────────┘
        │                 │
        ↓                 ↓
┌──────────────┐   ┌──────────────┐
│ Worker Node 1│   │ Worker Node 2│
│ - Runs tasks │   │ - Runs tasks │
│ - Reports    │   │ - Reports    │
│   status     │   │   status     │
└──────────────┘   └──────────────┘
```

---

### 2. Services (Not Containers!)

**My understanding**:
> **In Swarm, you deploy services (declarative), not containers (imperative). Service = desired state, Swarm ensures it**
>
> Service vs Container:
> - **Container** (Docker): `docker run nginx` → 1 container on this host
> - **Service** (Swarm): `docker service create --replicas 3 nginx` → 3 replicas across cluster
>
> Service defines:
> - Image to run (`nginx:alpine`)
> - Number of replicas (3, 10, 100)
> - Resource constraints (1 CPU, 512MB RAM)
> - Update strategy (rolling, parallel)
> - Networks, volumes, secrets
>
> Swarm ensures:
> - Desired replicas running (if 1 crashes, start new one)
> - Load balancing (traffic to healthy replicas)
> - Placement (spread across nodes for HA)

**Why important**:
> **Shift from "start container" to "declare desired state" - Swarm handles rest**

---

### 3. Nodes - Managers and Workers

**My understanding**:
> **Manager = control plane (orchestrates). Worker = data plane (runs containers). Manager can also run containers**
>
> Manager node:
> - Maintains cluster state (Raft consensus)
> - Schedules services to workers
> - Handles API requests (`docker service create`)
> - High availability: 3-5 managers (quorum)
>
> Worker node:
> - Executes tasks assigned by manager
> - Reports status back to manager
> - No orchestration logic (just runs containers)
>
> Promotion/demotion:
> ```bash
> docker node promote worker1   # Worker → Manager
> docker node demote manager2   # Manager → Worker
> ```

**Why important**:
> **Separates control (managers) from execution (workers) - scale independently**

**HA configuration**:
```
1 Manager:  No HA (manager fails = cluster dead)
3 Managers: Tolerates 1 failure (quorum = 2)
5 Managers: Tolerates 2 failures (quorum = 3)
7+ Managers: Diminishing returns (Raft overhead)

Recommendation: 3 managers for small prod, 5 for large
```

---

### 4. Stacks - Multi-Service Deployments

**My understanding**:
> **Stack = docker-compose.yml for Swarm. Deploy entire app (web + db + cache) with one command**
>
> How it works:
> - Write `docker-compose.yml` (same syntax as Compose)
> - Add Swarm-specific options (`deploy`, `replicas`)
> - Deploy: `docker stack deploy -c docker-compose.yml myapp`
> - Swarm creates all services in stack
>
> Example:
> ```yaml
> version: '3.8'
> services:
>   web:
>     image: nginx
>     deploy:
>       replicas: 3           # Swarm-specific!
>       update_config:        # Swarm-specific!
>         parallelism: 1
>         delay: 10s
>     networks:
>       - webnet
>
>   db:
>     image: postgres
>     deploy:
>       replicas: 1
>       placement:
>         constraints:
>           - node.role == manager  # Only run on managers
> ```

**Why important**:
> **Reuse Compose knowledge for production Swarm deployments - smooth dev→prod transition**

---

### 5. Desired State Reconciliation

**My understanding**:
> **You tell Swarm "I want 5 replicas". Swarm ensures 5 replicas always running, no matter what**
>
> How it works:
> - You declare: `--replicas 5`
> - Swarm creates 5 tasks across nodes
> - Node fails → 2 replicas lost
> - Swarm detects (health checks)
> - Swarm reschedules 2 new replicas on healthy nodes
> - Back to 5 replicas (desired state restored)
>
> This is declarative:
> - Imperative: "Start container on node2" (manual)
> - Declarative: "Ensure 5 replicas running" (automated)

**Why important**:
> **Self-healing system - you declare intent, Swarm handles failures automatically**

---

## 💻 Minimal Example

### Context
3-node Swarm cluster (1 manager, 2 workers). Deploy nginx service with 6 replicas, demonstrate auto-recovery.

### Code

**1. Initialize Swarm (on manager node)**:
```bash
# On manager node (e.g., server1)
docker swarm init --advertise-addr 192.168.1.10
# Output: Token to join as worker

# Swarm initialized: current node (abc123) is now a manager.
# To add a worker to this swarm, run:
#   docker swarm join --token SWMTKN-1-xyz... 192.168.1.10:2377
```

**2. Join workers**:
```bash
# On worker1 (server2)
docker swarm join --token SWMTKN-1-xyz... 192.168.1.10:2377

# On worker2 (server3)
docker swarm join --token SWMTKN-1-xyz... 192.168.1.10:2377
```

**3. Verify cluster**:
```bash
# On manager
docker node ls
# ID        HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
# abc123 *  server1   Ready   Active        Leader
# def456    server2   Ready   Active
# ghi789    server3   Ready   Active
```

**4. Deploy service**:
```bash
# Create nginx service with 6 replicas
docker service create \
  --name web \
  --replicas 6 \
  --publish 80:80 \
  nginx:alpine

# Check distribution
docker service ps web
# ID      NAME   IMAGE          NODE     DESIRED STATE  CURRENT STATE
# abc1    web.1  nginx:alpine   server1  Running        Running
# abc2    web.2  nginx:alpine   server2  Running        Running
# abc3    web.3  nginx:alpine   server3  Running        Running
# abc4    web.4  nginx:alpine   server1  Running        Running
# abc5    web.5  nginx:alpine   server2  Running        Running
# abc6    web.6  nginx:alpine   server3  Running        Running
# → 2 replicas per node (evenly distributed!)
```

**5. Test auto-recovery**:
```bash
# Simulate node failure - shut down worker1 (server2)
# On server2:
sudo systemctl stop docker

# On manager - check service status
docker service ps web
# ID      NAME       NODE     DESIRED STATE  CURRENT STATE
# abc1    web.1      server1  Running        Running
# abc2    web.2      server2  Running        Running 5 minutes ago
# NEW1    web.2      server3  Running        Running 10 seconds ago  ← NEW!
# abc3    web.3      server3  Running        Running
# abc4    web.4      server1  Running        Running
# abc5    web.5      server2  Running        Running 5 minutes ago
# NEW2    web.5      server1  Running        Running 10 seconds ago  ← NEW!
# abc6    web.6      server3  Running        Running

# Swarm detected 2 replicas lost (server2 down)
# Automatically rescheduled to server1 and server3!
# 6 replicas still running (desired state maintained)
```

**6. Deploy stack (multi-service app)**:
```yaml
# stack.yml
version: '3.8'
services:
  web:
    image: nginx:alpine
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
    ports:
      - "80:80"

  api:
    image: myapi:latest
    deploy:
      replicas: 5
    networks:
      - backend

networks:
  backend:
    driver: overlay

volumes:
  db-data:
```

```bash
# Deploy entire stack
docker stack deploy -c stack.yml myapp

# Check all services
docker stack services myapp
# NAME        MODE         REPLICAS
# myapp_web   replicated   3/3
# myapp_api   replicated   5/5

# Remove stack (all services)
docker stack rm myapp
```

**Output**: 6 nginx replicas distributed across 3 nodes. Node failure → automatic rescheduling. Stack deploys multi-service app with one command.

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Manager Node Failure in Single-Manager Cluster

**Symptom**: Single manager node fails, entire cluster becomes unusable (can't deploy, scale, update services).

**What I did wrong**:
> Ran production with 1 manager node (no HA). Manager crashed, cluster "frozen" - workers keep running existing containers but can't deploy new services.

**Why wrong**:
> Managers maintain cluster state via Raft consensus. 1 manager = single point of failure. No quorum possible with 1 node.

**Solution**:
```bash
# ✅ Always run 3 managers minimum (prod)
docker swarm init  # Manager 1
docker swarm join-token manager  # Get token

# On server2 and server3 - join as managers
docker swarm join --token SWMTKN-MANAGER-xyz... 192.168.1.10:2377

# Verify 3 managers
docker node ls
# 3 managers → Tolerates 1 manager failure (quorum = 2)
```

**Lesson**: Production Swarm needs 3-5 manager nodes for HA - never run single manager

---

### Pitfall 2: Published Port Conflicts

**Symptom**: Can't create service, error: "port already in use".

**What I did wrong**:
```bash
# ❌ Wrong - published port 80 conflicts across replicas
docker service create --replicas 3 --publish 80:80 nginx
# Error: port 80 already allocated (multiple replicas can't bind same host port)
```

**Why wrong**:
> `--publish 80:80` maps host port 80 to container. If 2 replicas on same node, both try to bind port 80 → conflict.

**Solution**:
```bash
# ✅ Swarm routing mesh handles this automatically
docker service create --replicas 3 --publish 80:80 nginx
# Swarm ingress network routes port 80 to available replica
# Multiple replicas on same node = OK (routing mesh distributes traffic)

# Or use global mode (1 replica per node max)
docker service create --mode global --publish 80:80 nginx
```

**Lesson**: Swarm routing mesh allows multiple replicas with same published port - trust Swarm's load balancing

---

### Pitfall 3: Forgetting to Use Overlay Network

**Symptom**: Services can't communicate across nodes ("connection refused").

**What I did wrong**:
> Used default bridge network (single-host). Service replicas on different nodes can't reach each other.

**Solution**:
```bash
# ✅ Create overlay network (multi-host)
docker network create --driver overlay my-overlay

# Deploy services on overlay network
docker service create --network my-overlay --name api myapi
docker service create --network my-overlay --name web nginx

# web replica on node1 can reach api replica on node2!
# (see [[docker-swarm-overlay-networks]] for details)
```

**Lesson**: Swarm services need overlay networks for multi-host communication - bridge is single-host only

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> What's the difference between deploying a container with docker run and deploying a service with docker service create?</summary>

**Answer**: `docker run` creates a single container on the current host (imperative). `docker service create` declares desired state (replicas, image) and Swarm distributes across cluster, ensures desired replicas always running, handles failures automatically (declarative). Service = production-ready with HA, container = dev/single-host.
</details>

<details>
<summary><strong>Q2:</strong> Why do you need 3 manager nodes instead of 1 in production Swarm?</summary>

**Answer**: Managers maintain cluster state via Raft consensus (requires quorum). 1 manager = single point of failure (manager crashes = cluster frozen). 3 managers tolerates 1 failure (quorum = 2 remaining). 5 managers tolerates 2 failures. Recommendation: 3 managers minimum for prod HA.
</details>

<details>
<summary><strong>Q3:</strong> How does Swarm handle desired state reconciliation when a node fails?</summary>

**Answer**: You declare desired replicas (`--replicas 5`). Node fails → some replicas lost. Swarm detects via health checks, automatically reschedules lost replicas to healthy nodes, restores desired state (5 replicas running). Self-healing system - you declare intent, Swarm ensures it's maintained.
</details>

<details>
<summary><strong>Q4:</strong> What's the difference between docker-compose up and docker stack deploy?</summary>

**Answer**: `docker-compose up` deploys to single Docker host (dev). `docker stack deploy` deploys to Swarm cluster with replicas distributed across multiple nodes (prod). Same YAML syntax, but stack adds Swarm-specific options (`deploy`, `replicas`, `update_config`). Compose for dev, stack for prod.
</details>

---

## 📊 Stats

```yaml
Total time: 2h (40% assisted / 60% autonomous)
Used in: [[2025-12-glasck-deployment/learnings]]
```

---

**Last update**: 2025-12-26
**Next review**: 2026-01-26 (+1 month)
