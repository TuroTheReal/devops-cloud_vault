# Glasck - Production Docker Deployment

## 📋 Metadata

```yaml
tags: [project, docker, docker-compose, traefik, production, automation, 2025-12]
started: 2025-12-01
completed: 2025-12-20
duration: ~40h total (15h infrastructure, 10h debugging, 10h automation, 5h optimization)
real-repo: https://github.com/Glasck-int (private)
status: production
```

**Technologies used**: Docker Compose (with Swarm evolution path), Traefik v3, Redis, Taskfile, Git automation
**Goal**: Deploy scalable web application with zero-downtime updates, automated workflows, and production-grade infrastructure

---

## 🎯 Project Context

### Objective
Build production-ready deployment infrastructure for Glasck web application with:
- **Zero-downtime deployments**: Rolling updates with automatic rollback
- **Network isolation**: Public-facing Traefik, private backend services
- **Automation**: Taskfile-based workflows for deployment, rollback, monitoring
- **Scalability**: Docker Compose orchestration (Swarm-ready architecture for future scaling)

### Architecture

```
┌────────────────────────────────────────────────────┐
│ Internet (HTTPS)                                   │
└──────────────────┬─────────────────────────────────┘
                   │
           ┌───────▼────────┐
           │   Traefik v3   │ (Reverse Proxy + Load Balancer)
           │  Port 80/443   │ - Dynamic routing via labels
           │  Dashboard:8082│ - Auto SSL (Let's Encrypt ready)
           └───────┬────────┘ - Rate limiting, middlewares
                   │
         ┌─────────┴──────────┐
         │                    │
┌────────▼────────┐  ┌────────▼──────────┐
│   Frontend      │  │   API Backend     │
│   (Next.js)     │  │   (Fastify)       │
│   Port 3000     │  │   Port 3001       │
│   - SSR/SSG     │  │   - REST API      │
│   - Public +    │  │   - Public +      │
│     Private nets│  │     Private nets  │
└─────────────────┘  └────────┬──────────┘
                              │
                     ┌────────▼────────┐
                     │   Redis Cache   │
                     │   Port 6379     │
                     │   - Private net │
                     │   - Persistence │
                     └─────────────────┘

Networks:
• public-facing (bridge): Traefik + Frontend + API
• internal-backend (bridge): Frontend + API + Redis
```

### Tech Stack
- **Orchestration**: Docker Compose (designed for easy Swarm migration)
- **Reverse Proxy**: Traefik v3 with Docker provider
- **Cache/Session**: Redis 7 Alpine with persistence
- **Automation**: Taskfile (YAML-based task runner)
- **Deployment**: Multi-stage Dockerfiles, versioned images
- **Networks**: Bridge networks with MTU 1450, zero-trust isolation

### Evolution Path to Production Swarm

This project uses Docker Compose for development/single-node deployment with an architecture designed for seamless Swarm migration:
- **Current**: Docker Compose with bridge networks, manual scaling
- **Future**: Docker Swarm with overlay networks, automatic scaling, multi-node HA
- **Migration**: Stack files already compatible, just change driver from bridge to overlay

---

## 📚 What I Learned

### New Concepts Mastered

1. **Docker Swarm Overlay Networks** [[concepts/docker/docker-swarm-overlay-networks]]
   - Time to learn: 6h (4h practice + 2h debugging)
   - Difficulty: ⭐⭐⭐⭐ (4/5)
   - **Key insight**: MTU 1450 prevents VXLAN fragmentation, `attachable: true` allows debug containers

2. **Docker Swarm Deployment Strategies** [[concepts/docker/docker-swarm-deployment-strategies]]
   - Time to learn: 4h (2h implementation + 2h testing rollback)
   - Difficulty: ⭐⭐⭐ (3/5)
   - **Key insight**: `monitor` period must exceed `start_period` for automatic rollback

3. **Traefik Swarm Integration** [[concepts/traefik/traefik-integration]]
   - Time to learn: 8h (5h configuration + 3h debugging routing)
   - Difficulty: ⭐⭐⭐⭐ (4/5)
   - **Key insight**: `traefik.docker.network` label CRITICAL for multi-network services

4. **Docker Network Isolation** [[concepts/docker/docker-network-isolation]]
   - Time to learn: 3h (1h design + 2h implementation)
   - Difficulty: ⭐⭐⭐ (3/5)
   - **Key insight**: Zero-trust architecture - Redis never on public network

5. **Taskfile Automation** [[cheatsheets/taskfile/taskfile-commands]]
   - Time to learn: 4h (2h learning + 2h building workflows)
   - Difficulty: ⭐⭐ (2/5)
   - **Key insight**: YAML > Makefile for readability, cross-platform support

### Skills Applied Successfully
- **Infrastructure as Code**: All configuration in Git (stack files, Traefik config)
- **GitOps workflow**: Automated repo cloning, version management
- **Zero-downtime deployments**: Tested rollouts with `start-first`, verified no dropped connections
- **Monitoring integration**: Healthchecks at Docker + Traefik levels
- **Automation**: 15+ Taskfile commands for complete deployment lifecycle

---

## ⚠️ Challenges & Solutions

### Challenge 1: MTU Fragmentation Causing Connection Hangs

**Problem**: Large HTTP requests (file uploads, big JSON payloads) hung indefinitely

**Context**: After deploying Swarm overlay networks, small requests worked but large transfers failed

**Impact**:
- Time lost: 3h debugging
- Severity: High (production blocker)

**Root Cause**: Default MTU 1500 doesn't account for VXLAN overhead (50 bytes). Packets >1450 bytes fragmented, often dropped.

**Solution**:
```yaml
networks:
  public-facing:
    driver: overlay
    driver_opts:
      com.docker.network.driver.mtu: "1450"  # 1500 - 50 (VXLAN overhead)
```

**Lesson Learned**: Always set MTU to 1450 for Swarm overlay networks. Test with large payloads before production.

**Knowledge Base Update**:
→ Added to: [[concepts/docker/docker-swarm-overlay-networks]]

---

### Challenge 2: Traefik 502 Errors (Multi-Network Routing)

**Problem**: Traefik discovered frontend service but returned 502 Bad Gateway

**Context**: Frontend on TWO networks (public-facing for Traefik, internal-backend for API calls)

**Impact**:
- Time lost: 2h debugging
- Severity: Critical (app inaccessible)

**Root Cause**: Traefik didn't know which network to use for backend communication. Tried `internal-backend` where Traefik itself wasn't connected.

**Solution**:
```yaml
services:
  front-website-dev:
    networks:
      - public-facing
      - internal-backend
    deploy:
      labels:
        - "traefik.docker.network=glasck_public-facing"  # Explicit!
```

**Lesson Learned**: When service connects to multiple networks, ALWAYS specify `traefik.docker.network` label.

**Knowledge Base Update**:
→ Added to: [[concepts/traefik/traefik-integration]]

---

### Challenge 3: Automatic Rollback Not Working

**Problem**: Deployed broken version, expected automatic rollback, didn't happen

**Context**: Set `failure_action: rollback` but new containers marked healthy despite crashing

**Impact**:
- Time lost: 2h debugging + 30min fixing deployment
- Severity: High (broken version in production)

**Root Cause**: `monitor: 45s` but `start_period: 120s`. Monitor window ended before healthcheck failures counted.

**Solution**:
```yaml
healthcheck:
  start_period: 120s  # Grace period

deploy:
  update_config:
    monitor: 180s  # MUST be > start_period (120s + buffer)
    failure_action: rollback
```

**Lesson Learned**: Monitor period must exceed start period. Otherwise monitoring ends while container still in "starting" state.

**Knowledge Base Update**:
→ Added to: [[concepts/docker/docker-swarm-deployment-strategies]]

---

### Challenge 4: Git Repository Automation Complexity

**Problem**: Manually cloning/updating multiple repositories was error-prone

**Context**: Glasck has frontend + API in separate repos, need coordinated deployment

**Impact**:
- Time lost: 1h initial design + 2h implementation
- Severity: Medium (developer experience)

**Root Cause**: No automated workflow for multi-repo management

**Solution - Taskfile automation**:
```yaml
vars:
  SERVICES: |
    front-website:git@github.com:Glasck-int/front.git
    api-fastify:git@github.com:Glasck-int/api.git

tasks:
  setup-l:
    desc: Clone/update all repos
    cmds:
      - |
        while IFS= read -r line; do
          service=$(echo "$line" | cut -d':' -f1)
          repo=$(echo "$line" | cut -d':' -f2-)

          if [ ! -d "service/$service/src/.git" ]; then
            git clone "$repo" "service/$service/src"
          else
            cd "service/$service/src" && git pull && cd -
          fi
        done <<< "{{.SERVICES}}"
```

**Lesson Learned**: Automate repetitive multi-repo operations. Taskfile better than bash scripts for readability.

**Knowledge Base Update**:
→ Added to: [[cheatsheets/taskfile/taskfile-commands]]

---

## 🔧 Reusable Assets Created

### Scripts Extracted to devops-cloud_tools
None yet - Taskfile commands stay in project (project-specific)

### Configs Extracted
None - Configuration too project-specific (Traefik labels reference exact domain names)

### Patterns Documented

1. **Stack Separation Pattern**:
   - `stack.vps.yml`: Infrastructure (Traefik, Redis, networks)
   - `stack.dev.yml`: Application services (Frontend, API)
   - Deploy infra first, then app → clean separation

2. **Multi-Stage Build Pattern**:
   - Stage 1: Build (all dependencies + dev tools)
   - Stage 2: Production (runtime dependencies only)
   - Result: 85% size reduction (typical)

3. **Version Management**:
   - `VERSION` file per service
   - Taskfile reads version, tags image
   - Environment variable for stack deployment
   - Easy rollback to specific version

4. **Healthcheck Hierarchy**:
   - Docker healthcheck (service-level)
   - Traefik healthcheck (load-balancer level)
   - Both with appropriate intervals/timeouts

### Useful Commands & One-Liners Discovered

```bash
# Deploy Swarm stack with environment variables
export API_VERSION=$(cat service/api/VERSION)
export FRONT_VERSION=$(cat service/frontend/VERSION)
docker stack deploy -c stack.dev.yml glasck

# Watch service rollout
watch -n 1 'docker service ps glasck_front-website'

# Check Traefik routing
curl -H "Host: glasck.com" http://localhost

# View service update status
docker service inspect glasck_api --format '{{.UpdateStatus}}'

# Quick healthcheck test
docker service ps glasck_front-website --filter "desired-state=running"
```

**Added to Cheatsheet**: [[cheatsheets/docker/docker-swarm-commands]]

---

## 📊 Time Breakdown

### Detailed Breakdown

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|----------|
| Architecture Design | 1h | 2h | 3h |
| Docker Swarm Setup | 30min | 3h | 3h30 |
| Traefik Configuration | 2h | 5h | 7h |
| Network Implementation | - | 2h | 2h |
| Debugging (MTU, Routing, Rollback) | 1h | 6h | 7h |
| Taskfile Automation | 30min | 3h30 | 4h |
| Testing & Validation | - | 4h | 4h |
| Documentation (this file) | 1h | 1h | 2h |
| Optimization | - | 3h | 3h |
| **TOTAL** | **6h (16%)** | **29h30 (84%)** | **35h30** |

**Assisted/Autonomous Ratio**: 16% ✅ (target <30%)

### Time Distribution by Technology

| Technology | Time Spent | Notes |
|------------|-----------|-------|
| Docker Swarm | 12h | Overlay networks, deployment strategies, healthchecks |
| Traefik | 10h | Configuration, middleware, multi-network routing |
| Taskfile | 4h | Learning + building 15+ automated workflows |
| Redis | 1h | Configuration, persistence, healthcheck |
| Git Automation | 2h | Multi-repo cloning/updating |
| Debugging | 7h | MTU issue (3h), routing (2h), rollback (2h) |

---

## 📝 Extractions TODO

### Already Completed ✅
- [x] Extract [[concepts/docker/docker-swarm-overlay-networks]] (6h) - MTU, attachable networks
- [x] Extract [[concepts/docker/docker-swarm-deployment-strategies]] (4h) - Rolling updates, automatic rollback
- [x] Extract [[concepts/traefik/traefik-integration]] (8h) - Dynamic routing, multi-network labels
- [x] Extract [[concepts/docker/docker-network-isolation]] (3h) - Zero-trust architecture
- [x] Update [[cheatsheets/docker/docker-swarm-commands]] with stack management (included in concept)
- [x] Update [[cheatsheets/traefik/traefik-commands]] with label patterns (included in concept)
- [x] Create [[cheatsheets/taskfile/taskfile-commands]] with Glasck automation (4h)

**Extraction completed**: 25h (concepts + cheatsheets)

---

## 🎯 Outcomes & Results

### Success Metrics
- ✅ **Zero-downtime deployments**: Verified with load testing, no dropped connections
- ✅ **Network isolation**: Redis inaccessible from internet, verified with security scan
- ✅ **Automated workflows**: 15 Taskfile commands covering full lifecycle
- ✅ **Production ready**: Deployed to VPS, stable for 1+ week
- ✅ **Rollback capability**: Tested automatic + manual rollback, both work
- ✅ **Monitoring**: Healthchecks + logs, easy debugging

### What Worked Well
1. **Stack separation**: Infrastructure vs application stacks = clean deployment flow
2. **Taskfile automation**: YAML-based tasks easier to maintain than bash scripts
3. **Version management**: VERSION files + automated tagging = easy rollback
4. **Incremental deployment**: Deploy infrastructure → wait → deploy app
5. **Documentation as code**: All configuration in Git, easy to replicate

### What Could Be Improved
1. **Monitoring**: Add Prometheus + Grafana for metrics (logs only currently)
2. **Secrets management**: Using environment files, should use Docker secrets
3. **Multi-node**: Tested single-node, multi-node needs validation
4. **CI/CD**: Manual deployment, should integrate with GitHub Actions
5. **Backup automation**: Manual backups, should automate with cron

### Performance/Cost Metrics
- **Deployment time**: ~2min (infrastructure + application)
- **Rollback time**: <30s (automatic), <1min (manual to specific version)
- **Image size**: 180MB frontend, 150MB API (after multi-stage optimization)
- **Memory usage**: ~2GB total (Frontend 1GB, API 500MB, Redis 200MB, Traefik 300MB)
- **Monthly cost**: VPS ~$10/month (single node)
- **Uptime**: 99.9%+ (tested 1 week, zero crashes)

---

## 💡 Key Takeaways

### Technical Insights
1. **Overlay networks need MTU 1450**: VXLAN adds 50 bytes overhead, 1500 causes fragmentation
2. **Multi-network services need explicit routing**: `traefik.docker.network` label is mandatory
3. **Monitor > start_period**: For automatic rollback to work properly
4. **Network isolation is defense-in-depth**: Even if firewall fails, DB isolated
5. **Taskfile > Makefile**: Better readability, built-in cross-platform support

### Process Improvements
1. **Test rollback before production**: Intentionally deploy broken version, verify rollback works
2. **Incremental deployment**: Infra first (networks, Traefik, Redis) → then app
3. **Version everything**: Services, images, even backups
4. **Automate from start**: Don't accumulate manual tasks, automate incrementally
5. **Document pitfalls immediately**: Write down solution while debugging, not after

### Personal Growth
- **Confidence before**: Docker Swarm: ⭐⭐ (2/5), Traefik: ⭐ (1/5)
- **Confidence after**: Docker Swarm: ⭐⭐⭐⭐ (4/5), Traefik: ⭐⭐⭐⭐ (4/5)
- **New skills acquired**: Swarm orchestration, Traefik dynamic routing, Taskfile automation
- **Skills improved**: Network architecture, zero-downtime deployments, debugging

---

## 📝 Post-Project Checklist

### Immediate Actions
- [x] Project deployed and stable in production
- [x] Documentation written (this file)
- [x] Knowledge extracted to concepts/cheatsheets
- [x] Pitfalls documented
- [ ] Backups automated (manual currently)
- [ ] Monitoring enhanced (add metrics)
- [ ] CI/CD pipeline (manual deployment currently)

### Follow-up (Week 1)
- [ ] Retention test: Explain Swarm networking without notes (Day +7)
- [ ] Review Traefik configuration, optimize middlewares
- [ ] Test multi-node deployment on local cluster

### Follow-up (Month 1)
- [ ] Retention test: Deep recall of all components (Day +30)
- [ ] Implement Prometheus + Grafana
- [ ] Add automated backups to Taskfile
- [ ] Document infrastructure costs and optimize

---

## 🔗 Links & Resources

### Project Repository
- **Main repo**: GitHub Glasck-int (private)
- **Infrastructure code**: [deployment/](https://github.com/Glasck-int/deployment)
- **Source files**:
  - [stack.vps.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.vps.yml)
  - [stack.dev.yml](/home/artberna/abGitHub/Glasck/deployment/docker/stack.dev.yml)
  - [Taskfile.yml](/home/artberna/abGitHub/Glasck/deployment/Taskfile.yml)
  - [traefik.yml](/home/artberna/abGitHub/Glasck/deployment/config/traefik/traefik.yml)

### Related Projects
- [[projects/2024-transcendence-glasck-extraction]] - Knowledge extraction session
- Future: Multi-node cluster deployment
- Future: CI/CD integration

### External Resources Used
- [Traefik Swarm Provider Docs](https://doc.traefik.io/traefik/providers/swarm/)
- [Docker Swarm Networking](https://docs.docker.com/engine/swarm/networking/)
- [Taskfile Documentation](https://taskfile.dev/)

---

## 📅 Timeline

```
2025-12-01 : Project kickoff, architecture design
2025-12-03 : Docker Swarm setup, first deployment test
2025-12-05 : MTU fragmentation discovered and fixed (3h)
2025-12-08 : Traefik integration, routing working
2025-12-10 : Multi-network routing issue fixed (2h)
2025-12-12 : Taskfile automation implemented
2025-12-15 : Rollback testing, discovered monitor/start_period issue (2h)
2025-12-18 : Production deployment to VPS
2025-12-20 : Testing and validation complete
2025-12-23 : Post-mortem and knowledge extraction
```

---

## 🎓 Retention Checks

### Day +7 (2025-12-30)
- [ ] Explain overlay network MTU without notes
- [ ] Configure Traefik multi-network routing from scratch
- [ ] Write deployment strategy config from memory
- **Score**: TBD
- **Status**: TBD

### Day +30 (2026-01-23)
- [ ] Design similar architecture for new project
- [ ] Debug Swarm networking issues quickly
- [ ] Build Taskfile automation for different project
- **Score**: TBD
- **Status**: TBD

---

**Project completed**: 2025-12-20
**Last updated**: 2025-12-23
**Next similar project**: Multi-node cluster deployment (Q1 2026)
**Production status**: Deployed, stable, monitored

---

## 📊 Technology Scorecard

| Technology | Before | After | Confidence Gain |
|------------|--------|-------|-----------------|
| Docker Swarm | ⭐⭐ | ⭐⭐⭐⭐ | +2 |
| Traefik | ⭐ | ⭐⭐⭐⭐ | +3 |
| Overlay Networks | ⭐ | ⭐⭐⭐⭐ | +3 |
| Taskfile | ⭐ | ⭐⭐⭐ | +2 |
| Network Security | ⭐⭐ | ⭐⭐⭐⭐ | +2 |
| Zero-Downtime Deploy | ⭐ | ⭐⭐⭐⭐ | +3 |

**Average Growth**: +2.5 stars across 6 technologies
