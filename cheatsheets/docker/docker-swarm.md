# Docker Swarm - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, docker, swarm, orchestration]
created: 2025-12-23
updated: 2025-12-23
```

**Related**: [[concepts/docker/docker-swarm-*]], [[docker-compose]]

---

## 🚀 Quick Start

```bash
# Initialize Swarm (on manager node)
docker swarm init

# Join worker node
docker swarm join --token <token> <manager-ip>:2377

# Deploy stack
docker stack deploy -c stack.yml myapp

# List services
docker service ls

# View service logs
docker service logs myapp_web
```

---

## 📦 Stack Management

### Deploy & Update
```bash
# Deploy stack from compose file
docker stack deploy -c docker-compose.yml mystack

# Deploy with environment substitution
export VERSION=1.2.0
docker stack deploy -c stack.yml mystack

# Update stack (redeploy)
docker stack deploy -c stack.yml mystack

# Remove stack
docker stack rm mystack

# List all stacks
docker stack ls

# List services in stack
docker stack services mystack

# View tasks in stack
docker stack ps mystack
```

### Service Operations
```bash
# List all services
docker service ls

# Inspect service
docker service inspect mystack_web --pretty

# View service logs
docker service logs mystack_web
docker service logs -f mystack_web  # Follow
docker service logs --tail 50 mystack_web

# Scale service
docker service scale mystack_web=3

# Update service image
docker service update --image myapp:v2 mystack_web

# Force update (without image change)
docker service update --force mystack_web

# Rollback service
docker service rollback mystack_web

# Remove service
docker service rm mystack_web
```

### Task Management
```bash
# List tasks for service
docker service ps mystack_web

# Include shutdown tasks
docker service ps --no-trunc mystack_web

# Filter running tasks only
docker service ps --filter "desired-state=running" mystack_web

# View task history
docker service ps --no-trunc --format "table {{.Name}}\t{{.Node}}\t{{.CurrentState}}\t{{.Error}}" mystack_web
```

---

## 🌐 Network Management

### Overlay Networks
```bash
# Create overlay network
docker network create --driver overlay mynetwork

# Create attachable overlay (for standalone containers)
docker network create --driver overlay --attachable mynetwork

# Create with custom MTU (recommended for production)
docker network create \
  --driver overlay \
  --attachable \
  --opt com.docker.network.driver.mtu=1450 \
  mynetwork

# List networks
docker network ls

# Inspect network
docker network inspect mynetwork

# Remove network
docker network rm mynetwork

# Prune unused networks
docker network prune
```

---

## 🏷️ Common Deploy Labels (Traefik)

### Basic Service Exposure
```yaml
deploy:
  labels:
    - "traefik.enable=true"
    - "traefik.docker.network=mystack_public"
    - "traefik.http.routers.myapp.rule=Host(`example.com`)"
    - "traefik.http.routers.myapp.entrypoints=websecure"
    - "traefik.http.routers.myapp.tls=true"
    - "traefik.http.services.myapp.loadbalancer.server.port=3000"
```

### HTTP → HTTPS Redirect
```yaml
deploy:
  labels:
    # HTTP Router (redirect)
    - "traefik.http.routers.myapp-http.rule=Host(`example.com`)"
    - "traefik.http.routers.myapp-http.entrypoints=web"
    - "traefik.http.routers.myapp-http.middlewares=redirect-to-https"

    # HTTPS Router (main)
    - "traefik.http.routers.myapp-https.rule=Host(`example.com`)"
    - "traefik.http.routers.myapp-https.entrypoints=websecure"
    - "traefik.http.routers.myapp-https.tls=true"

    # Middleware
    - "traefik.http.middlewares.redirect-to-https.redirectscheme.scheme=https"
    - "traefik.http.middlewares.redirect-to-https.redirectscheme.permanent=true"
```

---

## 🔄 Deployment Strategies

### Zero-Downtime Update
```yaml
deploy:
  replicas: 3
  update_config:
    parallelism: 1           # Update 1 at a time
    delay: 10s               # Wait between updates
    failure_action: rollback # Auto-rollback on failure
    monitor: 60s             # Watch for 60s after start
    max_failure_ratio: 0     # Zero tolerance
    order: start-first       # New starts before old stops

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

---

## 🏥 Healthchecks

### Service Healthcheck
```yaml
services:
  web:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 40s  # Grace period for startup
```

### Common Healthcheck Commands
```yaml
# HTTP endpoint
test: ["CMD", "curl", "-f", "http://localhost:3000/health"]

# Database
test: ["CMD", "pg_isready", "-U", "postgres"]

# Redis
test: ["CMD", "redis-cli", "ping"]

# Custom script
test: ["CMD-SHELL", "node /app/healthcheck.js"]
```

---

## 🎯 Placement & Constraints

```yaml
deploy:
  # Node role constraint
  placement:
    constraints:
      - node.role == manager
      - node.role == worker

  # Node label constraint
  placement:
    constraints:
      - node.labels.type == frontend

  # Node hostname
  placement:
    constraints:
      - node.hostname == node1

  # Multiple constraints (AND)
  placement:
    constraints:
      - node.role == worker
      - node.labels.zone == us-east-1

  # Preferences (soft constraints)
  placement:
    preferences:
      - spread: node.labels.zone  # Spread across zones
```

---

## 💾 Resource Limits

```yaml
deploy:
  resources:
    limits:
      cpus: '2.0'       # Max 2 CPUs
      memory: 4G        # Max 4GB RAM

    reservations:
      cpus: '0.5'       # Reserve 0.5 CPU
      memory: 1G        # Reserve 1GB RAM
```

---

## 🔍 Debugging & Inspection

```bash
# Inspect service (JSON)
docker service inspect mystack_web

# Pretty format
docker service inspect --pretty mystack_web

# Get specific field
docker service inspect mystack_web --format '{{.Spec.TaskTemplate.ContainerSpec.Image}}'

# View service events
docker events --filter type=service

# Check why task failed
docker service ps mystack_web --no-trunc

# Execute command in service container
TASK_ID=$(docker service ps -q mystack_web | head -n1)
CONTAINER_ID=$(docker inspect --format '{{.Status.ContainerStatus.ContainerID}}' $TASK_ID)
docker exec -it $CONTAINER_ID sh

# View network details
docker network inspect mystack_public | jq '.[0].Containers'

# Check node availability
docker node ls

# Inspect node
docker node inspect node1 --pretty
```

---

## 🔧 Node Management

```bash
# List nodes
docker node ls

# Inspect node
docker node inspect node1 --pretty

# Add label to node
docker node update --label-add type=frontend node1

# Remove label
docker node update --label-rm type node1

# Drain node (for maintenance)
docker node update --availability drain node1

# Make node active again
docker node update --availability active node1

# Remove node
docker node rm node1

# Promote worker to manager
docker node promote node1

# Demote manager to worker
docker node demote node1
```

---

## 📊 Monitoring

```bash
# Service stats (real-time)
docker stats $(docker ps --filter "label=com.docker.swarm.service.name=mystack_web" -q)

# Watch service status
watch -n 1 'docker service ps mystack_web'

# Check service replicas
docker service ls --format "table {{.Name}}\t{{.Replicas}}"

# View service update status
docker service inspect mystack_web --format '{{.UpdateStatus}}'
```

---

## 🚨 Common Issues & Solutions

### Service Won't Start
```bash
# Check task error
docker service ps mystack_web --no-trunc

# Check logs
docker service logs mystack_web

# Inspect service config
docker service inspect mystack_web --pretty
```

### Network Connectivity Issues
```bash
# Verify network exists
docker network ls | grep mynetwork

# Check service network
docker service inspect mystack_web --format '{{json .Spec.TaskTemplate.Networks}}' | jq

# Test connectivity from container
docker exec <container> ping <service-name>
docker exec <container> nslookup <service-name>
```

### Update Stuck
```bash
# Check update status
docker service inspect mystack_web --format '{{json .UpdateStatus}}'

# Cancel update
docker service update --rollback mystack_web

# Force new deployment
docker service update --force mystack_web
```

---

## 📚 Best Practices

1. **Networks**: Always set MTU to 1450 for overlay networks
2. **Healthchecks**: Use appropriate `start_period` for slow-starting apps
3. **Updates**: Use `start-first` + `monitor > start_period` for safety
4. **Multi-network**: Specify `traefik.docker.network` label explicitly
5. **Ports**: Use `mode: host` for Traefik to preserve client IPs
6. **Placement**: Run Traefik on manager node for Docker socket access
7. **Resources**: Set both limits and reservations for predictable scheduling

---

**Last updated**: 2025-12-23
**Source**: Extracted from Glasck deployment
