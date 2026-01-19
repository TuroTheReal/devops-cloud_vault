# Docker Restart - Config Not Applied (Network Missing)

## 📋 Metadata

```yaml
tags: [troubleshooting, docker, docker-compose, network, prometheus, monitoring, status/documented]
created: 2026-01-19
updated: 2026-01-19
severity: 🟠 High
time-to-fix: 1h
```

**Related concept**: [[concepts/docker/docker-compose-basics]]
**Affected versions**: Docker 20.x+, Docker Compose v2+

---

## 🔍 Symptom

> **Alerting loop on Grafana/Prometheus: service seen as "down" while it's actually UP and running fine.**

```bash
# Container UP for a long time but no "(healthy)" status
docker ps
CONTAINER_ID   IMAGE          STATUS
abc123         traefik:v3.0   Up 30 hours    # No "(healthy)"

# Prometheus can't reach the container
docker exec prometheus wget -qO- http://reverse-proxy:8080/metrics
wget: bad address 'reverse-proxy:8080'

# Container is NOT in the expected network
docker inspect reverse-proxy --format='{{json .NetworkSettings.Networks}}' | jq 'keys'
["glasck-public-facing"]  # Missing "glasck-monitoring"!
```

**Common scenarios where this occurs**:
- After VPS reboot (`restart: unless-stopped` restarts with old config)
- After `docker restart <container>`
- docker-compose.yml modified without running `up -d`

---

## 🎯 Root Cause

> Docker **freezes the config at container creation time**. A restart does NOT re-read docker-compose.yml.

**Cause chain**:

```
Container created with old config (networks: [public-facing] only)
    ↓
docker-compose.yml modified (added networks: [monitoring])
    ↓
VPS reboot → Docker restarts the container
    ↓
Restart = stop + start with FROZEN config (old one)
    ↓
Container still missing "monitoring" network
    ↓
Prometheus (on monitoring network) can't reach the container
    ↓
Target "down" → Alert loop
```

| Action | Behavior |
|--------|----------|
| `docker-compose up -d` | Recreates container with **current** config |
| `docker restart` | Stop + Start with **frozen** config (old) |
| VPS reboot | Same as restart |

---

## ✅ Quick Fix

### Solution 1: Recreate containers (Recommended)

```bash
# Recreate containers with current config
docker-compose -f docker-compose.yml up -d

# Docker-compose auto-detects changes and recreates only modified containers

# Verify network is now attached
docker inspect <container> --format='{{json .NetworkSettings.Networks}}' | jq 'keys'
# Should show: ["glasck-public-facing", "glasck-monitoring"]

# Verify Prometheus can now scrape
docker exec prometheus wget -qO- http://<container>:<port>/metrics | head -5
```

**Explanation**: `up -d` compares current compose config vs container config and recreates if different.

---

### Solution 2: Manual network connect (Temporary)

```bash
# Manually connect to missing network
docker network connect glasck-monitoring <container>

# ⚠️ Will be lost on next restart/recreate!
```

**When to use**: Quick fix without recreating container, but NOT permanent.

---

## 🔬 Diagnosis Steps

**If quick fix doesn't work, debug with**:

```bash
# 1. List container networks
docker inspect <container> --format='{{json .NetworkSettings.Networks}}' | jq 'keys'

# 2. Compare with docker-compose.yml
grep -A5 "networks:" docker-compose.yml

# 3. Check container creation date vs compose modification
docker inspect <container> --format='{{.Created}}'
stat docker-compose.yml

# 4. Test connectivity from Prometheus
docker exec prometheus wget -qO- http://<container>:<port>/metrics
```

---

## 🛡️ Prevention

> **Golden rule**: After any docker-compose.yml modification → `docker-compose up -d`, never `docker restart`

**Correct workflow**:
```bash
# 1. Modify docker-compose.yml
vim docker-compose.yml

# 2. Apply changes (NOT docker restart!)
docker-compose up -d

# 3. Verify
docker ps  # Should show "(healthy)" if healthcheck defined
```

**Best practices**:
- Use `up -d` in deployment scripts (Taskfile, Makefile)
- Don't confuse `restart: unless-stopped` (restart policy) with config application
- After VPS reboot, run `docker-compose up -d` to ensure latest config

---

## 🔗 Related Issues

- [[troubleshooting/monitoring/grafana-no-data]] - Dashboard shows no data
- [[troubleshooting/docker/disk-full-logs]] - Docker disk space issues

**For deep understanding**: See [[concepts/docker/docker-compose-basics]]

---

## 📊 Stats

```yaml
Encountered: 1 times
Last occurrence: 2026-01-19
Typical fix time: 5min
Projects affected: [[projects/2025-12-glasck-deployment]]
```

---

**Quick reference**:
```bash
# ❌ Wrong - doesn't apply new config
docker restart <container>

# ✅ Correct - recreates with current config
docker-compose up -d
```

---

**Last update**: 2026-01-19
