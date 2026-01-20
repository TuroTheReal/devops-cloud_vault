# MOC - Docker for Production Deployments

> **What**: From containers basics to production-ready orchestration with Docker Swarm
>
> **Who**: DevOps engineers deploying multi-container applications to production
>
> **Why**: Deploy scalable, resilient applications with automated recovery and zero-downtime updates

---

## 🎯 Learning Objective

Master Docker from fundamentals to production orchestration. You'll understand why containers solve the "works on my machine" problem, manage persistent data, orchestrate multi-container apps, and deploy to production clusters with high availability. After this path, you'll confidently deploy containerized applications with proper networking, data persistence, and automated failover.

**Real-world use case**: Deploy a full-stack application (React frontend, Node.js API, PostgreSQL database) to a 3-node Docker Swarm cluster with automated scaling, rolling updates, health checks, and data persistence - exactly how modern SaaS applications run in production.

---

## 📚 Learning Path

### Phase 1: Container Fundamentals (20h total)

**Goal**: Understand containers, images, and why they're critical for modern deployments

1. **[[docker-why-containers]]** ⭐ (2h)
   - **Why now**: Foundation - understand the problem containers solve before learning implementation
   - **You'll learn**: VMs vs containers, portability, "works on my machine" solution, resource efficiency

2. **[[docker-containers-lifecycle]]** ⭐⭐ (5h)
   - **Why now**: After understanding "why", learn "how" - manage container lifecycle
   - **You'll learn**: Create, start, stop containers, inspect logs, execute commands inside containers, port mapping

3. **[[docker-images-layers]]** ⭐⭐⭐ (8h)
   - **Why now**: Containers run from images - understand image architecture for efficient builds
   - **You'll learn**: Layer caching, Dockerfile best practices, multi-stage builds, image optimization, build context

4. **[[docker-volumes-persistence]]** ⭐⭐ (5h)
   - **Why now**: Containers are ephemeral - critical to understand data persistence before production
   - **You'll learn**: Named volumes, bind mounts, tmpfs, when to use each, data lifecycle

**✅ Checkpoint**: You can build optimized images, manage containers, persist data correctly.

---

### Phase 2: Multi-Container Orchestration (15h total)

**Goal**: Orchestrate applications with multiple interconnected services

5. **[[docker-compose-basics]]** ⭐⭐ (6h)
   - **Why now**: Single containers are rare - most apps need database, cache, API, frontend
   - **You'll learn**: docker-compose.yml syntax, service communication, networks, volumes, depends_on, restart policies (always/on-failure/unless-stopped)

6. **[[docker-compose-healthchecks]]** ⭐⭐ (3h)
   - **Why now**: depends_on controls start order, not readiness - need healthchecks for reliability
   - **You'll learn**: Define healthchecks, wait for service ready (not just started), avoid connection errors

7. **[[docker-network-isolation]]** ⭐⭐⭐ (6h)
   - **Why now**: After connecting services, learn to isolate them for security
   - **You'll learn**: Bridge vs overlay networks, service isolation, internal vs external networks, architecture public/private (frontend exposé vs backend isolé sur même infra)

**✅ Checkpoint**: You can deploy multi-container apps with proper networking, healthchecks, and isolation.

---

### Phase 3: Production Clustering (22h total)

**Goal**: Deploy to production with high availability and automated recovery

8. **[[docker-swarm-basics]]** ⭐⭐⭐⭐ (10h)
   - **Why now**: Compose is dev-only - production needs clustering, scaling, failover
   - **You'll learn**: Managers vs workers, services vs containers, desired state reconciliation, stacks, service scaling

9. **[[docker-swarm-overlay-networks]]** ⭐⭐⭐⭐ (4h)
   - **Why now**: Swarm services span multiple nodes - need multi-host networking
   - **You'll learn**: Overlay networks, service discovery across nodes, routing mesh, load balancing

10. **[[docker-swarm-deployment-strategies]]** ⭐⭐⭐⭐ (8h)
    - **Why now**: After basic Swarm, learn production deployment patterns
    - **You'll learn**: Rolling updates (update_config), restart_policy (condition, delay, max_attempts), rollback strategies, update parallelism, blue-green deployments

**✅ Checkpoint**: You can deploy to a Swarm cluster with rolling updates, auto-restart, and rollback.

---

### Phase 4: Production Infrastructure (11h total)

**Goal**: Add reverse proxy, SSL, and backup strategies for production-ready setup

11. **[[traefik-integration]]** ⭐⭐⭐⭐ (8h)
    - **Why now**: Production apps need HTTPS, routing, SSL termination - Traefik automates this
    - **You'll learn**: Automatic service discovery, Let's Encrypt SSL, path-based routing, load balancing, middleware basics

12. **[[docker-backup-strategies]]** ⭐⭐⭐ (3h)
    - **Why now**: Data loss = game over - plan backups before disaster
    - **You'll learn**: Volume backup, image export/compression, automation scripts, restore testing
    - **Current status**: Semi-auto via Taskfile, image compression, volumes datés (Prometheus, Grafana). Pas encore full auto. DB non géré (bare metal, pas de container DB pour l'instant).
    - **Next step**: Automatisation complète, stratégie off-site, backup DB si containerisé

**✅ Checkpoint**: You have a production-ready setup with SSL, reverse proxy, and backup strategy.

---

## 🛠️ Practice Project

**Apply your knowledge**: [[2025-12-glasck-deployment/learnings]]

**What you'll build**: Deploy a full-stack application with Docker orchestration, featuring Traefik reverse proxy, Redis caching, automated health checks, rolling updates, and zero-downtime deployments.

**Why this project**: Combines all 12 concepts in production scenario - multi-container app, data persistence, service networking, orchestration, automated recovery, SSL termination. This is exactly how you'd deploy a real SaaS application.

---

## 📊 Progress Tracking

| Phase | Concepts | Time |
|-------|----------|------|
| Phase 1: Fundamentals | 4 | 20h |
| Phase 2: Multi-Container | 3 | 15h |
| Phase 3: Clustering | 3 | 22h |
| Phase 4: Production | 2 | 11h |
| **TOTAL** | **12** | **68h** |

```yaml
Total estimated time: 68h
Difficulty: ⭐⭐⭐⭐ (4/5) - Swarm et Traefik sont complexes pour un débutant
Prerequisites: [[linux-basics]], [[networking-fundamentals]] (understand ports, IPs)
```

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **[[MOC-Kubernetes-Fundamentals]]**: More powerful orchestration (complex but industry standard)
- **[[MOC-CICD-Pipeline]]**: Automate Docker image builds and deployments with GitHub Actions
- **[[MOC-Monitoring-Observability]]**: Add Prometheus/Grafana for container metrics and alerts

---

## 🔧 Troubleshooting

**Common issues and solutions**:

1. **[[troubleshooting/docker/restart-config-not-applied]]** ⭐⭐
   - Container config not applied after VPS reboot
   - `docker restart` ≠ `docker-compose up -d`

2. **[[troubleshooting/docker/disk-full-logs]]** ⭐⭐
   - Disk full from container logs
   - Log rotation and limits

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **Containers share kernel**: Not VMs - lighter (MB vs GB) but less isolated. Can't run Windows app on Linux container.
2. **Ephemeral filesystem**: Data written to container filesystem disappears on restart - use volumes for persistence
3. **Service names = DNS**: In Compose/Swarm, services reach each other by name (db, redis) - never localhost
4. **Desired state**: In Swarm, declare "I want 5 replicas" - Swarm ensures it's maintained even after node failures
5. **Overlay networks**: Required for multi-host communication in Swarm - bridge networks are single-host only
6. **Restart policy**: `always` (prod), `on-failure` (dev), `unless-stopped` (flexible) - configure BEFORE disaster

**Common pitfalls**:
- Storing data in container filesystem (use named volumes for databases, bind mounts for dev)
- Using localhost for inter-service communication (use service names - db, redis, api)
- depends_on without healthchecks (controls start order, not readiness - add healthchecks!)
- Single manager in Swarm (always 3-5 managers for HA - 1 manager = single point of failure)
- Forgetting to use overlay networks in Swarm (services on different nodes can't communicate without overlay)
- Publishing same port multiple times (Swarm routing mesh handles this - trust it!)
- Assuming containers are mini-VMs (containers run processes, not operating systems - process exits = container exits)
- Using `docker restart` instead of `docker-compose up -d` (restart uses frozen config, up applies current compose)
- No backup strategy (data loss is permanent - automate backups BEFORE you need them)

---

**Created**: 2025-12-26
**Last updated**: 2026-01-20
