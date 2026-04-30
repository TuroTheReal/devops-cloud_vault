# Docker Backup Strategies - Docker

## 📋 Metadata

```yaml
tags: [concept, docker, backup, disaster-recovery, status/learning]
created: 2026-01-20
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 10h
```

**Prerequisites**: [[docker-volumes-persistence]], [[docker-images-layers]]
**Related to**: [[docker-swarm-basics]], [[git-github-fundamentals]] (code backup)

---

## 🎯 TL;DR (30 seconds)

> Persistent volumes ≠ protected data. Backup = copy data elsewhere (different disk, server, cloud). No backup + disk crash = everything lost. Automate BEFORE disaster happens.
>
> **Analogy**: Volume = safe inside your house. Backup = copy of important documents at your parents' house. House fire = safe destroyed, but copies are safe.

---

## 🤔 When to Use?

### ✅ Good for
1. **Stateful application data**: Prometheus metrics, Grafana dashboards/config, Redis dumps
2. **Custom images**: Locally built images not pushed to registry
3. **Critical configurations**: docker-compose.yml, Traefik configs, encrypted secrets

### ❌ Bad for
- **Easily regenerable data** → rebuild instead of backup (node_modules, cache)
- **Public images** → re-pull from registry, no backup needed
- **Large logs** → use rotation/archiving, not classic backup (see [[MOC-Monitoring-Observability]])
- **Source code** → use Git, not file backup (see [[git-github-fundamentals]])

---

## 📚 Key Concepts

### 1. Volume Backup

**My understanding**:
> Docker volumes store persistent data but live on the same disk as the host. Backup = copy this data elsewhere. Two approaches: direct copy of volume folder, or temporary `docker run` to export.

**Why important**:
> Volume without backup = false sense of security. Disk crash, corruption, human error = data permanently lost.

**Mental model**:
```
Without backup:
┌─────────────────────────────────────┐
│           SAME DISK                 │
│  ┌─────────┐      ┌──────────────┐  │
│  │ Container│ ───→ │ Volume data  │  │  ← Disk crash = EVERYTHING lost
│  └─────────┘      └──────────────┘  │
└─────────────────────────────────────┘

With backup:
┌─────────────────────────────────────┐     ┌─────────────────┐
│           PRIMARY DISK              │     │  BACKUP (other) │
│  ┌─────────┐      ┌──────────────┐  │     │ ┌─────────────┐ │
│  │ Container│ ───→ │ Volume data  │──┼────→│ │ Dated copy  │ │
│  └─────────┘      └──────────────┘  │     │ └─────────────┘ │
└─────────────────────────────────────┘     └─────────────────┘
                                              ↑ External disk,
                                                S3, other server
```

---

### 2. Image Export/Compression

**My understanding**:
> `docker save` exports an image to .tar file. Useful for custom locally-built images. Compress with gzip/zstd to reduce size. Restore with `docker load`.

**Why important**:
> Custom images not pushed to registry = lost if host crashes. Export = backup of your build work.

**Key commands**:
```bash
# Export image
docker save myapp:latest | gzip > myapp_2026-01-20.tar.gz

# Restore image
gunzip -c myapp_2026-01-20.tar.gz | docker load

# With zstd compression (faster, better ratio)
docker save myapp:latest | zstd > myapp_2026-01-20.tar.zst
zstd -d myapp_2026-01-20.tar.zst | docker load
```

---

### 3. The 3-2-1 Rule

**My understanding**:
> Industry standard backup rule: **3** copies of data, on **2** different media types, with **1** off-site. Not mandatory from day 1, but the target to aim for.

**Why important**:
> Single copy on the same server = not a real backup. Protects against: losing one copy, media failure, local disaster.

**Practical application**:
```
3 copies:
├── Original (Docker volume on VPS)
├── Local backup (different disk/partition on VPS)
└── Off-site backup (S3, other server, home NAS)

2 media types:
├── VPS SSD
└── Object storage (S3) or external HDD

1 off-site:
└── Geographically separated (different datacenter, cloud, home)
```

**Reality check**: Start with 2 copies (original + local backup), then add off-site when ready.

---

### 4. Naming Convention for Backups

**My understanding**:
> Consistent naming = easy to find, sort, and automate rotation. Include: what, when, optionally version/type.

**Why important**:
> `backup.tar.gz` tells you nothing. `prometheus_data_2026-01-20_daily.tar.gz` tells you everything.

**Recommended pattern**:
```
{service}_{type}_{YYYY-MM-DD}_{frequency}.{ext}

Examples:
prometheus_data_2026-01-20_daily.tar.gz
grafana_config_2026-01-20_daily.tar.gz
myapp_image_2026-01-20_weekly.tar.zst
full_backup_2026-01-15_monthly.tar.gz
```

**Directory structure**:
```
/backups/
├── daily/
│   ├── prometheus_data_2026-01-20.tar.gz
│   ├── prometheus_data_2026-01-19.tar.gz
│   └── grafana_data_2026-01-20.tar.gz
├── weekly/
│   └── full_backup_2026-01-15.tar.gz
├── monthly/
│   └── full_backup_2026-01-01.tar.gz
└── images/
    └── myapp_v1.2.3_2026-01-20.tar.zst  ← version in filename for images
```

**ISO 8601 date format** (`YYYY-MM-DD`): sorts chronologically, unambiguous internationally.

---

### 5. Git as Backup for Code

**My understanding**:
> Git + remote (GitHub/GitLab) = excellent backup for source code. Every clone is a full backup. But Git is NOT for: large binary files, secrets, database dumps.

**Why important**:
> Don't backup what Git already handles. Focus Docker backup on DATA, not code.

**What Git covers**:
```
✅ Git handles (no need for Docker backup):
├── Source code
├── docker-compose.yml
├── Dockerfile
├── Configuration files (non-secret)
└── Documentation

❌ Git does NOT cover (need Docker backup):
├── Volume data (databases, metrics)
├── Runtime secrets (use Docker secrets + backup)
├── Custom built images (if not pushed to registry)
└── User uploads / generated files
```

**Note**: Versioning (Semantic Versioning like v1.2.3) is a Git/release management topic → see [[git-semantic-versioning]].

---

### 6. Automation Levels

**My understanding**:
> Manual backup = forgotten backup. Automate with Taskfile, cron, or scripts. Start semi-auto (scripted but manually triggered), evolve to full auto (cron/systemd timer).

**Why important**:
> The backup you forget to make is the one you'll need.

**Progression**:
```
Level 1: Manual
└── Commands typed by hand when you remember
    → Risk: forgotten, inconsistent

Level 2: Semi-Auto (current) ✓
└── Taskfile with backup commands
    → task backup:volumes
    → task backup:images
    → Risk: need to remember, but at least it's standardized

Level 3: Full Auto (target)
└── Cron/systemd timer runs tasks
    → Daily automatic backup
    → Notification on failure
    → Old backup rotation
```

---

## 💻 Minimal Example

### Context
Backup Prometheus and Grafana volumes with Taskfile, compression, and dated naming.

### Code

**Taskfile.yml**:
```yaml
version: '3'

vars:
  BACKUP_DIR: /home/user/backups
  DATE: '{{now | date "2006-01-02"}}'

tasks:
  backup:volumes:
    desc: Backup all important Docker volumes
    cmds:
      - mkdir -p {{.BACKUP_DIR}}/daily
      # Backup Prometheus data
      - docker run --rm -v prometheus_data:/data -v {{.BACKUP_DIR}}/daily:/backup alpine tar czf /backup/prometheus_data_{{.DATE}}.tar.gz -C /data .
      # Backup Grafana data
      - docker run --rm -v grafana_data:/data -v {{.BACKUP_DIR}}/daily:/backup alpine tar czf /backup/grafana_data_{{.DATE}}.tar.gz -C /data .
      - echo "Backup completed: {{.BACKUP_DIR}}/daily/*_{{.DATE}}.tar.gz"

  backup:images:
    desc: Export and compress custom images
    vars:
      VERSION: '{{default "latest" .VERSION}}'
    cmds:
      - mkdir -p {{.BACKUP_DIR}}/images
      - docker save myapp:{{.VERSION}} | gzip > {{.BACKUP_DIR}}/images/myapp_{{.VERSION}}_{{.DATE}}.tar.gz
      - echo "Image exported: myapp_{{.VERSION}}_{{.DATE}}.tar.gz"

  backup:all:
    desc: Full backup (volumes + images)
    cmds:
      - task: backup:volumes
      - task: backup:images

  backup:rotate:
    desc: Remove backups older than 7 days
    cmds:
      - find {{.BACKUP_DIR}}/daily -type f -mtime +7 -delete
      - echo "Old backups cleaned up"

  restore:volume:
    desc: Restore a volume from backup
    vars:
      VOLUME: '{{.VOLUME}}'
      BACKUP_FILE: '{{.BACKUP_FILE}}'
    cmds:
      - docker run --rm -v {{.VOLUME}}:/data -v {{.BACKUP_DIR}}:/backup alpine sh -c "rm -rf /data/* && tar xzf /backup/{{.BACKUP_FILE}} -C /data"
      - echo "Volume {{.VOLUME}} restored from {{.BACKUP_FILE}}"
```

**Usage**:
```bash
# Backup all volumes
task backup:volumes

# Backup images with version tag
task backup:images VERSION=v1.2.3

# Full backup
task backup:all

# Cleanup old backups
task backup:rotate

# Restore specific volume
task restore:volume VOLUME=prometheus_data BACKUP_FILE=daily/prometheus_data_2026-01-20.tar.gz
```

**Output**: Dated .tar.gz files in `/home/user/backups/daily/`

---

## 📊 Current Status

```yaml
Implementation: Semi-Auto via Taskfile

What works:
  - Prometheus/Grafana volume backup (dated, compressed)
  - Custom image export
  - Standardized commands
  - Proper naming convention

What's missing:
  - Cron automation (full auto)
  - Off-site backup (S3 or other server)
  - Old backup rotation (automated)
  - Failure notification
  - Containerized DB (not applicable - DB is bare metal)

Next steps:
  - [ ] Add cron for daily backup
  - [ ] Configure rclone for S3/Backblaze sync
  - [ ] Rotation script (keep 7 daily, 4 weekly, 3 monthly)
  - [ ] Notification on failure (email or webhook)
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Backup while container is writing

**Symptom**: Corrupted or inconsistent backup because data was being written.

**What I did wrong**:
```bash
# ❌ Wrong - backup while Prometheus writes
docker run --rm -v prometheus_data:/data ... tar czf ...
# Potentially inconsistent data
```

**Solution**:
```bash
# ✅ Option 1: Stop container before backup (downtime)
docker stop prometheus
docker run --rm -v prometheus_data:/data ... tar czf ...
docker start prometheus

# ✅ Option 2: Use app's native tools
# Prometheus: API snapshot
curl -X POST http://localhost:9090/api/v1/admin/tsdb/snapshot
# Then backup the snapshot folder

# ✅ Option 3: Accept risk for tolerant apps
# Grafana SQLite handles hot backups well
```

**Lesson**: Some apps need proper stop or native tools for consistent backup.

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> Why is a Docker volume not a backup by itself?</summary>

**Answer**: The volume is on the same disk as the host. Disk crash, corruption, or human error → volume AND data lost. Backup = copy elsewhere (different disk, server, cloud).
</details>

<details>
<summary><strong>Q2:</strong> What is the 3-2-1 backup rule?</summary>

**Answer**: 3 copies of data, on 2 different media types, with 1 off-site (geographically separated). Protects against: losing one copy, media failure, local disaster (fire, flood).
</details>

<details>
<summary><strong>Q3:</strong> What naming convention should you use for backups?</summary>

**Answer**: `{service}_{type}_{YYYY-MM-DD}_{frequency}.{ext}` - Example: `prometheus_data_2026-01-20_daily.tar.gz`. ISO 8601 date sorts chronologically and is unambiguous.
</details>

<details>
<summary><strong>Q4:</strong> Should you backup source code with Docker backup?</summary>

**Answer**: No. Use Git + remote (GitHub/GitLab) for code backup. Every clone is a full backup. Docker backup is for DATA: volumes, custom images, runtime secrets - not code.
</details>

---

## 📊 Stats

```yaml
Total time: 2h (50% assisted / 50% autonomous)
Used in: [[2025-12-glasck-deployment]]
```

---

**Last update**: 2026-01-20
**Next review**: 2026-02-20 (+1 month)
