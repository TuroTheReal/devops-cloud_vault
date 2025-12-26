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

### Phase 1: Container Fundamentals (3h30 total)

**Goal**: Understand containers, images, and why they're critical for modern deployments

1. **[[docker-why-containers]]** ⭐ (30min)
   - **Why now**: Foundation - understand the problem containers solve before learning implementation
   - **You'll learn**: VMs vs containers, portability, "works on my machine" solution, resource efficiency

2. **[[docker-containers-lifecycle]]** ⭐⭐ (1h)
   - **Why now**: After understanding "why", learn "how" - manage container lifecycle
   - **You'll learn**: Create, start, stop containers, inspect logs, execute commands inside containers

3. **[[docker-images-layers]]** ⭐⭐ (1h)
   - **Why now**: Containers run from images - understand image architecture for efficient builds
   - **You'll learn**: Layer caching, Dockerfile best practices, multi-stage builds, image optimization

4. **[[docker-volumes-persistence]]** ⭐⭐ (1h)
   - **Why now**: Containers are ephemeral - critical to understand data persistence before production
   - **You'll learn**: Named volumes, bind mounts, tmpfs, when to use each, data lifecycle

---

### Phase 2: Multi-Container Orchestration (3h total)

**Goal**: Orchestrate applications with multiple interconnected services

5. **[[docker-compose-basics]]** ⭐⭐ (1h30)
   - **Why now**: Single containers are rare - most apps need database, cache, API, frontend
   - **You'll learn**: docker-compose.yml syntax, service communication, networks, volumes, depends_on

6. **[[docker-compose-healthchecks]]** ⭐⭐ (1h)
   - **Why now**: depends_on controls start order, not readiness - need healthchecks for reliability
   - **You'll learn**: Define healthchecks, wait for service ready (not just started), avoid connection errors

7. **[[docker-network-isolation]]** ⭐⭐ (30min)
   - **Why now**: After connecting services, learn to isolate them for security
   - **You'll learn**: Bridge vs overlay networks, service isolation, internal vs external networks

---

### Phase 3: Production Clustering (4h30 total)

**Goal**: Deploy to production with high availability and automated recovery

8. **[[docker-swarm-basics]]** ⭐⭐⭐ (2h)
   - **Why now**: Compose is dev-only - production needs clustering, scaling, failover
   - **You'll learn**: Managers vs workers, services vs containers, desired state reconciliation, stacks

9. **[[docker-swarm-overlay-networks]]** ⭐⭐⭐ (1h)
   - **Why now**: Swarm services span multiple nodes - need multi-host networking
   - **You'll learn**: Overlay networks, service discovery across nodes, routing mesh, load balancing

10. **[[docker-swarm-deployment-strategies]]** ⭐⭐⭐ (1h30)
    - **Why now**: After basic Swarm, learn production deployment patterns
    - **You'll learn**: Rolling updates, blue-green deployments, rollback strategies, update parallelism

---

### Phase 4: Production Infrastructure (2h total)

**Goal**: Add reverse proxy and monitoring for production-ready setup

11. **[[traefik-swarm-integration]]** ⭐⭐⭐ (2h)
    - **Why now**: Production apps need HTTPS, routing, SSL termination - Traefik automates this
    - **You'll learn**: Automatic service discovery, Let's Encrypt SSL, path-based routing, load balancing

---

## 🛠️ Practice Project

**Apply your knowledge**: [[2025-12-glasck-deployment/learnings]]

**What you'll build**: Deploy a full-stack application with Docker orchestration, featuring Traefik reverse proxy, Redis caching, automated health checks, rolling updates, and zero-downtime deployments.

**Why this project**: Combines all 11 concepts in production scenario - multi-container app, data persistence, service networking, orchestration, automated recovery, SSL termination. This is exactly how you'd deploy a real SaaS application.

---

## 📊 Progress Tracking

```yaml
Total estimated time: 13h
Difficulty: ⭐⭐⭐ (3/5)
Prerequisites: [[linux-basics]], [[networking-fundamentals]] (understand ports, IPs)
```

---

## 🔄 Next Steps

After completing this path, you'll be ready for:
- **Kubernetes Basics**: More powerful orchestration (complex but industry standard)
- **CI/CD Pipeline**: Automate Docker image builds and deployments with GitHub Actions
- **Monitoring & Observability**: Add Prometheus/Grafana for container metrics and alerts

---

## 📖 Quick Reference

**Core concepts to remember**:
1. **Containers share kernel**: Not VMs - lighter (MB vs GB) but less isolated. Can't run Windows app on Linux container.
2. **Ephemeral filesystem**: Data written to container filesystem disappears on restart - use volumes for persistence
3. **Service names = DNS**: In Compose/Swarm, services reach each other by name (db, redis) - never localhost
4. **Desired state**: In Swarm, declare "I want 5 replicas" - Swarm ensures it's maintained even after node failures
5. **Overlay networks**: Required for multi-host communication in Swarm - bridge networks are single-host only

**Common pitfalls**:
- Storing data in container filesystem (use named volumes for databases, bind mounts for dev)
- Using localhost for inter-service communication (use service names - db, redis, api)
- depends_on without healthchecks (controls start order, not readiness - add healthchecks!)
- Single manager in Swarm (always 3-5 managers for HA - 1 manager = single point of failure)
- Forgetting to use overlay networks in Swarm (services on different nodes can't communicate without overlay)
- Publishing same port multiple times (Swarm routing mesh handles this - trust it!)
- Assuming containers are mini-VMs (containers run processes, not operating systems - process exits = container exits)

---

**Created**: 2025-12-26
**Last updated**: 2025-12-26
