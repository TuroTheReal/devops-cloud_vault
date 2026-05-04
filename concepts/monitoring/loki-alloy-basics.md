---
tags:
  - concept
  - monitoring
  - loki
  - alloy
  - grafana
  - logs
  - status/learning
created: 2026-01-19
difficulty: 3
time-to-master: 6h
---

# Loki & Alloy - Lightweight Logs for Grafana

**Prerequisites**: [[prometheus-grafana-basics]], [[concepts/docker/docker-compose-basics]]
**Related to**: [[elk-stack-basics]], [[alerting-best-practices]]

---

## 🎯 TL;DR (30 seconds)

Loki = log storage that indexes only **labels** (not content). Alloy = collector that grabs logs and sends them to Loki. Together with Grafana = lightweight, Grafana-native log stack.

**Analogy**:
- **ELK** = library with full index of every word in every book (powerful but heavy)
- **Loki** = library that only indexes shelves and categories (lightweight, you search by section then browse)

You don't search "error timeout" across all logs. You filter by container/service first, then read.

---

## 🤔 When to Use?

### ✅ Good for
1. **Existing Grafana stack**: You already have Prometheus + Grafana → Loki integrates natively
2. **Limited resources**: VPS with low RAM → Loki runs on 256-512MB vs 4GB+ for ELK
3. **Metrics/logs correlation**: Same Prometheus labels → easy switch between metrics and logs in Grafana

### ❌ Bad for
- **Complex full-text search** → Use [[elk-stack-basics]] instead (regex on massive content)
- **Analytics/BI on logs** → ELK with Kibana dashboards
- **Long-term compliance/audit** → Enterprise solutions (Splunk, Datadog)

---

## 📚 Key Concepts

### 1. Labels = your only indexation

**My understanding**:
Loki does NOT index log content. It only indexes labels (metadata). When you query, you filter by labels first, then Loki scans the content of matching chunks.

**Why important**:
This is what makes Loki 10x lighter than Elasticsearch. Less indexing = less CPU/RAM/storage.

**Mental model**:
```
ELK:  "Find 'error' in 10TB of logs" → scan giant index → result
Loki: "Find 'error' in container=api last 5 minutes" → filter by label → scan small chunk → result
```

---

### 2. Alloy = the modern collector

**My understanding**:
Alloy (formerly Grafana Agent) collects logs and sends them to Loki. It replaces Promtail. Config uses "River" (Grafana's declarative language).

**Why important**:
One agent for everything: logs, metrics, traces. No need for separate Promtail + Node Exporter.

**Alloy Pipeline**:
```
discovery.docker → discovery.relabel → loki.source.docker → loki.write
     │                    │                    │                 │
  Find the            Add the              Read Docker       Send to
  containers          labels               logs              Loki
```

---

### 3. LogQL = PromQL for logs

**My understanding**:
LogQL looks like PromQL. You filter by labels `{container="api"}` then you can parse/filter content with `|=`, `|~`, `| json`, etc.

**Why important**:
If you know PromQL, LogQL feels natural. Same label syntax, same logic.

**Basic syntax**:
```logql
# Filter by label
{container_name="reverse-proxy"}

# Filter by content (contains)
{container_name="api-fastify-prod"} |= "error"

# Regex on content
{container_name="api-fastify-prod"} |~ "status=[45][0-9]{2}"

# Parse JSON and filter
{container_name="api-fastify-prod"} | json | level="error"
```

---

## 💻 Minimal Example

### Context
Docker stack with Alloy collecting logs from all containers and sending to Loki.

### Architecture
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Containers │────▶│    Alloy    │────▶│    Loki     │
│  (Docker)   │     │  (collect)  │     │   (store)   │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                                        ┌──────▼──────┐
                                        │   Grafana   │
                                        │  (explore)  │
                                        └─────────────┘
```

### Alloy Config (config.alloy)
```hcl
// 1. Docker container discovery
discovery.docker "containers" {
  host = "unix:///var/run/docker.sock"
}

// 2. Relabeling - add Docker metadata as labels
discovery.relabel "docker_labels" {
  targets = discovery.docker.containers.targets

  // Extract container name (remove "/" prefix)
  rule {
    source_labels = ["__meta_docker_container_name"]
    regex         = "/(.*)"
    target_label  = "container_name"
  }

  // Keep image name
  rule {
    source_labels = ["__meta_docker_container_image"]
    target_label  = "image"
  }

  // Compose service (if exists)
  rule {
    source_labels = ["__meta_docker_container_label_com_docker_compose_service"]
    target_label  = "compose_service"
  }
}

// 3. Read Docker logs with labels
loki.source.docker "docker_logs" {
  host       = "unix:///var/run/docker.sock"
  targets    = discovery.relabel.docker_labels.output
  forward_to = [loki.write.local.receiver]
}

// 4. Send to Loki
loki.write "local" {
  endpoint {
    url = "http://loki:3100/loki/api/v1/push"
  }
}
```

### Minimal Loki Config (loki-config.yml)
```yaml
auth_enabled: false

server:
  http_listen_port: 3100

common:
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory
  replication_factor: 1
  path_prefix: /var/lib/loki

schema_config:
  configs:
  - from: 2020-01-01
    store: tsdb
    object_store: filesystem
    schema: v13
    index:
      prefix: index_
      period: 24h

storage_config:
  filesystem:
    directory: /var/lib/loki/chunks
```

### Grafana Datasource (loki.yml)
```yaml
apiVersion: 1
datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki:3100
    isDefault: false
```

**Usage in Grafana** → Explore → Datasource: Loki → Query:
```logql
{container_name="api-fastify-prod"} |= "error"
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Loki "target down" but logs arriving

**Symptom**: Prometheus alert "Loki down" while logs display fine in Grafana

**What I did wrong**:
Loki image is distroless → no shell, no wget/curl for Docker healthcheck.

```yaml
# ❌ Wrong - healthcheck impossible on distroless
healthcheck:
  test: ["CMD", "wget", "-q", "--spider", "http://localhost:3100/ready"]
```

**Solution**:
```yaml
# ✅ Correct - disable healthcheck, check manually
healthcheck:
  disable: true  # distroless, check via: curl http://localhost:3100/ready
```

**Time wasted**: 30min
**Lesson**: Distroless images = no classic Docker healthcheck, use external probe or disable.

---

### Pitfall 2: Labels not visible in Grafana

**Symptom**: Query `{container_name="api"}` returns nothing, but logs exist

**What I did wrong**:
Relabeling misconfigured - the label `__meta_docker_container_name` includes a `/` prefix.

```hcl
# ❌ Wrong - keeps the "/" in the name
rule {
  source_labels = ["__meta_docker_container_name"]
  target_label  = "container_name"
}
```

**Solution**:
```hcl
# ✅ Correct - regex to remove the "/"
rule {
  source_labels = ["__meta_docker_container_name"]
  regex         = "/(.*)"    # Capture everything after /
  target_label  = "container_name"
}
```

**Time wasted**: 1h
**Lesson**: Always check raw labels with `{__name__=~".+"}` or in Alloy targets before relabeling.

---

### Pitfall 3: Fake positives - searching content that "looks like" errors

**Symptom**: Query `{container_name="api"} |= "error"` returns tons of results, most are not actual errors

**What I did wrong**:
Simple string match catches everything containing "error" - including normal logs like:
- `"Processing error_handler middleware"`
- `"error_count: 0"`
- `"No errors found"`

```logql
# ❌ Wrong - too broad, catches false positives
{container_name="api"} |= "error"
```

**Solution**:
```logql
# ✅ Correct - use JSON parsing + level filter
{container_name="api"} | json | level="error"

# ✅ Or case-insensitive regex with word boundary
{container_name="api"} |~ "(?i)\\berror\\b.*:"

# ✅ Or exclude known false positives
{container_name="api"} |= "error" != "error_count" != "error_handler"
```

**Time wasted**: 30min debugging "errors" that weren't errors
**Lesson**: For JSON logs, always parse first then filter by level. For plain text, use word boundaries or exclusions.

---

## 🔄 Loki vs ELK - When to choose what?

| Criteria | Loki | ELK |
|----------|------|-----|
| **Minimum RAM** | 256MB | 4GB+ |
| **Query language** | LogQL (≈PromQL) | Lucene/KQL |
| **Indexation** | Labels only | Full-text |
| **Grafana integration** | Native | Plugin |
| **Main use case** | Debug, metrics correlation | Analytics, full-text search |
| **Learning curve** | Easy if you know Prometheus | More complex |

**Simple rule**:
- You have Prometheus + Grafana → **Loki**
- You need full-text search or analytics → **ELK**

---

## 📊 Stats

```yaml
Total time: 6h (40% assisted / 60% autonomous)
Used in: [[projects/2025-12-glasck-deployment]]
```

---

**Last update**: 2026-01-19
**Next review**: 2026-02-19
