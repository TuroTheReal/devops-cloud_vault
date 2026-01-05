# Docker Disk Full - Container Logs

## 📋 Metadata

```yaml
tags: [troubleshooting, docker, storage, logs, disk-space, status/documented]
created: 2026-01-05
updated: 2026-01-05
severity: 🔴 Critical
time-to-fix: 2h
```

**Related concept**: [[concepts/docker/docker-compose-basics]], [[concepts/docker/docker-logging]]
**Affected versions**: Docker 20.x+, all versions without log rotation config

---

## 🔍 Symptom

> **Disk at 100%, containers won't start, site down with "no space left" errors**
>
> System becomes unresponsive, Docker operations fail, existing services crash, SSH not available.

```bash
# Error messages
ENOSPC: no space left on device

# Disk check shows 98-100% usage
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        226G   225G  1.0G  99% /
```

**Common scenarios where this occurs**:
- Production site crashes after weeks/months without restart
- Verbose application logging (every API request logged)
- Error loops causing massive log output
- First deployment without log rotation configured

---

## 🎯 Root Cause

> Docker logs stored in `/var/lib/docker/containers/<id>/<id>-json.log` with NO rotation by default. Files grow infinitely until disk is full.

**Why this happens**:
- Default `json-file` driver has no size limits
- Every `console.log()` / `print()` / stdout output → JSON log file
- Logs accumulate: 100MB → 1GB → 10GB → disk full
- No automatic cleanup or rotation mechanism

---

## ✅ Quick Fix

### Solution 1: Truncate logs immediately (Recommended for emergency)

```bash
# Find large log files (>100MB)
sudo find /var/lib/docker/containers -name "*-json.log" -size +100M -exec ls -lh {} \;

# Empty all large logs (SAFE - containers keep running!)
sudo find /var/lib/docker/containers -name "*-json.log" -size +100M -exec truncate -s 0 {} \;

# Verify disk space recovered
df -h
```

**Explanation**: `truncate -s 0` empties the file without deleting it. Docker keeps running, no restart needed. Instant disk space recovery.

---

### Solution 2: Configure log rotation (Permanent fix)

**In docker-compose.yml** (add to EVERY service):

```yaml
version: '3.8'

services:
  myservice1:
    image: my-service1
    logging:
      driver: "json-file"
      options:
        max-size: "50m"     # Max 50MB per log file
        max-file: "3"       # Keep 3 files (150MB total)
    # ... rest of config

  myservice2:
    image: my-service2
    logging:
      driver: "json-file"
      options:
        max-size: "50m"
        max-file: "3"
```

```bash
# Apply changes (MUST recreate containers!)
docker-compose down
docker-compose up -d

# Verify config is applied
docker inspect backend --format '{{.HostConfig.LogConfig}}'
# Expected: {json-file map[max-file:3 max-size:50m]}
```

**When to use**: Always! Add this to every docker-compose.yml file you create.

---

## 🔬 Diagnosis Steps

**If quick fix doesn't work, debug with**:

```bash
# 1. Check overall disk usage
df -h

# 2. Find what's using space
sudo du -sh /* 2>/dev/null | sort -hr | head -10
sudo du -sh /var/lib/docker/* 2>/dev/null | sort -hr

# 3. Check Docker-specific usage
docker system df

# 4. Identify log files by container name
for log in $(sudo find /var/lib/docker/containers -name "*-json.log"); do
  size=$(du -h "$log" | cut -f1)
  container_id=$(echo "$log" | grep -oP '/containers/\K[^/]+')
  container_name=$(docker inspect "$container_id" 2>/dev/null --format '{{.Name}}' | sed 's/\///' || echo "stopped")
  echo "$size - $container_name ($container_id)"
done | sort -hr | head -10

# 5. Verify current log config
docker inspect <container_name> --format '{{.HostConfig.LogConfig}}'
```

---

## 🛡️ Prevention

> Always configure log rotation before deploying to production. Add to all docker-compose.yml files.

**Best practices**:
- **Always add logging block** to every service in docker-compose.yml
- **Set global defaults** in `/etc/docker/daemon.json` as safety net:
  ```json
  {
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "50m",
      "max-file": "3"
    }
  }
  ```
- **Monitor disk usage** with cron alerts (email when >85% full)
- **Reduce application verbosity** in production (don't log every request)
- **Regular cleanup** of unused images/volumes: `docker system prune`

---

## 🔗 Related Issues

- [[troubleshooting/docker/build-cache-full]] - Docker build cache consuming space
- [[troubleshooting/docker/image-cleanup]] - Unused images filling disk
- [[troubleshooting/system/disk-monitoring]] - Proactive disk monitoring setup

**For deep understanding**: See [[concepts/docker/docker-logging-drivers]]

---

## 📊 Stats

```yaml
Encountered: 1 time
Last occurrence: 2026-01-05
Typical fix time: 2min (emergency) / 10min (permanent)
Projects affected: [[projects/glasck-deployment]]
```

---

**Common mistakes**:
- ❌ Using `docker restart` to apply log config (doesn't work - must use `--force-recreate`)
- ❌ Deleting log files with `rm` while container running (causes corruption - use `truncate`)
- ❌ Not verifying config with `docker inspect` after changes

**Quick reference**:
```bash
# Emergency: empty logs NOW
sudo find /var/lib/docker/containers -name "*-json.log" -size +100M -exec truncate -s 0 {} \;

# Prevention: add to docker-compose.yml
logging:
  driver: "json-file"
  options:
    max-size: "50m"
    max-file: "3"

# Apply: recreate containers
docker-compose up -d --force-recreate

# Verify: check config
docker inspect <name> --format '{{.HostConfig.LogConfig}}'
```

---

**Last update**: 2026-01-05
