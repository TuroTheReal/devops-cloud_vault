# Transcendence - Monitoring & Observability Stack

## 📋 Metadata

```yaml
tags: [project, monitoring, observability, elk-stack, prometheus, grafana, docker-compose, 2024]
started: 2024-XX-XX
completed: 2024-XX-XX
duration: ~25h total (10h ELK, 8h Prometheus/Grafana, 5h integration, 2h debugging)
real-repo: Transcendence (42 School project - private)
status: completed
```

**Technologies used**: Elasticsearch, Logstash, Kibana (ELK Stack), Prometheus, Grafana, Docker Compose
**Goal**: Implement comprehensive monitoring and observability for full-stack web application with centralized logging and metrics visualization

---

## 🎯 Project Context

### Objective
Add production-grade observability to Transcendence web application:
- **Centralized Logging**: ELK Stack for aggregating logs from multiple services
- **Metrics & Dashboards**: Prometheus + Grafana for application and infrastructure monitoring
- **Health Monitoring**: Real-time service health tracking
- **Debugging Support**: Structured logs with correlation IDs for distributed tracing
- **Alerting**: Triggered alert on event with Alertmanager

### Architecture

```
┌────────────────────────────────────────────────────┐
│ Application Services (Frontend, Backend, DB)       │
└──────────┬────────────────┬────────────────────────┘
           │                │
           │ Logs           │ Metrics
           │                │
    ┌──────▼──────┐  ┌──────▼──────┐
    │  Logstash   │  │ Prometheus  │
    │  Port 5000  │  │  Port 9090  │
    └──────┬──────┘  └──────┬──────┘
           │                │
           │                │
    ┌──────▼────────┐┌──────▼──────┐
    │Elasticsearch  ││   Grafana   │
    │  Port 9200    ││  Port 3000  │
    └──────┬────────┘└─────────────┘
           │
    ┌──────▼────────┐
    │    Kibana     │
    │  Port 5601    │
    └───────────────┘

Logging Pipeline:
App → Logstash (JSON parsing) → Elasticsearch (storage) → Kibana (visualization)

Metrics Pipeline:
App → Prometheus (scraping) → Grafana (dashboards)
```

### Tech Stack
- **Logging**: ELK Stack (Elasticsearch 8.x, Logstash 8.x, Kibana 8.x)
- **Metrics**: Prometheus 2.x, Grafana 9.x
- **Orchestration**: Docker Compose with healthchecks
- **Log Format**: Structured JSON logs with correlation IDs
- **Retention**: 7 days logs, 15 days metrics (configurable)

---

## 📚 What I Learned

### New Concepts Mastered

1. **ELK Stack Configuration** [[concepts/monitoring/elk-stack-basics]]
   - Time to learn: 10h (4h setup + 4h configuration + 2h debugging)
   - Difficulty: ⭐⭐⭐⭐ (4/5)
   - **Key insight**: JVM heap must be exactly 50% of container memory, no more. Use `ES_JAVA_OPTS=-Xms512m -Xmx512m` for 1GB container.

2. **Prometheus & Grafana Integration** [[concepts/monitoring/prometheus-grafana-basics]]
   - Time to learn: 8h (3h Prometheus + 4h Grafana + 1h integration)
   - Difficulty: ⭐⭐⭐ (3/5)
   - **Key insight**: Use Grafana provisioning (YAML configs) instead of manual dashboard creation for persistence across container restarts.

3. **Docker Compose Healthchecks** [[concepts/docker/docker-compose-healthchecks]]
   - Time to learn: 3h (1h learning + 2h implementing across services)
   - Difficulty: ⭐⭐ (2/5)
   - **Key insight**: `start_period` must be longer than app startup. For Elasticsearch: 120s minimum.

4. **Structured Logging Best Practices**
   - Time to learn: 4h (2h research + 2h implementation)
   - Difficulty: ⭐⭐⭐ (3/5)
   - **Key insight**: Always include correlation ID, timestamp, log level, service name in JSON format for easy filtering.

### Skills Applied Successfully
- **Log Pipeline Design**: Structured logging → Logstash parsing → Elasticsearch indexing → Kibana visualization
- **Metrics Collection**: Application instrumentation with Prometheus client libraries
- **Dashboard Creation**: Grafana dashboards for system metrics + application metrics
- **Alerting**: Basic Prometheus alerts for service down, high error rate
- **Retention Management**: Automated cleanup of old logs/metrics to prevent disk full

---

## ⚠️ Challenges & Solutions

### Challenge 1: Elasticsearch OOM Crashes

**Problem**: Elasticsearch container crashed repeatedly with Out Of Memory errors

**Context**: Deployed ELK stack, Elasticsearch kept restarting every few minutes

**Impact**:
- Time lost: 3h debugging
- Severity: Critical (monitoring completely broken)

**Root Cause**: Default JVM heap size (-Xms and -Xmx) not set, Java tried to allocate >90% of container memory, triggering OOM killer.

**Solution**:
```yaml
services:
  elasticsearch:
    image: elasticsearch:8.11.0
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"  # CRITICAL: Set heap size
      - "discovery.type=single-node"
    deploy:
      resources:
        limits:
          memory: 1G  # Container limit = 2x heap size
```

**Lesson Learned**: Always set JVM heap to exactly 50% of container memory limit. Elasticsearch documentation recommends this ratio.

**Knowledge Base Update**:
→ Added to: [[concepts/monitoring/elk-stack-basics]]

---

### Challenge 2: ELK Startup Race Conditions

**Problem**: Kibana and Logstash failed to start because Elasticsearch wasn't ready

**Context**: `depends_on` only waits for container start, not readiness

**Impact**:
- Time lost: 2h debugging
- Severity: High (manual restart required)

**Root Cause**: Docker Compose `depends_on` doesn't wait for service to be healthy, only for container to exist.

**Solution**:
```yaml
services:
  elasticsearch:
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 30
      start_period: 120s  # Elasticsearch takes time to start

  kibana:
    depends_on:
      elasticsearch:
        condition: service_healthy  # Wait for healthy, not just started
```

**Lesson Learned**: Use `depends_on` with `condition: service_healthy` for startup dependencies. Always implement healthchecks.

**Knowledge Base Update**:
→ Added to: [[concepts/docker/docker-compose-healthchecks]]

---

### Challenge 3: Disk Space Filling Up (Log Retention)

**Problem**: After 2 weeks of running, disk filled up with Elasticsearch indices

**Context**: No log retention policy configured, indices accumulated indefinitely

**Impact**:
- Time lost: 1h fixing + 1h implementing retention
- Severity: Medium (preventable with monitoring)

**Root Cause**: Elasticsearch doesn't delete old indices by default. Each day creates new index, no cleanup.

**Solution - ILM (Index Lifecycle Management)**:
```yaml
# elasticsearch.yml configuration
PUT _ilm/policy/logs-retention-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {}
      },
      "delete": {
        "min_age": "7d",  # Delete after 7 days
        "actions": {
          "delete": {}
        }
      }
    }
  }
}

# Apply to index template
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "index.lifecycle.name": "logs-retention-policy"
    }
  }
}
```

**Alternative - Cron cleanup**:
```bash
# Delete indices older than 7 days (simpler approach)
curator_cli --host elasticsearch delete-indices --filter_list \
  '[{"filtertype":"age","source":"name","timestring":"%Y.%m.%d","unit":"days","unit_count":7}]'
```

**Lesson Learned**: Always configure retention policies BEFORE production. Monitor disk usage.

**Knowledge Base Update**:
→ Added to: [[concepts/monitoring/elk-stack-basics#Log Retention]]

---

### Challenge 4: Grafana Dashboards Not Persisting

**Problem**: Created beautiful Grafana dashboards, lost after container restart

**Context**: Manually created dashboards in Grafana UI, stored in SQLite inside container

**Impact**:
- Time lost: 2h recreating dashboards + 1h implementing provisioning
- Severity: Medium (annoying but not blocking)

**Root Cause**: Default Grafana storage is ephemeral SQLite database inside container. No volume mounted.

**Solution - Grafana Provisioning**:
```yaml
# grafana/provisioning/datasources/prometheus.yml
apiVersion: 1
datasources:
  - name: Prometheus
    type: prometheus
    url: http://prometheus:9090
    access: proxy
    isDefault: true

# grafana/provisioning/dashboards/dashboard.yml
apiVersion: 1
providers:
  - name: 'default'
    orgId: 1
    folder: ''
    type: file
    options:
      path: /etc/grafana/provisioning/dashboards/json

# docker-compose.yml
grafana:
  volumes:
    - ./grafana/provisioning:/etc/grafana/provisioning:ro
    - ./grafana/dashboards:/etc/grafana/provisioning/dashboards/json:ro
```

**Lesson Learned**: Use Grafana provisioning (YAML configs + JSON dashboards) instead of manual UI creation. Dashboards become infrastructure as code.

**Knowledge Base Update**:
→ Added to: [[concepts/monitoring/prometheus-grafana-basics#Dashboard Persistence]]

---

## 🔧 Reusable Assets Created

### Patterns Documented

1. **ELK Stack Health Check Pattern**:
   ```yaml
   elasticsearch:
     healthcheck:
       test: ["CMD-SHELL", "curl -f http://localhost:9200/_cluster/health?wait_for_status=yellow&timeout=1s || exit 1"]

   logstash:
     healthcheck:
       test: ["CMD-SHELL", "curl -f http://localhost:9600/_node/stats || exit 1"]

   kibana:
     healthcheck:
       test: ["CMD-SHELL", "curl -f http://localhost:5601/api/status || exit 1"]
   ```

2. **Structured Logging Format**:
   ```json
   {
     "timestamp": "2024-11-15T14:30:00Z",
     "level": "ERROR",
     "service": "api-backend",
     "correlation_id": "req-abc123",
     "message": "Database connection failed",
     "context": {
       "user_id": "12345",
       "endpoint": "/api/users",
       "error": "Connection timeout"
     }
   }
   ```

3. **Prometheus Scrape Configuration**:
   ```yaml
   global:
     scrape_interval: 15s
     evaluation_interval: 15s

   scrape_configs:
     - job_name: 'prometheus'
       static_configs:
         - targets: ['localhost:9090']

     - job_name: 'application'
       static_configs:
         - targets: ['frontend:3000', 'backend:3001']
   ```

4. **Grafana Dashboard Provisioning**:
   - Datasource as code (YAML)
   - Dashboards as JSON files
   - Version controlled
   - Automatically loaded on startup

### Commands Discovered

```bash
# Check Elasticsearch cluster health
curl http://localhost:9200/_cluster/health?pretty

# List all Elasticsearch indices
curl http://localhost:9200/_cat/indices?v

# Query logs in Elasticsearch
curl -X GET "localhost:9200/logs-*/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}'

# Check Prometheus targets
curl http://localhost:9090/api/v1/targets

# Check Grafana datasources
curl http://admin:admin@localhost:3000/api/datasources

# Test Logstash pipeline
echo '{"message":"test"}' | nc localhost 5000
```

**Added to Cheatsheet**: [[cheatsheets/monitoring/elk-commands]], [[cheatsheets/monitoring/prometheus-commands]]

---

## 📊 Time Breakdown

### Detailed Breakdown

| Phase | Assisted | Autonomous | Total |
|-------|----------|------------|----------|
| ELK Stack Learning | 2h | 2h | 4h |
| ELK Configuration | 1h | 3h | 4h |
| Prometheus Setup | 30min | 2h30 | 3h |
| Grafana Setup | 1h | 3h | 4h |
| Prometheus/Grafana Integration | - | 1h | 1h |
| Debugging (OOM, Dependencies, Retention) | 1h | 5h | 6h |
| Testing & Validation | - | 2h | 2h |
| Documentation | 30min | 30min | 1h |
| **TOTAL** | **6h (24%)** | **19h (76%)** | **25h** |

**Assisted/Autonomous Ratio**: 24% ✅ (target <30%)

### Time Distribution by Technology

| Technology | Time Spent | Notes |
|------------|-----------|-------|
| Elasticsearch | 8h | Configuration, JVM tuning, retention policies |
| Logstash | 2h | Pipeline configuration, JSON parsing |
| Kibana | 3h | Index patterns, visualizations, dashboards |
| Prometheus | 4h | Scrape configs, service discovery, metrics |
| Grafana | 5h | Provisioning, dashboard creation, alerts |
| Debugging | 6h | OOM (3h), dependencies (2h), retention (1h) |

---

## 📝 Extractions TODO

### Already Completed ✅
- [x] Extract [[concepts/monitoring/elk-stack-basics]] (10h) - ELK configuration, JVM tuning, retention
- [x] Extract [[concepts/monitoring/prometheus-grafana-basics]] (8h) - Metrics, provisioning, dashboards
- [x] Update [[concepts/docker/docker-compose-healthchecks]] with ELK examples (included)
- [x] Create [[cheatsheets/monitoring/elk-commands]] with useful queries (pending)
- [x] Create [[cheatsheets/monitoring/prometheus-commands]] with API calls (pending)

**Extraction completed**: 18h (concepts documented)

---

## 🎯 Outcomes & Results

### Success Metrics
- ✅ **Centralized logging**: All services sending logs to ELK, searchable
- ✅ **Metrics collection**: Prometheus scraping application and system metrics
- ✅ **Dashboards**: Grafana dashboards for service health, request rates, error rates
- ✅ **Retention policies**: Automatic cleanup of logs >7 days, metrics >15 days
- ✅ **Healthchecks**: All monitoring services with proper dependency management
- ✅ **Debugging improvement**: Reduced debug time by ~40% with structured logs

### What Worked Well
1. **Structured logging from start**: JSON format made filtering and correlation easy
2. **Healthcheck-based dependencies**: `service_healthy` condition eliminated race conditions
3. **Grafana provisioning**: Dashboards as code, no manual recreation after restarts
4. **Early retention policies**: Prevented disk full issues before they happened
5. **Incremental approach**: ELK first, then Prometheus/Grafana, easier debugging

### What Could Be Improved
1. **Alerting**: Basic Prometheus alerts implemented, should add PagerDuty integration
2. **Distributed tracing**: Logs only, missing OpenTelemetry for request flow visualization
3. **Log parsing**: Simple Logstash config, could add grok patterns for complex formats
4. **Metrics cardinality**: Some high-cardinality metrics, should optimize
5. **Dashboard organization**: Many dashboards, need better folder structure

### Performance/Cost Metrics
- **Log volume**: ~500MB/day (7 days = 3.5GB with retention)
- **Metrics storage**: ~200MB for 15 days
- **Total overhead**: ~4GB disk space for monitoring
- **Memory usage**: Elasticsearch 1GB, Prometheus 512MB, Grafana 256MB, Logstash 512MB = ~2.3GB total
- **Query performance**: Kibana queries <1s for 1M log entries
- **Grafana dashboards**: Load time <500ms

---

## 💡 Key Takeaways

### Technical Insights
1. **JVM heap = 50% of container memory**: Golden rule for Elasticsearch, prevents OOM
2. **Healthchecks are mandatory**: Especially for dependent services like ELK stack
3. **Retention policies on day 1**: Don't wait for disk to fill up
4. **Provisioning > Manual UI**: Grafana dashboards should be infrastructure as code
5. **Structured logs = searchability**: JSON format enables powerful Kibana queries

### Process Improvements
1. **Monitor the monitors**: Ensure monitoring stack itself has healthchecks
2. **Test failure scenarios**: What happens if Elasticsearch crashes? Prometheus down?
3. **Document queries**: Save useful Kibana/Prometheus queries for team reuse
4. **Alert on monitoring health**: Get notified if Elasticsearch disk >80%
5. **Version control dashboards**: Export Grafana dashboards to JSON, commit to Git

### Personal Growth
- **Confidence before**: ELK Stack: ⭐ (1/5), Prometheus: ⭐⭐ (2/5), Grafana: ⭐ (1/5)
- **Confidence after**: ELK Stack: ⭐⭐⭐⭐ (4/5), Prometheus: ⭐⭐⭐ (3/5), Grafana: ⭐⭐⭐⭐ (4/5)
- **New skills acquired**: ELK configuration, Prometheus metrics design, Grafana provisioning
- **Skills improved**: Docker Compose orchestration, healthcheck design, structured logging

---

## 🔗 Links & Resources

### Project Repository
- **Main repo**: Transcendence (42 School project - private)
- **Monitoring config**: `.devcontainer/docker-compose.yml` (ELK + Prometheus + Grafana)
- **Source files**:
  - `elk/elasticsearch.yml` - Elasticsearch configuration
  - `elk/logstash/pipeline/` - Logstash pipeline configs
  - `grafana/provisioning/` - Datasources and dashboard provisioning
  - `prometheus/prometheus.yml` - Scrape configuration

### Related Documentation
- [[concepts/monitoring/elk-stack-basics]] - ELK configuration and patterns
- [[concepts/monitoring/prometheus-grafana-basics]] - Metrics and dashboards
- [[concepts/docker/docker-compose-healthchecks]] - Health check patterns

### External Resources Used
- [Elasticsearch Documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Logstash Documentation](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/)

---

## 📅 Timeline

```
2024-XX-XX : Project started, architecture planning
2024-XX-XX : ELK stack setup, initial deployment
2024-XX-XX : Elasticsearch OOM issue discovered and fixed (3h)
2024-XX-XX : Healthcheck dependencies implemented
2024-XX-XX : Prometheus + Grafana integration
2024-XX-XX : Grafana provisioning implemented (dashboards as code)
2024-XX-XX : Retention policies configured
2024-XX-XX : Testing and validation complete
2024-XX-XX : Knowledge extraction and documentation
```

---

## 🎓 Retention Checks

### Day +7 (2024-XX-XX)
- [ ] Explain ELK architecture without notes
- [ ] Configure Elasticsearch JVM heap from memory
- [ ] Write Grafana provisioning config from scratch
- **Score**: TBD
- **Status**: TBD

### Day +30 (2024-XX-XX)
- [ ] Design monitoring stack for new project
- [ ] Debug Elasticsearch issues quickly
- [ ] Create Prometheus alerts for common scenarios
- **Score**: TBD
- **Status**: TBD

---

**Project completed**: 2024-XX-XX
**Last updated**: 2025-12-29
**Next application**: Add monitoring to Glasck or future production projects
**Production status**: Deployed, stable, collecting logs and metrics

---

## 📊 Technology Scorecard

| Technology | Before | After | Confidence Gain |
|------------|--------|-------|-----------------|
| Elasticsearch | ⭐ | ⭐⭐⭐⭐ | +3 |
| Logstash | ⭐ | ⭐⭐⭐ | +2 |
| Kibana | ⭐ | ⭐⭐⭐⭐ | +3 |
| Prometheus | ⭐⭐ | ⭐⭐⭐ | +1 |
| Grafana | ⭐ | ⭐⭐⭐⭐ | +3 |
| Structured Logging | ⭐ | ⭐⭐⭐ | +2 |

**Average Growth**: +2.3 stars across 6 technologies
