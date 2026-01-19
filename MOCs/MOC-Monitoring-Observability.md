# MOC - Monitoring & Observability

> **What**: Complete monitoring path from basic metrics to production SLOs.
>
> **Who**: DevOps/SRE beginner to intermediate with Docker environment.
>
> **Why**: Build production-ready monitoring and know how to respond to incidents.

---

## 🎯 Learning Objective

Master the 3 pillars of observability (Metrics, Logs, Traces) with focus on Prometheus/Grafana metrics. You'll be able to deploy a complete monitoring stack, create relevant dashboards, and define alerts that don't generate noise.

**Real-world use case**: Detect performance degradation before users complain, quickly diagnose root cause, and prevent recurrence.

---

## 📚 Learning Path

### Phase 1: Fundamentals (6h)

**Goal**: Understand basics and deploy a working Prometheus/Grafana stack.

1. **[[concepts/monitoring/prometheus-grafana-basics]]** ⭐⭐ (4h)
   - **Why now**: Essential technical foundation - how to collect and visualize metrics
   - **You'll learn**: Prometheus architecture, metric types, PromQL basics, Grafana provisioning

2. **[[cheatsheets/prometheus/promql-commands]]** ⭐⭐ (2h)
   - **Why now**: Quick reference for queries - you'll need this constantly
   - **You'll learn**: PromQL syntax, rate/sum/histogram_quantile functions, common patterns

**✅ Checkpoint**: You can deploy Prometheus + Grafana in Docker and create a basic dashboard with node-exporter.

---

### Phase 2: Production Monitoring (8h)

**Goal**: Know what to monitor and how to structure dashboards.

3. **[[concepts/monitoring/golden-signals-slo]]** ⭐⭐⭐ (6h)
   - **Why now**: Now that you can collect metrics, learn which ones matter
   - **You'll learn**: 4 Golden Signals, SLI/SLO/SLA, Error Budget, metrics by service type

4. **[[concepts/monitoring/alerting-best-practices]]** ⭐⭐⭐ (4h)
   - **Why now**: Dashboards are nice, but alerts wake you when things break
   - **You'll learn**: Alert on symptoms vs causes, avoid alert fatigue, runbooks

**✅ Checkpoint**: You have a Golden Signals dashboard for each service and alerts configured with Alertmanager.

---

### Phase 3: Logging (6h)

**Goal**: Complement metrics with logs for debugging.

**Choose your stack**:

| Stack | Components | Complexity | Best for |
|-------|------------|------------|----------|
| **Loki** (Recommended) | Alloy + Loki + Grafana | ⭐⭐ Light | Small/medium, already using Grafana |
| **ELK** | Elasticsearch + Logstash + Kibana | ⭐⭐⭐⭐ Heavy | Large scale, full-text search, analytics |

#### Option A: Loki + Alloy Stack (Recommended for starters, or small budget/space)

5a. **[[concepts/monitoring/loki-alloy-basics]]** ⭐⭐⭐ (6h)
   - **Why Loki**: Same visualizer as metrics (Grafana), lighter than ELK, label-based like Prometheus
   - **You'll learn**: Alloy collector, Loki storage, LogQL queries
   - **Cheatsheet**: [[cheatsheets/loki/logql-commands]]

**PLG vs ELK comparison**:
```
PLG (Promtail + Loki + Grafana)          ELK (Elasticsearch + Logstash + Kibana)
├── Lightweight (~500MB RAM)              ├── Heavy (~4GB+ RAM)
├── Label-based indexing                  ├── Full-text indexing
├── LogQL (similar to PromQL)             ├── KQL/Lucene queries
├── Single UI: Grafana                    ├── Separate UI: Kibana
├── Good for: grep-style queries          ├── Good for: analytics, aggregations
└── Cost: Low storage                     └── Cost: High storage (indexes everything)
```

#### Option B: ELK Stack (For scale)

5b. **[[concepts/monitoring/elk-stack-basics]]** ⭐⭐⭐ (6h)
   - **Why ELK**: Full-text search, powerful analytics, industry standard
   - **You'll learn**: Elasticsearch, Logstash, Kibana, log aggregation

**✅ Checkpoint**: You can correlate a Prometheus alert with corresponding logs (in Grafana Explore or Kibana).

---

### Phase 4: Troubleshooting (2h)

**Goal**: Know how to react when monitoring itself has problems.

6. **[[troubleshooting/monitoring/grafana-no-data]]** ⭐⭐ (1h)
   - **Why now**: "No data" is the #1 problem you'll encounter
   - **You'll learn**: Debug selectors, Grafana variables, check Prometheus targets

7. **[[troubleshooting/docker/disk-full-logs]]** ⭐⭐ (1h)
   - **Why now**: Logs can fill disk and crash the whole monitoring stack
   - **You'll learn**: Log rotation, Docker limits, cleanup

8. **[[troubleshooting/docker/restart-config-not-applied]]** ⭐⭐ (30min)
   - **Why now**: Docker restart ≠ config reload - monitoring targets go "down" after VPS reboot
   - **You'll learn**: Container lifecycle, docker-compose up vs restart, network reconnection

**✅ Checkpoint**: You can diagnose and solve common monitoring stack problems.

---

## 📊 Progress Tracking

```yaml
Total estimated time: 22h
Difficulty: ⭐⭐⭐ (3/5)
Prerequisites: [[MOC-Docker-Production]] (containers basics)
```

---

## 🔄 Next Steps

After completing this path:
- **Distributed Tracing**: Jaeger/Tempo to trace requests across microservices
- **Kubernetes Monitoring**: Adapt for kube-prometheus-stack, ServiceMonitors
- **On-Call & Incident Management**: PagerDuty, advanced runbooks, post-mortems

---

## 📖 Quick Reference

**Core concepts**:
1. **Golden Signals**: Latency, Traffic, Errors, Saturation - cover 90% of problems
2. **SLO vs SLA**: SLO = internal target (99.9%), SLA = customer contract (99.5%) - always SLO > SLA
3. **Symptoms vs Causes**: Alert on latency (symptom), not CPU (cause)
4. **Rate required**: Always `rate(counter[5m])`, never raw counter
5. **Grafana variables**: Always `=~` (regex), never `=` (exact match)

**Common pitfalls**:
- Using `=` instead of `=~` with Grafana variables → "No data"
- No `for:` in alerts → False positives on spikes
- SLO too ambitious (99.99%) → Error budget always exhausted
- Alerting on everything "just in case" → Alert fatigue

**Metrics by service**:

| Service | Key Metric #1 | Key Metric #2 | Alert Threshold |
|---------|---------------|---------------|-----------------|
| API | Latency p99 | Error rate | > 500ms, > 1% |
| PostgreSQL | Cache Hit Ratio | Connections | < 95%, > 80% |
| Redis | Hit Rate | Memory | < 90%, > 80% |
| Traefik | Request rate | 5xx rate | baseline, > 1% |
| Docker | CPU/Memory | Restarts | > 85%, > 0 |

---

**Created**: 2025-01-07
**Last updated**: 2026-01-19
