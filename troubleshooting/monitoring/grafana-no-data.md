# Grafana "No Data" - Quick Fix

## 📋 Metadata

```yaml
tags: [troubleshooting, grafana, prometheus, monitoring, status/documented]
created: 2025-01-07
updated: 2025-01-07
severity: 🟠 High
time-to-fix: 10min
```

**Related concept**: [[concepts/monitoring/prometheus-grafana-basics]]
**Affected versions**: Grafana 9.x+, Prometheus 2.x+

---

## 🔍 Symptom

> **Panel shows "No data" when metrics should exist in Prometheus.**
>
> Dashboard displays empty panels despite the exporter running and Prometheus scraping.

```
┌─────────────────────────┐
│                         │
│        No data          │
│                         │
└─────────────────────────┘
```

**Common scenarios where this occurs**:
- New dashboard imported/created
- After changing template variables
- Dashboard copied from Kubernetes to Docker environment
- After modifying Prometheus config

---

## 🎯 Root Cause

> PromQL selector doesn't match actual metric labels in Prometheus.

**Why this happens** (by frequency):

| # | Cause | Frequency |
|---|-------|-----------|
| 1 | `=` instead of `=~` for variables | 40% |
| 2 | Bad template variable query | 25% |
| 3 | Metric name doesn't exist | 15% |
| 4 | Wrong datasource selected | 10% |
| 5 | Prometheus not scraping target | 5% |

---

## ✅ Quick Fix

### Solution 1: Fix selector `=` → `=~` (40% of cases)

```promql
# ❌ Wrong - exact match with Grafana variable
metric{instance="$instance"}

# ✅ Correct - regex match
metric{instance=~"$instance"}
```

**How to fix**: Panel Edit → Query → Replace all `="$variable"` with `=~"$variable"`

**Explanation**: `=` does exact match, `=~` does regex match. Grafana variables can contain regex or `.*` for "All".

---

### Solution 2: Fix template variable (25% of cases)

Dashboard Settings → Variables → Check query returns values

```promql
# ❌ K8s variables on Docker (returns nothing)
label_values(redis_up{namespace="$namespace", pod="$pod"}, instance)

# ✅ Simplified for Docker
label_values(redis_up, instance)
```

**How to fix**: Remove unused K8s variables (namespace, pod) for Docker setups.

---

### Solution 3: Verify metric exists (15% of cases)

```bash
# List available metrics
curl -s "http://prometheus:9090/api/v1/label/__name__/values" | jq '.data[]' | grep redis

# Test specific metric
curl -s "http://prometheus:9090/api/v1/query?query=redis_up"
```

**If metric doesn't exist**: Check exporter is deployed and Prometheus is scraping it.

---

### Solution 4: Check datasource (10% of cases)

Panel Edit → Query → Verify datasource dropdown shows correct Prometheus instance.

---

### Solution 5: Verify Prometheus scraping (5% of cases)

```bash
curl -s "http://prometheus:9090/api/v1/targets" | jq '.data.activeTargets[] | {job, health}'
```

**If target DOWN**: Check exporter is running and on same Docker network.

---

## 🔬 Diagnosis Steps

**If quick fixes don't work, debug with**:

```bash
# 1. Query Inspector in Grafana
# Panel Edit → "i" icon → See exact query sent

# 2. Test query in Prometheus UI
# Copy query → http://prometheus:9090/graph

# 3. Check actual labels on metric
curl "http://prometheus:9090/api/v1/query?query=metric_name" | jq '.data.result[0].metric'
```

---

## 🛡️ Prevention

> Always use regex matchers with Grafana variables and test queries in Prometheus first.

**Best practices**:
- **Always use `=~`** with Grafana variables, never `=`
- **Test queries in Prometheus UI** before adding to Grafana
- **Remove K8s variables** (namespace, pod) for Docker setups
- **Verify targets** after any prometheus.yml change: `curl -X POST http://prometheus:9090/-/reload`

---

## 🔗 Related Issues

- [[troubleshooting/docker/disk-full-logs]] - If Prometheus runs out of disk space

**For deep understanding**: See [[concepts/monitoring/prometheus-grafana-basics]]

---

## 📊 Stats

```yaml
Encountered: 5+ times
Last occurrence: 2025-01-07
Typical fix time: 5-10min
Projects affected: [[projects/2025-12-glasck-deployment]]
```

---

**Quick reference**:
```promql
# ❌ Wrong (exact match)
metric{instance="$instance"}

# ✅ Correct (regex match)
metric{instance=~"$instance"}
```

```bash
# Debug: check what Prometheus has
curl -s "http://prometheus:9090/api/v1/query?query=up" | jq '.data.result'
```

---

**Last update**: 2025-01-07
