# Docker - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, docker, containers, images]
created: 2025-12-23
updated: 2025-12-23
```

**Related**: [[docker-compose-basics]], [[docker-swarm-basics]], [[concepts/docker/*]]

---

## 📚 Table of Contents

1. [Quick Start](#-quick-start)
2. [Container Management](#-container-management)
3. [Image Management](#️-image-management)
4. [Network Management](#-network-management)
5. [Volume Management](#-volume-management)
6. [Dockerfile Best Practices](#️-dockerfile-best-practices)
7. [Debugging & Troubleshooting](#-debugging--troubleshooting)
8. [Cleanup & Maintenance](#-cleanup--maintenance)
9. [Docker Registry](#-docker-registry)
10. [Useful One-Liners](#-useful-one-liners)
11. [Security Best Practices](#-security-best-practices)
12. [Quick Reference](#-quick-reference)
13. [Best Practices Summary](#-best-practices-summary)

---

## 🚀 Quick Start

```bash
# Run container
docker run -d -p 80:80 --name web nginx

# List running containers
docker ps

# View logs
docker logs web

# Stop container
docker stop web

# Remove container
docker rm web
```

---

## 📦 Container Management

### Run Containers
```bash
# Basic run
docker run nginx

# Detached mode (background)
docker run -d nginx

# Interactive mode with shell
docker run -it ubuntu bash

# With name
docker run -d --name myapp nginx

# With port mapping
docker run -d -p 8080:80 nginx

# With multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx

# With environment variables
docker run -d -e NODE_ENV=production -e PORT=3000 myapp

# With volume mount
docker run -d -v /host/path:/container/path nginx

# With bind mount (current directory)
docker run -d -v $(pwd):/app node

# With network
docker run -d --network mynetwork nginx

# With resource limits
docker run -d --memory="512m" --cpus="1.5" nginx

# Remove container after exit
docker run --rm alpine echo "Hello"

# With restart policy
docker run -d --restart unless-stopped nginx

# Read-only filesystem
docker run -d --read-only nginx
```

### Container Operations
```bash
# List running containers
docker ps

# List all containers (including stopped)
docker ps -a

# List with custom format
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Stop container
docker stop myapp

# Stop with timeout
docker stop -t 30 myapp

# Start stopped container
docker start myapp

# Restart container
docker restart myapp

# Pause container
docker pause myapp
docker unpause myapp

# Kill container (force stop)
docker kill myapp

# Remove container
docker rm myapp

# Remove running container (force)
docker rm -f myapp

# Remove all stopped containers
docker container prune
docker container prune -f  # Without confirmation
```

### Container Inspection
```bash
# View container logs
docker logs myapp

# Follow logs (real-time)
docker logs -f myapp

# Last 100 lines
docker logs --tail 100 myapp

# Logs with timestamps
docker logs -t myapp

# Since specific time
docker logs --since 2024-12-23T10:00:00 myapp

# Inspect container (full details)
docker inspect myapp

# Get specific field
docker inspect --format='{{.State.Status}}' myapp
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp

# Container stats (real-time)
docker stats myapp

# All containers stats
docker stats

# Container processes
docker top myapp

# Container resource usage
docker container diff myapp
```

### Execute Commands in Container
```bash
# Execute command
docker exec myapp ls /app

# Interactive shell
docker exec -it myapp sh
docker exec -it myapp bash

# As specific user
docker exec -u root -it myapp bash

# With environment variable
docker exec -e DEBUG=true myapp node script.js

# Run in working directory
docker exec -w /app myapp npm test
```

### Copy Files
```bash
# Copy from container to host
docker cp myapp:/app/logs/app.log ./logs/

# Copy from host to container
docker cp ./config.json myapp:/app/config.json

# Copy entire directory
docker cp myapp:/var/log ./logs/
```

---

## 🖼️ Image Management

### Build Images
```bash
# Build from Dockerfile
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build args
docker build --build-arg NODE_VERSION=18 -t myapp .

# Build with no cache
docker build --no-cache -t myapp .

# Build specific target (multi-stage)
docker build --target production -t myapp:prod .

# Build with tag and push
docker build -t username/myapp:v1.0.0 .
docker push username/myapp:v1.0.0
```

### Image Operations
```bash
# List images
docker images

# List with digests
docker images --digests

# List image IDs only
docker images -q

# Pull image
docker pull nginx
docker pull nginx:1.25-alpine

# Pull from specific registry
docker pull myregistry.com/myapp:latest

# Tag image
docker tag myapp:latest myapp:v1.0.0
docker tag myapp:latest username/myapp:latest

# Push image
docker push username/myapp:latest

# Save image to tar
docker save myapp:latest -o myapp.tar

# Load image from tar
docker load -i myapp.tar

# Remove image
docker rmi myapp:latest

# Remove unused images
docker image prune

# Remove all unused images (not just dangling)
docker image prune -a

# Remove images matching pattern
docker images | grep myapp | awk '{print $3}' | xargs docker rmi
```

### Image Inspection
```bash
# Inspect image
docker inspect nginx:latest

# View image layers
docker history nginx:latest

# View image layers with size
docker history --no-trunc nginx:latest

# Get image creation date
docker inspect --format='{{.Created}}' nginx:latest

# Get image size
docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

---

## 🌐 Network Management

### Network Operations
```bash
# List networks
docker network ls

# Create network
docker network create mynetwork

# Create with subnet
docker network create --subnet=172.18.0.0/16 mynetwork

# Create bridge network
docker network create --driver bridge mybridge

# Inspect network
docker network inspect mynetwork

# Connect container to network
docker network connect mynetwork myapp

# Disconnect container from network
docker network disconnect mynetwork myapp

# Remove network
docker network rm mynetwork

# Remove all unused networks
docker network prune
```

### Common Network Patterns
```bash
# Create custom bridge with DNS
docker network create \
  --driver bridge \
  --subnet 172.20.0.0/16 \
  --gateway 172.20.0.1 \
  myapp-network

# Run container on custom network
docker run -d --network myapp-network --name api myapp-api
docker run -d --network myapp-network --name web myapp-web

# Containers can now reach each other by name
# From web container: curl http://api:3000
```

---

## 💾 Volume Management

### Volume Operations
```bash
# List volumes
docker volume ls

# Create volume
docker volume create mydata

# Create with driver options
docker volume create --driver local \
  --opt type=none \
  --opt device=/path/on/host \
  --opt o=bind \
  mydata

# Inspect volume
docker volume inspect mydata

# Remove volume
docker volume rm mydata

# Remove all unused volumes
docker volume prune

# Remove all volumes (DANGEROUS)
docker volume rm $(docker volume ls -q)
```

### Volume Usage
```bash
# Named volume
docker run -d -v mydata:/app/data nginx

# Bind mount
docker run -d -v /host/path:/container/path nginx

# Read-only volume
docker run -d -v mydata:/app/data:ro nginx

# Anonymous volume
docker run -d -v /app/data nginx

# Copy data to volume
docker run --rm -v mydata:/data -v $(pwd):/backup alpine \
  cp /backup/data.txt /data/
```

---

## 🏗️ Dockerfile Best Practices

### Efficient Dockerfile
```dockerfile
# Use specific version, not latest
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (cache optimization)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Use non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Expose port (documentation)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "server.js"]
```

### Multi-Stage Build
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

---

## 🔍 Debugging & Troubleshooting

### Logs & Events
```bash
# Container logs
docker logs -f --tail 100 myapp

# Docker daemon logs (Ubuntu/Debian)
journalctl -u docker.service

# Docker events (real-time)
docker events

# Filter events
docker events --filter type=container
docker events --filter event=start
docker events --filter container=myapp
```

### Inspect & Debug
```bash
# Why is container stopped?
docker inspect --format='{{.State.Status}}' myapp
docker inspect --format='{{.State.ExitCode}}' myapp

# Container IP address
docker inspect --format='{{.NetworkSettings.IPAddress}}' myapp

# Environment variables
docker inspect --format='{{range .Config.Env}}{{println .}}{{end}}' myapp

# Mounted volumes
docker inspect --format='{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}' myapp

# Debug failed container
docker logs myapp
docker inspect myapp
docker events --filter container=myapp

# Access container filesystem (even if stopped)
docker export myapp -o myapp.tar
tar -xf myapp.tar

# Check container disk usage
docker exec myapp df -h

# Check container processes
docker top myapp
```

### Resource Usage
```bash
# Real-time stats
docker stats

# Disk usage summary
docker system df

# Detailed disk usage
docker system df -v

# Check specific container size
docker ps -s --filter name=myapp

# See which images are used by containers
docker ps --format '{{.Image}}' | sort | uniq
```

---

## 🧹 Cleanup & Maintenance

### Remove Resources
```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune

# Remove unused volumes
docker volume prune

# Remove unused networks
docker network prune

# Remove everything unused
docker system prune

# Remove everything (including volumes)
docker system prune -a --volumes

# Remove all stopped containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Remove dangling images
docker rmi $(docker images -f "dangling=true" -q)

# Remove images older than 24h
docker image prune -a --filter "until=24h"
```

### Scheduled Cleanup
```bash
# Cron job for daily cleanup (example)
0 2 * * * docker system prune -af --filter "until=24h" >> /var/log/docker-cleanup.log 2>&1
```

---

## 🔧 Docker Registry

### Docker Hub
```bash
# Login
docker login

# Login to specific registry
docker login myregistry.com

# Logout
docker logout

# Push image
docker push username/myapp:v1.0.0

# Pull image
docker pull username/myapp:v1.0.0

# Search Docker Hub
docker search nginx
```

### Private Registry
```bash
# Tag for private registry
docker tag myapp:latest myregistry.com/myapp:latest

# Push to private registry
docker push myregistry.com/myapp:latest

# Pull from private registry
docker pull myregistry.com/myapp:latest

# Run local registry
docker run -d -p 5000:5000 --name registry registry:2

# Push to local registry
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest
```

---

## 📊 Useful One-Liners

```bash
# Stop all running containers
docker stop $(docker ps -q)

# Remove all containers
docker rm $(docker ps -aq)

# Remove all images
docker rmi $(docker images -q)

# Get container IP addresses
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -q)

# See container port mappings
docker ps --format "table {{.Names}}\t{{.Ports}}"

# Follow logs of multiple containers
docker logs -f container1 & docker logs -f container2

# Container memory usage sorted
docker stats --no-stream --format "table {{.Name}}\t{{.MemUsage}}" | sort -k 2 -h

# Check if container is running
docker ps -q -f name=myapp | grep -q . && echo "Running" || echo "Stopped"

# Count running containers
docker ps -q | wc -l

# Size of all containers
docker ps -as --format "table {{.Names}}\t{{.Size}}"

# Recent container changes
docker diff myapp
```

---

## 🔐 Security Best Practices

```bash
# Run as non-root user
docker run -u 1000:1000 myapp

# Read-only root filesystem
docker run --read-only myapp

# Drop capabilities
docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE myapp

# No new privileges
docker run --security-opt=no-new-privileges myapp

# Scan image for vulnerabilities (with Trivy)
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image myapp:latest

# Check image for best practices (with Dive)
docker run --rm -it -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest myapp:latest
```

---

## 🎯 Quick Reference

### Container Lifecycle
```
create → start → running → stop → stopped → remove
         └─────────────────┘
              restart
```

### Common Exit Codes
- `0`: Success
- `1`: Application error
- `137`: Killed (OOM or `docker kill`)
- `139`: Segmentation fault
- `143`: Graceful termination (SIGTERM)

### Resource Limits
```bash
# Memory
--memory="512m"     # Max 512MB
--memory-swap="1g"  # Total memory (RAM + swap)

# CPU
--cpus="1.5"        # 1.5 CPU cores
--cpu-shares=512    # Relative weight

# I/O
--blkio-weight=500  # Block IO weight (10-1000)
```

---

## 📚 Best Practices Summary

1. **Images**: Use specific tags, not `latest`
2. **Layers**: Minimize layers, combine RUN commands
3. **Cache**: Order Dockerfile for optimal caching
4. **Size**: Use Alpine base images where possible
5. **Security**: Run as non-root user
6. **Logs**: Write to stdout/stderr, not files
7. **Data**: Use volumes for persistent data
8. **Cleanup**: Regular pruning of unused resources
9. **Health**: Add HEALTHCHECK to Dockerfiles
10. **Secrets**: Never store secrets in images

---

**Last updated**: 2025-12-23
**Source**: Docker best practices & real-world usage
