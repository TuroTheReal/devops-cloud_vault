# Prometheus & Grafana - Metrics Monitoring

## 📋 Metadata

```yaml
tags: [concept, monitoring, prometheus, grafana, metrics, status/learned]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 6h
```

**Prerequisites**: None
**Related to**: [[elk-stack-basics]]

---

## 🎯 TL;DR (30 seconds)

Prometheus collects numeric metrics (CPU, memory, request counts) from targets via HTTP scraping. Grafana provides beautiful dashboards to visualize these metrics over time. Together they form a complete metrics monitoring solution.

**Analogy**: Prometheus is like a health tracker that regularly checks your vitals (heart rate, temperature). Grafana is the app that shows you charts of your health trends over time.

---

## 💻 Key Implementation from Transcendence

```yaml
# Prometheus - Metrics collection and storage
prometheus:
  image: prom/prometheus:latest
  ports:
    - "9090:9090"
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - ./prometheus/alert_rules.yml:/etc/prometheus/alert_rules.yml:ro
    - prometheus_data:/prometheus
  command:
    - '--config.file=/etc/prometheus/prometheus.yml'
    - '--storage.tsdb.path=/prometheus'
    - '--storage.tsdb.retention.time=30d'
    - '--storage.tsdb.retention.size=10GB'
    - '--web.enable-lifecycle'
    - '--web.enable-admin-api'
  networks:
    - app-network

# Node Exporter - System metrics
node-exporter:
  image: prom/node-exporter:latest
  ports:
    - "9100:9100"
  volumes:
    - /proc:/host/proc:ro
    - /sys:/host/sys:ro
    - /:/rootfs:ro
  command:
    - '--path.procfs=/host/proc'
    - '--path.rootfs=/rootfs'
    - '--path.sysfs=/host/sys'
    - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'
    - '--collector.processes'
  networks:
    - app-network
  pid: host

# Grafana - Visualization
grafana:
  image: grafana/grafana:latest
  ports:
    - "3001:3000"
  volumes:
    - ./grafana/grafana.ini:/etc/grafana/grafana.ini:ro
    - ./grafana/provisioning:/etc/grafana/provisioning:ro
    - grafana_data:/var/lib/grafana
  environment:
    - GF_SECURITY_ADMIN_USER=${GRAFANA_USER}
    - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    - GF_USERS_ALLOW_SIGN_UP=false
    - GF_INSTALL_PLUGINS=grafana-clock-panel
    - GF_PROVISIONING_ENABLED=true
  depends_on:
    - prometheus
  networks:
    - app-network

# Alertmanager - Alert routing
alertmanager:
  image: prom/alertmanager:latest
  ports:
    - "9093:9093"
  volumes:
    - ./alertmanager/alertmanager.yml.template:/etc/alertmanager/alertmanager.yml.template:ro
    - ./alertmanager/templates/email.tmpl:/etc/alertmanager/email.tmpl:ro
    - alertmanager_data:/alertmanager
  environment:
    - ALERT_EMAIL=${ALERT_EMAIL}
    - ALERT_PASSWORD=${ALERT_PASSWORD}
  depends_on:
    - prometheus
  networks:
    - app-network
```

**Prometheus config** (prometheus.yml):
```yaml
global:
  scrape_interval: 15s      # Scrape targets every 15s
  evaluation_interval: 15s  # Evaluate rules every 15s

# Alerting configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['alertmanager:9093']

# Load alert rules
rule_files:
  - "/etc/prometheus/alert_rules.yml"

# Scrape configurations
scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter (system metrics)
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  # Application metrics
  - job_name: 'app'
    static_configs:
      - targets: ['dev:3001']  # App exposes /metrics endpoint
```

---

## ⚠️ Key Pitfalls from Experience

### 1. Retention Settings
**Issue**: Prometheus disk fills up quickly
**Solution**: Set retention policies via `--storage.tsdb.retention.time` and `--storage.tsdb.retention.size`

### 2. Scrape Interval vs Evaluation Interval
**Issue**: Alerts fire with old data
**Solution**: Ensure `evaluation_interval <= scrape_interval` for timely alerts

### 3. Grafana Provisioning
**Issue**: Dashboards reset on container restart
**Solution**: Use provisioning to persist datasources and dashboards as code

### 4. Node Exporter Host Access
**Issue**: Node exporter shows wrong metrics
**Solution**: Mount host /proc, /sys, / as read-only and use `pid: host`

---

## 🔧 Essential Commands

```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Query metrics via PromQL
curl 'http://localhost:9090/api/v1/query?query=up'

# Reload Prometheus config (without restart)
curl -X POST http://localhost:9090/-/reload

# Check Alertmanager status
curl http://localhost:9093/api/v2/status

# Access Grafana
http://localhost:3001
```

---

## 📊 Common Metrics to Monitor

From Transcendence implementation:
- **System**: CPU, memory, disk usage (node-exporter)
- **Application**: HTTP request rate, response time, error rate
- **Custom**: Business metrics (active users, transactions/sec)

---

## 📊 Stats

```yaml
Total time: 12h (60% assisted / 40% autonomous)
Status: ✅ Mastered
Used in: [[2024-transcendence-glasck-extraction/learnings]]
```

---

## 🧠 Retrieval Practice

Test your understanding without looking back:

<details>
<summary><strong>Q1:</strong> What's the difference between Prometheus and Grafana, and why do you need both?</summary>

**Answer**: Prometheus collects and stores numeric metrics via HTTP scraping (the data engine), Grafana visualizes metrics with dashboards and charts (the presentation layer). Prometheus handles data collection/storage, Grafana makes it human-readable. Together they form complete monitoring: collection + visualization.
</details>

<details>
<summary><strong>Q2:</strong> Why must evaluation_interval be less than or equal to scrape_interval?</summary>

**Answer**: evaluation_interval determines how often alert rules are evaluated, scrape_interval determines how often metrics are collected. If evaluation > scrape (e.g., evaluate every 60s but scrape every 15s), alerts fire with old stale data causing delays. Keep evaluation <= scrape for timely alerts with fresh data.
</details>

<details>
<summary><strong>Q3:</strong> What's the purpose of node-exporter and why does it need special host access?</summary>

**Answer**: node-exporter collects system-level metrics (CPU, memory, disk). Needs to mount host /proc, /sys, and / as read-only plus `pid: host` to access actual host metrics instead of container metrics. Without this, shows container stats not host stats, giving wrong monitoring data.
</details>

<details>
<summary><strong>Q4:</strong> Why should you use Grafana provisioning instead of manual dashboard creation?</summary>

**Answer**: Manual dashboards reset on container restart - all customization lost. Provisioning defines datasources and dashboards as code (YAML/JSON files), persisted across restarts. Enables version control, reproducibility, and infrastructure-as-code approach. Configure via /etc/grafana/provisioning directory.
</details>

---

**Last update**: 2025-12-23
