# Docker Compose - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, docker-compose, orchestration]
created: 2025-12-23
updated: 2025-12-23
```

**Related**: [[docker-commands]], [[docker-swarm-basics]], [[concepts/docker/docker-compose-healthchecks]]

---

## 🚀 Quick Start

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down
```

---

## 🎯 Essential Commands

### Basic Operations
```bash
# Start services (detached)
docker-compose up -d

# Start and rebuild images
docker-compose up -d --build

# Start specific service
docker-compose up -d web

# Scale service
docker-compose up -d --scale web=3

# Stop services (keep containers)
docker-compose stop

# Stop and remove containers
docker-compose down

# Stop and remove with volumes
docker-compose down -v

# Stop and remove with images
docker-compose down --rmi all
```

### Service Management
```bash
# List running services
docker-compose ps

# List all services (including stopped)
docker-compose ps -a

# Restart service
docker-compose restart web

# Restart all services
docker-compose restart

# Pause services
docker-compose pause
docker-compose unpause

# Remove stopped containers
docker-compose rm

# Force remove
docker-compose rm -f
```

### Logs & Monitoring
```bash
# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Logs for specific service
docker-compose logs web

# Last 100 lines
docker-compose logs --tail 100

# Logs with timestamps
docker-compose logs -t

# Service stats
docker stats $(docker-compose ps -q)
```

### Build & Images
```bash
# Build images
docker-compose build

# Build without cache
docker-compose build --no-cache

# Build specific service
docker-compose build web

# Pull images
docker-compose pull

# Push images
docker-compose push
```

### Execute Commands
```bash
# Execute command in service
docker-compose exec web sh

# Execute as specific user
docker-compose exec -u root web bash

# Run one-off command
docker-compose run web npm test

# Run without starting dependencies
docker-compose run --no-deps web npm test

# Run and remove container after
docker-compose run --rm web node script.js
```

### Configuration
```bash
# Validate compose file
docker-compose config

# View resolved configuration
docker-compose config --resolve-image-digests

# View only services
docker-compose config --services

# View only volumes
docker-compose config --volumes

# Use specific compose file
docker-compose -f docker-compose.prod.yml up -d

# Use multiple compose files (override)
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Use environment file
docker-compose --env-file .env.prod up -d
```

---

## 📝 Service Configuration

### Basic Service
```yaml
services:
  web:
    image: nginx:alpine
    container_name: my-web
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production
    volumes:
      - ./app:/usr/share/nginx/html
    networks:
      - frontend
    restart: unless-stopped
```

### Build Configuration
```yaml
services:
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
      args:
        NODE_VERSION: "18"
        BUILD_ENV: "production"
      target: production
    image: myapp-api:latest
```

### Advanced Build
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile.prod
      cache_from:
        - myapp:cache
      labels:
        - "version=1.0.0"
        - "environment=production"
```

### Environment Variables
```yaml
services:
  app:
    # Inline
    environment:
      NODE_ENV: production
      PORT: 3000
      DEBUG: "false"

    # From file
    env_file:
      - .env
      - .env.prod

    # From shell
    environment:
      HOST: ${HOST:-localhost}
      API_KEY: ${API_KEY}
```

---

## 🌐 Networking

### Network Configuration
```yaml
services:
  web:
    networks:
      - frontend
      - backend

  api:
    networks:
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true  # No external access
```

### Custom Network
```yaml
networks:
  mynetwork:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: br-myapp
    ipam:
      driver: default
      config:
        - subnet: 172.28.0.0/16
          gateway: 172.28.0.1
```

### External Network
```yaml
networks:
  existing-network:
    external: true
    name: my-existing-network
```

---

## 💾 Volumes

### Volume Types
```yaml
services:
  app:
    volumes:
      # Named volume
      - app-data:/app/data

      # Bind mount
      - ./local/path:/container/path

      # Read-only bind mount
      - ./config:/app/config:ro

      # Anonymous volume
      - /app/temp

volumes:
  app-data:
    driver: local
```

### Volume Configuration
```yaml
volumes:
  db-data:
    driver: local
    driver_opts:
      type: none
      device: /path/on/host
      o: bind

  cache-data:
    driver: local
    driver_opts:
      type: tmpfs
      device: tmpfs
      o: size=100m

  external-volume:
    external: true
    name: my-existing-volume
```

---

## 🏥 Health Checks

### Basic Health Check
```yaml
services:
  web:
    image: nginx
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### Health Check Variations
```yaml
services:
  # HTTP endpoint
  api:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 5s
      retries: 3
      start_period: 60s

  # Database
  postgres:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  # Custom script
  app:
    healthcheck:
      test: ["CMD", "node", "/app/healthcheck.js"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 120s

  # Disable health check
  cache:
    healthcheck:
      disable: true
```

---

## 🔗 Dependencies

### Basic Dependencies
```yaml
services:
  web:
    depends_on:
      - api
      - db

  api:
    depends_on:
      - db
      - redis

  db:
    image: postgres
```

### Conditional Dependencies (with healthchecks)
```yaml
services:
  web:
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started

  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

  redis:
    image: redis:alpine
```

---

## ⚙️ Resource Limits

### CPU & Memory
```yaml
services:
  app:
    image: myapp
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '0.5'
          memory: 1G
```

### Legacy Format (Compose v2)
```yaml
services:
  app:
    image: myapp
    mem_limit: 4g
    mem_reservation: 1g
    cpus: 2.0
```

---

## 🔄 Restart Policies

```yaml
services:
  # Always restart (except manual stop)
  web:
    restart: always

  # Restart on failure only
  api:
    restart: on-failure
    restart: on-failure:3  # Max 3 retries

  # Restart unless stopped manually
  cache:
    restart: unless-stopped

  # Never restart
  worker:
    restart: "no"
```

---

## 📋 Complete Examples

### Full Stack Web App
```yaml
version: '3.8'

services:
  # Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    image: myapp-frontend:latest
    container_name: myapp-frontend
    ports:
      - "80:3000"
    environment:
      - API_URL=http://api:4000
      - NODE_ENV=production
    depends_on:
      api:
        condition: service_healthy
    networks:
      - frontend
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  # Backend API
  api:
    build: ./api
    image: myapp-api:latest
    container_name: myapp-api
    expose:
      - "4000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    env_file:
      - .env.prod
    depends_on:
      db:
        condition: service_healthy
      cache:
        condition: service_started
    volumes:
      - api-uploads:/app/uploads
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:4000/health"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 60s

  # Database
  db:
    image: postgres:15-alpine
    container_name: myapp-db
    environment:
      - POSTGRES_DB=myapp
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - db-data:/var/lib/postgresql/data
      - ./db/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache
  cache:
    image: redis:7-alpine
    container_name: myapp-cache
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data
    networks:
      - backend
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
    internal: true

volumes:
  db-data:
    driver: local
  redis-data:
    driver: local
  api-uploads:
    driver: local
```

### Development Environment
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev

  db:
    image: postgres:15-alpine
    environment:
      - POSTGRES_DB=myapp_dev
      - POSTGRES_USER=dev
      - POSTGRES_PASSWORD=dev
    ports:
      - "5432:5432"
    volumes:
      - db-dev:/var/lib/postgresql/data

volumes:
  db-dev:
```

---

## 🎨 Advanced Patterns

### Override Files
```bash
# Base: docker-compose.yml
# Override: docker-compose.override.yml (auto-loaded)
# Production: docker-compose.prod.yml

# Development (uses override automatically)
docker-compose up -d

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Testing
docker-compose -f docker-compose.yml -f docker-compose.test.yml up -d
```

### Multi-File Configuration
```yaml
# docker-compose.yml (base)
services:
  web:
    image: nginx
    ports:
      - "80:80"

# docker-compose.prod.yml (production overrides)
services:
  web:
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
    restart: always
```

### Extension Fields (DRY)
```yaml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

x-healthcheck: &default-healthcheck
  interval: 10s
  timeout: 5s
  retries: 3

services:
  web:
    image: nginx
    logging: *default-logging
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "curl", "-f", "http://localhost"]

  api:
    image: myapi
    logging: *default-logging
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
```

---

## 🔍 Debugging

### Common Issues
```bash
# Check service logs
docker-compose logs -f web

# Check service status
docker-compose ps

# Validate compose file
docker-compose config

# Check specific service config
docker-compose config --service web

# View port mappings
docker-compose port web 80

# List containers
docker-compose ps -a

# Debug networking
docker-compose exec web ping api

# Check environment variables
docker-compose exec web env

# Rebuild and restart
docker-compose up -d --build --force-recreate
```

---

## 📚 Best Practices

1. **Version Control**: Don't commit `.env` files with secrets
2. **Environment**: Use `.env` files for configuration
3. **Networks**: Separate frontend/backend networks
4. **Volumes**: Use named volumes for persistence
5. **Health Checks**: Add healthchecks for all services
6. **Dependencies**: Use `depends_on` with conditions
7. **Resource Limits**: Set limits for production
8. **Restart Policies**: Use `unless-stopped` for services
9. **Build Cache**: Use BuildKit for faster builds
10. **Logging**: Configure log rotation

---

## 🚨 Common Pitfalls

### Port Already in Use
```bash
# Check what's using the port
sudo lsof -i :80

# Use different port
ports:
  - "8080:80"
```

### Volume Permissions
```bash
# Fix permissions
docker-compose exec web chown -R www-data:www-data /app/uploads

# Or in Dockerfile
USER www-data
```

### Database Not Ready
```yaml
# Use healthcheck + depends_on condition
services:
  app:
    depends_on:
      db:
        condition: service_healthy
  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
```

---

## 🎯 Quick Reference

### Environment Variable Precedence
1. Compose file `environment`
2. Shell environment variables
3. `.env` file
4. Dockerfile `ENV`

### Compose File Versions
- `3.8`: Latest Compose v3 (recommended)
- `3.0-3.7`: Older v3 versions
- `2.x`: Legacy (don't use)

### Useful Environment Variables
```bash
COMPOSE_PROJECT_NAME=myapp    # Project name
COMPOSE_FILE=compose.yml      # Compose file path
COMPOSE_PATH_SEPARATOR=:      # Path separator (: or ;)
```

---

**Last updated**: 2025-12-23
**Source**: Docker Compose best practices & production usage
