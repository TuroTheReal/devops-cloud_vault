# Prometheus & Grafana - Metrics Monitoring

## 📋 Metadata

```yaml
tags: [concept, monitoring, prometheus, grafana, metrics, status/learned]
created: 2025-12-23
updated: 2025-01-07
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 6h
```

**Prerequisites**: None
**Related to**: [[elk-stack-basics]], [[golden-signals-slo]], [[alerting-best-practices]]

---

## 🎯 TL;DR (30 seconds)

Prometheus collects numeric metrics (CPU, memory, request counts) from targets via HTTP scraping. Grafana provides dashboards to visualize these metrics over time. Together they form a complete metrics monitoring solution.

**Analogy**: Prometheus is like a health tracker that regularly checks your vitals (heart rate, temperature). Grafana is the app that shows you charts of your health trends over time.

---

## 📚 Prometheus Metric Types

| Type | Description | Example | Operation |
|------|-------------|---------|-----------|
| **Counter** | Only increases | `http_requests_total` | `rate()`, `increase()` |
| **Gauge** | Goes up and down | `temperature`, `memory_usage` | Direct value |
| **Histogram** | Distribution in buckets | `http_request_duration_seconds` | `histogram_quantile()` |
| **Summary** | Like histogram, quantiles computed client-side | `request_latency_seconds` | Direct quantiles |

### When to use what?
```
Event counter → Counter (requests, errors, bytes)
Current state → Gauge (connections, queue size, temperature)
Measure latency → Histogram (enables p50, p95, p99)
```

---

## 💻 PromQL Essentials

### Basic Operations
```promql
# Instant value
http_requests_total

# With labels
http_requests_total{job="api", status="200"}

# Range vector (last 5 min)
http_requests_total[5m]

# Rate (REQUIRED for counters)
rate(http_requests_total[5m])

# Sum by label
sum by (status) (rate(http_requests_total[5m]))
```

### Common Functions

| Function | Use | Example |
|----------|-----|---------|
| `rate()` | Change rate/sec (counters) | `rate(http_requests_total[5m])` |
| `increase()` | Total increase over period | `increase(http_requests_total[1h])` |
| `sum()` | Aggregation | `sum(rate(...))` |
| `avg()` | Average | `avg(cpu_usage)` |
| `max()` / `min()` | Extremes | `max(memory_usage)` |
| `histogram_quantile()` | Percentiles | `histogram_quantile(0.99, ...)` |
| `by` / `without` | Group/exclude labels | `sum by (job) (...)` |

### Common Patterns
```promql
# Error rate (%)
sum(rate(http_requests_total{status=~"5.."}[5m]))
/ sum(rate(http_requests_total[5m])) * 100

# Latency p99 (le = "less than or equal", bucket boundary)
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# CPU saturation
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage %
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

---

## 🔧 Common Exporters

| Exporter | Metrics | Port |
|----------|---------|------|
| **node-exporter** | CPU, RAM, Disk, Network | 9100 |
| **cadvisor** | Docker containers | 8080 |
| **postgres-exporter** | PostgreSQL stats | 9187 |
| **redis-exporter** | Redis stats | 9121 |
| **blackbox-exporter** | HTTP/TCP probes | 9115 |

---

## ⚠️ Key Pitfalls

| Issue | Solution |
|-------|----------|
| Prometheus disk fills up | Set `--storage.tsdb.retention.time=30d` and `--storage.tsdb.retention.size=10GB` |
| Alerts fire with stale data | Ensure `evaluation_interval <= scrape_interval` |
| Dashboards reset on restart | Use Grafana provisioning (dashboards as code) |
| Node exporter shows container stats | Mount host `/proc`, `/sys`, `/` and use `pid: host` |

---

## 🛠️ Essential Commands

```bash
# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Query metrics
curl 'http://localhost:9090/api/v1/query?query=up'

# Reload config without restart
curl -X POST http://localhost:9090/-/reload

# Check Alertmanager
curl http://localhost:9093/api/v2/status
```

---

## 📊 Stats

```yaml
Total time: 12h (60% assisted / 40% autonomous)
Used in: [[2024-XX-transcendence-monitoring/learnings]]
```

---

**Last update**: 2025-01-07
