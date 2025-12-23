# Docker Images - Layers & Optimization

## 📋 Metadata

```yaml
tags: [concept, docker, images, layers, optimization, status/mastered]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 4h
```

**Prerequisites**: [[docker-basics]]
**Related to**: [[dockerfile-best-practices]], [[multi-stage-builds]]

---

## 🎯 TL;DR (30 seconds)

Docker images are built from layers - each Dockerfile instruction creates a new read-only layer. Layers are cached and shared between images. Understanding layers is key to building fast, efficient images: order instructions by change frequency, combine commands to reduce layers, and use multi-stage builds to exclude build dependencies from final image.

**Analogy**: Like a layer cake where each layer is a filesystem snapshot. You can't change a baked layer, but you can add new layers on top. Multiple cakes can share bottom layers to save space.

---

## 🤔 When to Use?

### ✅ Good for
1. **Immutable deployments**: Same image ID = identical behavior everywhere
2. **Fast rebuilds**: Layer caching speeds up repeated builds
3. **Efficient storage**: Shared layers save disk space

### ❌ Bad for
- **Large build contexts** → Use .dockerignore
- **Secrets in images** → Use build secrets or runtime injection
- **Development hot-reload** → Use volumes instead

---

## 📚 Key Concepts (In My Own Words)

### 1. Image Layers Architecture

**My understanding**:
Each Dockerfile instruction (FROM, RUN, COPY, etc.) creates a new layer. Layers are stacked using Union Filesystem (OverlayFS). When you run a container, Docker adds a thin writable layer on top of read-only image layers.

**Why important**:
Understanding layers explains why `RUN apt-get update && apt-get install` is better than separate RUN commands - fewer layers = smaller image.

**Mental model**:
```
[Container Layer - Writable]  ← Runtime changes (logs, temp files)
------------------------
[Layer 4: CMD]               ← Read-only
[Layer 3: COPY app/]         ← Read-only
[Layer 2: RUN npm install]   ← Read-only
[Layer 1: FROM node:18]      ← Read-only (base image)
```

Each layer contains only the diff from previous layer.

---

### 2. Layer Caching

**My understanding**:
Docker caches each layer. When rebuilding, if instruction + context hasn't changed, Docker reuses cached layer. Cache breaks when instruction changes OR any previous layer changed.

**Why important**:
Proper layer ordering = fast rebuilds. Put frequently changing instructions (COPY source code) AFTER stable instructions (RUN apt-get install).

**Cache invalidation example**:
```dockerfile
# ❌ Bad - cache breaks on every code change
FROM node:18
COPY . /app              # Changes often → cache broken
RUN npm install          # Rebuilds every time!

# ✅ Good - cache preserved
FROM node:18
COPY package*.json /app/ # Changes rarely
RUN npm install          # Cached until package.json changes!
COPY . /app              # Code changes don't affect npm install
```

---

### 3. Multi-Stage Builds

**My understanding**:
Multiple FROM statements in single Dockerfile. Each FROM starts a new build stage. Final image only contains last stage, but you can COPY artifacts from previous stages.

**Why important**:
Build stage can have compilers, build tools (large). Production stage only has runtime dependencies (small). Massive size reduction.

**Real example from production**:
```dockerfile
# Stage 1: Build (large - 800MB)
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production (small - 150MB)
FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
USER node
CMD ["node", "dist/server.js"]
```

Final image: 150MB (not 800MB!)

---

### 4. Image Manifest & Digests

**My understanding**:
- **Tag**: Human-friendly name (myapp:v1.0.0, myapp:latest)
- **Digest**: SHA256 hash of image content (immutable)
- **Manifest**: JSON describing image layers, architecture, OS

**Why important**:
Tags can be overwritten (latest changes). Digests are immutable. For reproducible deployments, use digests.

**Example**:
```bash
# Tag can change
docker pull nginx:latest

# Digest is immutable
docker pull nginx@sha256:abc123...

# Get digest from tag
docker inspect nginx:latest --format='{{.RepoDigests}}'
```

---

## 💻 Minimal Example

### Context
Optimizing a Node.js application image - reducing size from 1.2GB to 180MB and build time from 5min to 30s (on cache hit).

### Code

**Bad Dockerfile (1.2GB, slow rebuilds)**:
```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
RUN npm test
EXPOSE 3000
CMD ["npm", "start"]
```

**Optimized Dockerfile (180MB, fast rebuilds)**:
```dockerfile
# ======================
# Stage 1: Dependencies
# ======================
FROM node:18-alpine AS deps
WORKDIR /app

# Copy package files (changes rarely)
COPY package*.json ./

# Install ALL dependencies (dev + prod)
RUN npm ci

# ======================
# Stage 2: Build
# ======================
FROM node:18-alpine AS builder
WORKDIR /app

# Copy dependencies from previous stage
COPY --from=deps /app/node_modules ./node_modules

# Copy source code (changes often)
COPY . .

# Build application
RUN npm run build

# Run tests (fail build if tests fail)
RUN npm test

# ======================
# Stage 3: Production
# ======================
FROM node:18-alpine AS production
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install ONLY production dependencies
RUN npm ci --only=production && \
    npm cache clean --force

# Copy built application from builder stage
COPY --from=builder /app/dist ./dist

# Security: run as non-root
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "dist/server.js"]
```

**.dockerignore** (exclude from build context):
```
node_modules
npm-debug.log
.git
.gitignore
.env
.env.*
dist
coverage
.vscode
.idea
*.md
Dockerfile*
docker-compose*.yml
```

### Line-by-Line Explanation
```
FROM node:18-alpine AS deps
  → Use Alpine (150MB vs 1GB for full node:18)
  → Name this stage "deps" for reference

COPY package*.json ./
  → Copy ONLY package files first
  → If package.json unchanged, next layer cached

RUN npm ci
  → Clean install (faster, reproducible)
  → This layer cached until package.json changes

COPY --from=builder /app/dist ./dist
  → Copy built artifacts from previous stage
  → Builder stage NOT included in final image

RUN npm ci --only=production && npm cache clean --force
  → Install ONLY runtime dependencies
  → Clean npm cache to reduce size
  → Combine in single RUN to reduce layers

USER nodejs
  → Security: run as non-root user (UID 1001)
  → Prevents privilege escalation attacks
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Layer Cache Invalidation

**Symptom**:
```
Every code change rebuilds npm install (5 minutes)
Build cache seems broken
```

**What I did wrong**:
```dockerfile
# ❌ Wrong - COPY . before npm install
FROM node:18
WORKDIR /app
COPY . .                 # Code changes frequently
RUN npm install          # Cache broken on every code change!
```

**Why wrong**: COPY . copies ALL files including source code. Any code change invalidates this layer and ALL subsequent layers (including npm install).

**Solution**:
```dockerfile
# ✅ Correct - Dependencies before code
FROM node:18
WORKDIR /app
COPY package*.json ./    # Changes rarely
RUN npm install          # Cached until package.json changes
COPY . .                 # Code copied last
```

**Time wasted**: 2h debugging + weeks of slow builds
**Lesson**: Order Dockerfile instructions by change frequency: stable first, volatile last.

---

### Pitfall 2: Large Image Size (node_modules in production)

**Symptom**:
```
Image size: 1.2GB
Most of size is devDependencies (webpack, typescript, jest)
Deployment slow, storage expensive
```

**What I did wrong**:
```dockerfile
# ❌ Wrong - All dependencies in final image
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install          # Installs dev + prod dependencies
COPY . .
RUN npm run build
CMD ["node", "dist/server.js"]
```

**Why wrong**: npm install includes devDependencies (build tools). Final image contains webpack, typescript, jest - not needed at runtime.

**Solution**:
```dockerfile
# ✅ Correct - Multi-stage build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci              # All deps for build
COPY . .
RUN npm run build

FROM node:18-alpine AS production
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production  # Only runtime deps
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/server.js"]
```

**Before**: 1.2GB | **After**: 180MB (85% reduction)

**Time wasted**: 3h initial implementation
**Lesson**: Multi-stage builds are essential for compiled languages. Build stage = all tools. Production stage = runtime only.

---

### Pitfall 3: Secrets in Image Layers

**Symptom**:
```
API key visible in docker history
Secret leaked when image pushed to registry
Security audit failed
```

**What I did wrong**:
```dockerfile
# ❌ NEVER DO THIS - Secret in layer
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm config set //registry.npmjs.org/:_authToken=${NPM_TOKEN}
RUN npm install
# Even if you delete token after, it's still in layer history!
```

**Why wrong**: Each RUN creates a layer. Layer content is permanent. Even if you delete the secret in next layer, it's still in previous layer (visible in docker history).

**Solution**:
```dockerfile
# ✅ Option 1: BuildKit secrets (recommended)
# syntax=docker/dockerfile:1
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN --mount=type=secret,id=npm_token \
    npm config set //registry.npmjs.org/:_authToken=$(cat /run/secrets/npm_token) && \
    npm install

# Build with:
# docker build --secret id=npm_token,src=.npmrc .

# ✅ Option 2: Multi-stage with ARG (less secure)
FROM node:18 AS builder
ARG NPM_TOKEN
RUN npm config set //registry.npmjs.org/:_authToken=${NPM_TOKEN}
RUN npm install
# (Don't use in production stage)

# ✅ Option 3: Runtime injection
# Don't include secret at build time
# Inject via environment at runtime
```

**Time wasted**: 4h security incident response
**Lesson**: NEVER put secrets in Dockerfile or image layers. Use BuildKit secrets, build args (with care), or runtime injection.

---

### Pitfall 4: Huge Build Context

**Symptom**:
```
"Sending build context to Docker daemon: 2.5GB"
Build hangs for 5 minutes before even starting
```

**What I did wrong**:
```
# ❌ No .dockerignore file
# Docker sends ENTIRE directory to daemon
# Including node_modules/, .git/, dist/, etc.
```

**Why wrong**: Docker sends entire build context (directory) to daemon. Without .dockerignore, sends node_modules (500MB), .git (200MB), old builds, etc.

**Solution**:
```
# ✅ .dockerignore
node_modules
.git
.gitignore
dist
coverage
*.log
.env*
.vscode
.idea
README.md
docker-compose*.yml
Dockerfile*
.dockerignore
```

**Before**: 2.5GB context, 5min send time
**After**: 50MB context, 5sec send time

**Time wasted**: 1h debugging
**Lesson**: ALWAYS create .dockerignore. Exclude node_modules, .git, build artifacts, IDE files.

---

## 🔧 Essential Commands

```bash
# Build image
docker build -t myapp:latest .

# Build with specific Dockerfile
docker build -f Dockerfile.prod -t myapp:prod .

# Build with build args
docker build --build-arg NODE_VERSION=18 -t myapp .

# Build without cache
docker build --no-cache -t myapp .

# View image layers
docker history myapp:latest

# View layer sizes
docker history --no-trunc --format "table {{.Size}}\t{{.CreatedBy}}" myapp:latest

# Inspect image
docker inspect myapp:latest

# Get image digest
docker inspect myapp:latest --format='{{.RepoDigests}}'

# View image size
docker images myapp

# Save image to tar
docker save myapp:latest -o myapp.tar

# Load image from tar
docker load -i myapp.tar

# Export container filesystem (loses layers)
docker export <container> -o container.tar

# Analyze image layers (with dive)
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest myapp:latest

# Build with BuildKit (faster, better caching)
DOCKER_BUILDKIT=1 docker build -t myapp .

# Build multi-platform image
docker buildx build --platform linux/amd64,linux/arm64 -t myapp .
```

---

## 🧪 Tests Done

### Understanding
- [x] Explained layers to colleague (3 min)
- [x] Optimized 3 production Dockerfiles
- [x] Compared single-stage vs multi-stage builds

### Practice
- [x] **Lab 1**: Node.js app optimization (1.2GB → 180MB) - ✅ Success (2h)
- [x] **Lab 2**: Multi-stage build for Go app - ✅ Success (1h)
- [x] **Lab 3**: Layer caching optimization - ✅ Success (1h30)

### Retention (Day+7)
- Date: TBD
- Result: TBD
- Score: TBD

---

## ❓ Unresolved Questions

1. **BuildKit cache mounts** - When to use vs layer caching? - To investigate
2. **Image signing & verification** - Best practices for supply chain security - Blocking: No

---

## 📊 Learning Timeline

```
2024-XX: Discovered multi-stage builds (reduced image 85%)
2024-XX: Learned layer caching (30x faster rebuilds)
2025-12: Mastered BuildKit secrets
2025-12-23: Documentation extraction
```

### Time Invested

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|-------|
| Discovery | 1h | 1h | 2h |
| Practice (optimization) | 30min | 3h | 3h30 |
| Debugging (4 pitfalls) | - | 3h | 3h |
| Documentation | 30min | 1h | 1h30 |
| **TOTAL** | **2h (20%)** | **8h (80%)** | **10h** |

**Ratio**: 20% ✅ (target <30%)

---

## 📝 Status

**Current status**: ✅ Mastered

---

## 🔗 Resources

### Official
- [Docker Image Layers](https://docs.docker.com/storage/storagedriver/)
- [Multi-stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [BuildKit](https://docs.docker.com/build/buildkit/)

### Tools
- [Dive](https://github.com/wagoodman/dive) - Analyze image layers
- [Docker Slim](https://github.com/docker-slim/docker-slim) - Minify images

### Personal
- [[cheatsheet-docker]]
- [[dockerfile-best-practices]]
- Real projects: All production Dockerfiles

---

## ✅ Mastery Checklist

### Understanding
- [x] Explain 3 min without notes
- [x] 3 concrete use cases
- [x] 4 common pitfalls (cache, size, secrets, context)
- [x] Explain layer caching mechanism

### Application
- [x] Multi-stage build implementation
- [x] Image size optimization (>50% reduction)
- [x] Fast rebuild configuration
- [x] Debug layer issues

### Solidification
- [x] Complete note with real examples
- [x] Extract to cheatsheet
- [x] Production implementations
- [ ] Retention test Day+7: TBD

**Validation date**: 2025-12-23
**Total time**: 10h

---

**Last update**: 2025-12-23
**Next review**: 2026-01-23 (+1 month)
