# ELK Stack - Elasticsearch, Logstash, Kibana

## 📋 Metadata

```yaml
tags: [concept, monitoring, elk, elasticsearch, logstash, kibana, logs, status/practiced]
created: 2025-12-23
updated: 2025-12-23
difficulty: ⭐⭐⭐⭐ (4/5)
time-to-master: 10h
```

**Prerequisites**: [[docker-compose]], [[logging-basics]]
**Related to**: [[prometheus-grafana]], [[distributed-tracing]]

---

## 🎯 TL;DR (30 seconds)

ELK Stack is a log aggregation and analysis platform:
- **Elasticsearch**: Stores and indexes logs (searchable database)
- **Logstash**: Collects, transforms, and forwards logs to Elasticsearch
- **Kibana**: Visualizes logs with dashboards and search UI

**Analogy**: Library system - Logstash is the librarian (organizes books), Elasticsearch is the catalog (searchable index), Kibana is the search interface (how you find books).

---

## 💻 Key Implementation from Transcendence

```yaml
# Elasticsearch - Search engine and storage
elasticsearch:
  image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
  environment:
    - discovery.type=single-node
    - xpack.security.enabled=true
    - ES_JAVA_OPTS=-Xms512m -Xmx512m  # JVM heap size
    - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
    - xpack.security.http.ssl.enabled=false  # Disable SSL for dev
  ports:
    - "9200:9200"
  volumes:
    - elasticsearch_data:/usr/share/elasticsearch/data
  networks:
    - app-network

# Logstash - Log processor
logstash:
  image: docker.elastic.co/logstash/logstash:8.11.0
  ports:
    - "5044:5044"  # Receive logs from applications
  volumes:
    - ./logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf
  depends_on:
    - elasticsearch
  environment:
    - LS_JAVA_OPTS=-Xms256m -Xmx256m
    - ELASTIC_USER=${ELASTIC_USER}
    - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
  networks:
    - app-network

# Kibana - Visualization UI
kibana:
  build:
    context: ./kibana
    dockerfile: Dockerfile
  ports:
    - "5601:5601"
  environment:
    - ELASTIC_URL=http://elasticsearch:9200
    - xpack.security.enabled=true
  volumes:
    - ./kibana/kibana.yml:/usr/share/kibana/config/kibana.yml:ro
    - kibana_data:/usr/share/kibana/data
  depends_on:
    - elasticsearch
  networks:
    - app-network

# Log cleanup cron job
log-cleanup:
  image: debian:bookworm-slim
  depends_on:
    - elasticsearch
  volumes:
    - ./scripts/logs-cleanup.sh:/usr/local/bin/logs-cleanup.sh:ro
    - ./scripts/logs-cleanup:/etc/cron.d/logs-cleanup:ro
  command: ["sh", "-c", "/usr/local/bin/entrypoint.sh"]
  environment:
    - ELASTIC_URL=${ELASTIC_URL}
    - ELASTIC_USER=${ELASTIC_USER}
    - ELASTIC_PASSWORD=${ELASTIC_PASSWORD}
  networks:
    - app-network
```

---

## ⚠️ Key Pitfalls from Experience

### 1. Elasticsearch Memory Requirements
**Issue**: Elasticsearch crashes with "Out of Memory" errors
**Solution**: Set appropriate JVM heap via `ES_JAVA_OPTS`. Rule of thumb: 50% of container memory limit, max 32GB.

### 2. Startup Dependency Chain
**Issue**: Kibana/Logstash start before Elasticsearch ready, fail to connect
**Solution**: Use `depends_on` + healthchecks. Consider init containers or startup delays.

### 3. Security Configuration
**Issue**: xpack.security enabled but SSL misconfigured causes connection failures
**Solution**: For dev, disable SSL (`xpack.security.http.ssl.enabled=false`). For prod, properly configure certificates.

### 4. Log Retention
**Issue**: Elasticsearch disk fills up with old logs
**Solution**: Implement log cleanup cron job or Index Lifecycle Management (ILM) policies.

---

## 📊 Use Case: Application Logging Pipeline

From Transcendence:
1. App sends logs to Logstash via TCP (port 5044)
2. Logstash parses, enriches, and forwards to Elasticsearch
3. Elasticsearch indexes logs for fast searching
4. Kibana provides UI for log search and visualization
5. Cron job deletes logs older than 30 days

---

## 🔧 Essential Commands

```bash
# Check Elasticsearch health
curl -u elastic:password http://localhost:9200/_cluster/health

# List indices
curl -u elastic:password http://localhost:9200/_cat/indices?v

# Delete old index
curl -X DELETE -u elastic:password http://localhost:9200/logs-2023-01-01

# Search logs
curl -u elastic:password http://localhost:9200/logs-*/_search?q=error

# View Logstash pipeline stats
curl http://localhost:9600/_node/stats/pipelines
```

---

## 📊 Learning Timeline

```
From Transcendence (2024): Implemented full ELK stack for app logging
2024-XX: Configured Logstash pipelines (2h)
2024-XX: Kibana dashboards and visualizations (3h)
2024-XX: Log retention and cleanup automation (1h30)
2025-12-23: Documentation extraction
```

### Time Invested
- **Total**: ~10h (Discovery: 3h, Implementation: 5h, Debugging: 2h)
- **Ratio**: 30% assisted, 70% autonomous ✅

---

## 📝 Status

**Current status**: 🟠 Practiced (Real implementation, needs retention review)

---

## 🔗 Resources

- [Elastic Stack Docs](https://www.elastic.co/guide/index.html)
- [[cheatsheet-elk]]
- Project: Transcendence
- Source: [Transcendence/.devcontainer/docker-compose.yml](/home/artberna/abGitHub/Transcendence/.devcontainer/docker-compose.yml)

---

**Last update**: 2025-12-23
