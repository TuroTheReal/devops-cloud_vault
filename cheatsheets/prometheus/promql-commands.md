# PromQL - Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, prometheus, promql, monitoring, status/documented]
created: 2025-01-07
```

**Related concepts**: [[concepts/monitoring/prometheus-grafana-basics]], [[concepts/monitoring/golden-signals-slo]]

---

## 🚀 Quick Start

```promql
up                                    # Instant value
up{job="prometheus"}                  # Filter by label
rate(http_requests_total[5m])         # Rate (REQUIRED for counters)
sum by (status) (rate(...))           # Aggregate by label
```

---

## 🏷️ Selectors & Matchers

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Exact match | `job="api"` |
| `!=` | Not equal | `job!="test"` |
| `=~` | Regex match | `status=~"2..\|3.."` |
| `!~` | Regex not match | `status!~"5.."` |

**⚠️ IMPORTANT for Grafana**: Always `=~` with variables!
```promql
# ❌ Wrong
metric{instance="$instance"}

# ✅ Correct
metric{instance=~"$instance"}
```

---

## ⏱️ Range Vectors

```promql
metric[5m]              # Last 5 minutes
metric[1h]              # Last hour
metric[5m] offset 1d    # 5 min window, 1 day ago
```

Units: `s` (seconds), `m` (minutes), `h` (hours), `d` (days), `w` (weeks), `y` (years)

---

## 🔧 Functions

### For Counters (always increasing)

| Function | Use | Example |
|----------|-----|---------|
| `rate()` | Per-second rate (smoothed) | `rate(requests_total[5m])` |
| `irate()` | Instant rate | `irate(requests_total[5m])` |
| `increase()` | Total increase | `increase(requests_total[1h])` |

### For Gauges (up and down)

| Function | Use | Example |
|----------|-----|---------|
| `avg_over_time()` | Average | `avg_over_time(cpu[1h])` |
| `max_over_time()` | Maximum | `max_over_time(memory[1h])` |
| `delta()` | Difference | `delta(temp[1h])` |

### For Histograms

```promql
# Percentile (le = "less than or equal", histogram bucket label)
histogram_quantile(0.99,
  sum(rate(http_duration_bucket[5m])) by (le)
)

# le values are bucket boundaries: 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10
```

---

## 📊 Aggregation

| Operator | Description |
|----------|-------------|
| `sum` | Sum |
| `avg` | Average |
| `min` / `max` | Extremes |
| `count` | Count series |
| `topk(n, ...)` | Top N |
| `bottomk(n, ...)` | Bottom N |

### Grouping

```promql
sum by (status) (rate(...))        # Keep status label
sum without (instance) (rate(...)) # Remove instance label
```

---

## ⚖️ Comparison Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `>` | Greater than | `cpu_usage > 80` |
| `<` | Less than | `memory_free < 1e9` |
| `>=` | Greater or equal | `error_rate >= 0.01` |
| `<=` | Less or equal | `latency_p99 <= 0.5` |
| `==` | Equal | `up == 1` |
| `!=` | Not equal | `up != 0` |

### With bool modifier (returns 1 or 0)

```promql
up == bool 1              # Returns 1 if up, 0 if down
cpu_usage > bool 80       # Returns 1 if > 80, else 0
```

### Logical operators

```promql
metric_a and metric_b     # Both exist
metric_a or metric_b      # Either exists
metric_a unless metric_b  # A but not B
```

---

## 🎯 Golden Signals Patterns

### Latency
```promql
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
```

### Traffic
```promql
sum(rate(http_requests_total[5m]))
```

### Errors
```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
/ sum(rate(http_requests_total[5m])) * 100
```

### Saturation
```promql
# CPU usage %
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage %
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100
```

---

## 🗄️ Service-Specific Patterns

### PostgreSQL
```promql
# Cache hit ratio %
sum(pg_stat_database_blks_hit)
/ (sum(pg_stat_database_blks_hit) + sum(pg_stat_database_blks_read)) * 100

# Connection usage %
sum(pg_stat_activity_count) / pg_settings_max_connections * 100
```

### Redis
```promql
# Hit rate %
sum(rate(redis_keyspace_hits_total[5m]))
/ (sum(rate(redis_keyspace_hits_total[5m])) + sum(rate(redis_keyspace_misses_total[5m]))) * 100

# Memory usage %
redis_memory_used_bytes / redis_memory_max_bytes * 100
```

### Traefik
```promql
# Requests/sec
sum by (entrypoint) (rate(traefik_entrypoint_requests_total[5m]))

# Error rate %
sum(rate(traefik_entrypoint_requests_total{code=~"5.."}[5m]))
/ sum(rate(traefik_entrypoint_requests_total[5m])) * 100
```

### Docker
```promql
# CPU by container %
sum by (name) (rate(container_cpu_usage_seconds_total[5m])) * 100

# Restarts
increase(container_restart_count[1h])
```

---

## 📝 Grafana Variables

```promql
# For dropdown
label_values(up, instance)
label_values(up, job)

# In panel query (always =~)
rate(http_requests_total{instance=~"$instance", job=~"$job"}[5m])
```

---

## ✅ Best Practices

1. **Always `rate()` on counters** - raw counter = useless increasing number
2. **Always `=~` with Grafana variables** - `=` causes "No data"
3. **Range vector choice**: `[1m]` reactive/noisy, `[5m]` balanced, `[15m]` smooth
4. **Test in Prometheus UI first** before adding to Grafana

---

**Last update**: 2025-01-07
