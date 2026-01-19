# LogQL Commands Cheatsheet

## 📋 Metadata

```yaml
tags: [cheatsheet, loki, logql, grafana, monitoring]
created: 2026-01-19
related: [[concepts/monitoring/loki-alloy-basics]]
```

---

## 🎯 Quick Reference

```logql
{label="value"}                    # Basic label filter
{label=~"regex"}                   # Regex label match
{label!="value"}                   # Not equal
{label!~"regex"}                   # Not regex match
```

---

## 📍 Label Selectors

### Basic Selection
```logql
# Exact match
{container_name="api-fastify-prod"}

# Multiple labels (AND)
{container_name="api-fastify-prod", compose_service="api"}

# Regex match
{container_name=~"api-.*"}

# Not equal
{container_name!="prometheus"}

# Not regex
{container_name!~".*exporter.*"}
```

### Common Patterns
```logql
# All containers
{container_name=~".+"}

# All compose services
{compose_service=~".+"}

# Specific image
{image=~".*traefik.*"}
```

---

## 🔍 Line Filters

### Contains / Not Contains
```logql
# Contains string (case-sensitive)
{container_name="api"} |= "error"

# Not contains
{container_name="api"} != "healthcheck"

# Case-insensitive contains
{container_name="api"} |~ "(?i)error"

# Regex match
{container_name="api"} |~ "status=[45][0-9]{2}"

# Not regex
{container_name="api"} !~ "GET /health"
```

### Chaining Filters
```logql
# Multiple filters (AND)
{container_name="api"} |= "error" != "healthcheck" != "metrics"

# Find errors excluding noise
{container_name="api"} |= "error" != "error_count" != "error_handler"
```

---

## 🔧 Parsers

### JSON Parser
```logql
# Parse JSON and access fields
{container_name="api"} | json

# Filter by JSON field
{container_name="api"} | json | level="error"

# Multiple field filters
{container_name="api"} | json | level="error" | status >= 500

# Extract specific fields
{container_name="api"} | json | line_format "{{.level}}: {{.msg}}"
```

### Logfmt Parser
```logql
# Parse key=value format
{container_name="app"} | logfmt

# Filter parsed fields
{container_name="app"} | logfmt | level="error"
```

### Regex Parser
```logql
# Extract with named groups
{container_name="nginx"} | regexp `(?P<ip>\d+\.\d+\.\d+\.\d+) .* "(?P<method>\w+) (?P<path>[^"]+)"`

# Use extracted fields
{container_name="nginx"} | regexp `(?P<status>\d{3})` | status >= 400
```

### Pattern Parser (simpler than regex)
```logql
# Pattern with placeholders
{container_name="nginx"} | pattern `<ip> - - [<_>] "<method> <path> <_>" <status>`

# Filter on extracted
{container_name="nginx"} | pattern `<_> <status>` | status >= 400
```

---

## 📊 Metric Queries

### Count Logs
```logql
# Count logs per second
count_over_time({container_name="api"} |= "error" [5m])

# Rate of logs
rate({container_name="api"} |= "error" [5m])

# Logs per container
sum by (container_name) (count_over_time({container_name=~".+"} [5m]))
```

### Bytes
```logql
# Bytes per container
sum by (container_name) (bytes_over_time({container_name=~".+"} [1h]))

# Rate of bytes
bytes_rate({container_name="api"} [5m])
```

### Aggregations
```logql
# Top 5 containers by log volume
topk(5, sum by (container_name) (bytes_over_time({container_name=~".+"} [1h])))

# Average per minute
avg_over_time(
  count_over_time({container_name="api"} |= "error" [1m])
[5m])
```

---

## ⚡ Useful One-Liners

### Error Detection
```logql
# All errors (JSON logs)
{container_name=~".+"} | json | level="error"

# HTTP 5xx errors
{container_name="traefik"} | json | status >= 500

# Errors with context
{container_name="api"} |= "error" | json | line_format "{{.time}} [{{.level}}] {{.msg}}"
```

### Performance
```logql
# Slow requests (>1s)
{container_name="api"} | json | duration > 1000

# Requests by status code
sum by (status) (count_over_time({container_name="traefik"} | json [5m]))
```

### Debug
```logql
# Last 100 lines from container
{container_name="api"} | limit 100

# Logs around timestamp
{container_name="api"} |= "error"  # then set time range in Grafana

# Specific request ID
{container_name=~".+"} |= "req-abc123"
```

### Traefik Specific
```logql
# Failed requests
{container_name="reverse-proxy"} | json | DownstreamStatus >= 400

# Requests to specific path
{container_name="reverse-proxy"} | json | RequestPath=~"/api/.*"

# Slow responses
{container_name="reverse-proxy"} | json | Duration > 1000000000
```

---

## 🚫 Common Mistakes

### ❌ Wrong
```logql
# No label selector
|= "error"

# Wrong operator order
{app="api"} | json |= "error"  # Filter BEFORE parser

# Case-sensitive when shouldn't be
{app="api"} |= "Error"  # Misses "error", "ERROR"
```

### ✅ Correct
```logql
# Always start with label
{container_name=~".+"} |= "error"

# Filter after parser for structured logs
{app="api"} | json | level="error"

# Case-insensitive
{app="api"} |~ "(?i)error"
```

---

## 🔗 Operators Reference

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Exact match | `{app="api"}` |
| `!=` | Not equal | `{app!="test"}` |
| `=~` | Regex match | `{app=~"api-.*"}` |
| `!~` | Not regex | `{app!~".*test.*"}` |
| `\|=` | Contains | `\|= "error"` |
| `!=` | Not contains | `!= "health"` |
| `\|~` | Regex filter | `\|~ "error\|warn"` |
| `!~` | Not regex filter | `!~ "debug"` |
| `\| json` | Parse JSON | `\| json` |
| `\| logfmt` | Parse logfmt | `\| logfmt` |
| `\| line_format` | Format output | `\| line_format "{{.msg}}"` |

---

## 📚 Resources

- [LogQL Documentation](https://grafana.com/docs/loki/latest/query/)
- [[concepts/monitoring/loki-alloy-basics]] - Loki & Alloy fundamentals

---

**Last update**: 2026-01-19
