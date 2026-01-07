# Alerting Best Practices - Monitoring

## 📋 Metadata

```yaml
tags: [concept, monitoring, alerting, sre, status/learned]
created: 2025-01-07
difficulty: ⭐⭐⭐ (3/5)
time-to-master: 4h
```

**Prerequisites**: [[golden-signals-slo]], [[prometheus-grafana-basics]]
**Related to**: [[elk-stack-basics]]

---

## 🎯 TL;DR (30 seconds)

A good alert wakes you up only when a **user is suffering** (symptom), not when an internal metric changes (cause). Every alert must be **actionable** (you know what to do) and have a **runbook**. Otherwise, it's noise leading to alert fatigue.

**Analogy**: A good alert is like a smoke detector: it rings when there's a real fire, not when you're cooking steak. If you disable it because it rings too often, you die in a fire.

---

## 🤔 When to Use?

### ✅ Good for
1. **Defining Prometheus/Alertmanager alerts**: Criteria for what to alert on
2. **Reducing alert fatigue**: Cleaning up noisy existing alerts
3. **Incident response**: Knowing how to react when it rings

### ❌ Bad for
- **Passive monitoring** → Use dashboards for exploration
- **Audit logs** → That's logging, not alerting

---

## 📚 Key Concepts

### 1. Symptoms vs Causes

Alert on what the **user experiences** (symptom), not internal metrics (cause).

| Type | Example | Alert? |
|------|---------|--------|
| **Symptom** | Latency p99 > 500ms | ✅ YES |
| **Symptom** | Error rate > 1% | ✅ YES |
| **Cause** | CPU > 80% | ❌ NO (unless critical) |
| **Cause** | Memory > 70% | ❌ NO |
| **Cause** | Disk I/O high | ❌ NO |

**Why**: CPU can be at 90% without users suffering. Conversely, CPU at 50% with a memory leak can crash everything.

```
[Internal Causes]     →     [User Symptoms]
CPU, Memory, Disk     →     Latency, Errors, Timeouts
      ↓                            ↓
   Dashboard                    Alerts
```

---

### 2. The 3 Severity Levels

| Severity | Action | Delay | Example |
|----------|--------|-------|---------|
| **🔴 CRITICAL** | Page immediately (PagerDuty) | < 5 min | Service down, data loss |
| **🟠 WARNING** | Slack + ticket | < 4h | Degradation, 80% threshold |
| **🟢 INFO** | Log only | Next business day | Minor anomalies |

**Golden rule**: If an alert wakes you up, you must be able to do something **immediately**. Otherwise, it's a WARNING.

---

### 3. Good Alert Characteristics

| Criteria | Question | ❌ Bad | ✅ Good |
|----------|----------|--------|---------|
| **Actionable** | Do I know what to do? | "Memory high" | "Memory > 90%, restart pod X" |
| **Symptom-based** | Is user suffering? | "CPU spike" | "Latency p99 > 1s" |
| **Rare** | < 5 critical/week? | Rings 10x/day | Rings 1x/month |
| **Documented** | Runbook exists? | Just a title | Link to procedure |
| **Tested** | Did I trigger it voluntarily? | Never tested | Tested in staging |

---

### 4. Prometheus Alert Structure

```yaml
groups:
  - name: api-golden-signals
    rules:
      - alert: APIHighLatency              # Clear, specific name
        expr: |
          histogram_quantile(0.99,
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 0.5
        for: 5m                            # Avoid false positives
        labels:
          severity: warning                # Not critical for 500ms
          service: api
        annotations:
          summary: "API latency p99 > 500ms ({{ $value | humanizeDuration }})"
          runbook_url: "https://wiki.example.com/runbooks/api-latency"
          dashboard: "https://grafana.example.com/d/api-overview"
```

**Essential elements**:
- `for: 5m` → Avoids alerts on 30-second spikes
- `severity` → Determines notification channel
- `runbook_url` → **MANDATORY** for any critical alert
- `dashboard` → Direct link to investigate

---

### 5. Runbook Template

Every CRITICAL alert must have a runbook:

```markdown
# Runbook: APIHighLatency

## Symptom
API responding slowly (p99 > 500ms)

## Impact
Users see timeouts, degraded UX

## Quick Diagnosis (< 2 min)
1. Check dashboard: [link]
2. Check logs: `docker logs api --tail 100`
3. Check dependencies: DB, Redis, external APIs

## Common Causes
| Cause | Check | Quick Fix |
|-------|-------|-----------|
| Slow DB | `pg_stat_activity` | Kill slow queries |
| Redis down | `redis-cli ping` | Restart Redis |
| Memory leak | `docker stats` | Restart API pod |

## Escalation
If not resolved in 15 min → @oncall-lead
```

---

## ⚠️ Pitfalls Experienced

### Pitfall 1: Alerting on everything "just in case"

**What I did wrong**: 50 alerts/day, team ignores everything, real outage missed
**Fix**: Less alerts = more attention on the ones that matter
**Lesson**: If you ignore an alert, delete it or downgrade it

### Pitfall 2: No `for` duration

**What I did wrong**: Alert fires then resolves in 30 seconds (no `for:` clause)
**Fix**: Always `for: 5m` minimum to avoid spikes
**Lesson**: Transient spikes are normal, sustained issues are problems

### Pitfall 3: Alerts without runbook

**What I did wrong**: Alert rings at 3am, nobody knows what to do
**Fix**: No runbook = no critical alert
**Lesson**: Write the runbook BEFORE enabling the alert

---

## 📊 Stats

```yaml
Total time: 4h (50% assisted / 50% autonomous)
Used in: [[2024-XX-transcendence-monitoring]], [[2025-12-glasck-deployment]]
```

---

**Last update**: 2025-01-07
**Next review**: 2025-02-07
