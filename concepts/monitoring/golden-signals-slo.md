# Golden Signals & SLO/SLI/SLA - Monitoring

## 📋 Metadata

```yaml
tags: [concept, monitoring, sre, observability, status/learned]
created: 2025-01-07
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 6h
```

**Prerequisites**: [[prometheus-grafana-basics]]
**Related to**: [[alerting-best-practices]], [[elk-stack-basics]]

---

## 🎯 TL;DR (30 seconds)

**Golden Signals** are 4 universal metrics (Latency, Traffic, Errors, Saturation) that cover 90% of problems for any service. **SLO** defines your reliability targets (e.g., 99.9% availability), **SLI** measures if you're hitting them, and **SLA** is the contract with customers.

**Analogy**: Golden Signals are like a car dashboard (speed, fuel, temperature, warning lights). You don't need 50 dials - these 4 tell you if everything's fine.

---

## 🤔 When to Use?

### ✅ Good for
1. **New service in prod**: ALWAYS start with 4 Golden Signals before adding custom metrics
2. **Alerting**: Base alerts on symptoms (Golden Signals), not causes
3. **SLO Budgeting**: Know when you can deploy vs when you need to stabilize

### ❌ Bad for
- **Deep debugging** → Use distributed tracing (Jaeger, Tempo) as complement
- **Business metrics** → Revenue, conversion are not Golden Signals

---

## 📚 Key Concepts

### 1. The 4 Golden Signals (Google SRE)

Google distilled decades of experience into 4 metrics that apply to ANY service (API, DB, cache, queue). If one is red, your service has a problem.

| Signal | Measures | Example | Typical Threshold |
|--------|----------|---------|-------------------|
| **Latency** | Response time | p99 API response | < 500ms |
| **Traffic** | Throughput | Requests/sec | baseline ±50% |
| **Errors** | Failure rate | 5xx / total | < 0.1% |
| **Saturation** | Resource usage | CPU, connections | < 80% |

**Why it matters**: These 4 are correlated. If Saturation ↑ → Latency ↑ → Errors ↑ → Traffic may ↓ (clients give up).

```
User Request → [Traffic] → Service → [Latency] → Response
                              ↓
                     [Saturation] → [Errors]
```

---

### 2. SLI / SLO / SLA - The Reliability Pyramid

| Term | Definition | Example | Who defines |
|------|------------|---------|-------------|
| **SLI** (Indicator) | The metric measured | `successful_requests / total_requests` | Engineer |
| **SLO** (Objective) | Internal target | 99.9% of requests OK | Team |
| **SLA** (Agreement) | Customer contract with penalties | 99.5% or refund | Business |

**Rule**: SLA < SLO (always keep a margin)
- Public SLA: 99.5%
- Internal SLO: 99.9%
- Margin = Error Budget to innovate

---

### 3. Error Budget

If your SLO is 99.9%, you're allowed 0.1% errors per month. That's your "budget" for:
- Deploying risky features
- Maintenance windows
- Accepting incidents

```
Error Budget (30 days) = 30 × 24 × 60 × 0.001 = 43.2 minutes downtime allowed
```

| SLO | Downtime/month | Downtime/year |
|-----|----------------|---------------|
| 99% | 7.3h | 3.65 days |
| 99.9% | 43min | 8.7h |
| 99.99% | 4.3min | 52min |
| 99.999% | 26sec | 5min |

---

### 4. Metrics by Service Type

#### Reverse Proxy (Traefik/Nginx)
| Signal | Metric | Alert if |
|--------|--------|----------|
| Latency | `traefik_entrypoint_request_duration_seconds` p99 | > 500ms |
| Traffic | `rate(traefik_entrypoint_requests_total[5m])` | ±50% baseline |
| Errors | `traefik_entrypoint_requests_total{code=~"5.."}` | > 1% |
| Saturation | Open connections | > 80% max |

#### Database (PostgreSQL)
| Signal | Metric | Alert if |
|--------|--------|----------|
| Latency | Query duration p99 | > 100ms |
| Traffic | `rate(pg_stat_database_xact_commit[5m])` | baseline |
| Errors | Deadlocks, rollbacks | > 0 |
| Saturation | Connections, Cache Hit < 95% | > 80% max |

#### Cache (Redis)
| Signal | Metric | Alert if |
|--------|--------|----------|
| Latency | Command latency | > 1ms |
| Traffic | `rate(redis_commands_total[5m])` | baseline |
| Errors | Evicted keys (if critical) | > 0 |
| Saturation | Memory, Hit Rate < 90% | > 80% |

#### Containers (Docker)
| Signal | Metric | Alert if |
|--------|--------|----------|
| Latency | N/A (not a service) | - |
| Traffic | Network I/O | baseline |
| Errors | Restart count | > 0 in 5min |
| Saturation | CPU > 80%, Memory > 85% | limit |

---

## 💻 Minimal Example

### PromQL Queries for Golden Signals
```promql
# LATENCY - p99 response time
histogram_quantile(0.99,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)

# TRAFFIC - Requests per second
sum(rate(http_request_duration_seconds_count[5m]))

# ERRORS - Error rate percentage
sum(rate(http_request_duration_seconds_count{status=~"5.."}[5m]))
/ sum(rate(http_request_duration_seconds_count[5m])) * 100

# SATURATION - CPU usage
100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### Alerting Rules (prometheus/rules.yml)
```yaml
groups:
  - name: golden-signals
    rules:
      - alert: HighLatency
        expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le)) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "API latency p99 > 500ms"

      - alert: HighErrorRate
        expr: |
          sum(rate(http_request_duration_seconds_count{status=~"5.."}[5m]))
          / sum(rate(http_request_duration_seconds_count[5m])) > 0.01
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Error rate > 1%"
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Alerting on causes, not symptoms

**What I did wrong**: Alert on CPU high (cause) instead of latency high (symptom)
**Fix**: Alert on what users experience (latency, errors), not internal metrics
**Lesson**: Users don't see CPU, they see slowness

### Pitfall 2: SLO too ambitious

**What I did wrong**: Set 99.99% SLO (4.3 min downtime/month) for a startup
**Fix**: Start with 99.9% (43 min), increase when you have redundancy
**Lesson**: 99.99% requires multi-region, 99.9% is achievable with single region

### Pitfall 3: Wrong label selectors in Grafana

**What I did wrong**: `instance="$instance"` (exact match)
**Fix**: `instance=~"$instance"` (regex match)
**Lesson**: Always use `=~` with Grafana variables

---

## 📊 Stats

```yaml
Total time: 6h (60% assisted / 40% autonomous)
Used in: [[2024-XX-transcendence-monitoring]], [[2025-12-glasck-deployment]]
```

---

**Last update**: 2025-01-07
**Next review**: 2025-02-07
