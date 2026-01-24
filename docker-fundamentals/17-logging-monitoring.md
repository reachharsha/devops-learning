---
render_with_liquid: false
---
# 17 - Logging & Monitoring

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Configure Docker logging drivers
- Implement centralized logging
- Monitor container health and metrics
- Set up log aggregation with ELK stack
- Use Prometheus and Grafana for monitoring
- Debug production issues with logs
- Implement alerting systems

---

## 📝 Docker Logging Drivers

### **Available Logging Drivers:**
```bash
# View available drivers
docker info --format '{ {.Plugins.Log} }'

# Common drivers:
# - json-file (default)
# - syslog
# - journald
# - gelf (Graylog)
# - fluentd
# - awslogs (CloudWatch)
# - splunk
# - gcplogs (Google Cloud)
```

### **Configure Logging Driver:**
```bash
# JSON file (default)
docker run -d \
  --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  --log-opt labels=env,app \
  nginx

# Syslog
docker run -d \
  --log-driver syslog \
  --log-opt syslog-address=tcp://192.168.0.42:514 \
  --log-opt tag="myapp" \
  nginx

# Fluentd
docker run -d \
  --log-driver fluentd \
  --log-opt fluentd-address=localhost:24224 \
  --log-opt tag=myapp.logs \
  nginx

# Disable logging (not recommended!)
docker run -d \
  --log-driver none \
  nginx
```

### **Default Logging in daemon.json:**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3",
    "labels": "service,environment"
  }
}
```

---

## 📊 Viewing Logs

### **Basic Log Commands:**
```bash
# View all logs
docker logs container

# Follow logs (tail -f)
docker logs -f container

# Show timestamps
docker logs -t container

# Show last N lines
docker logs --tail 100 container

# Show logs since timestamp
docker logs --since 2024-01-01T00:00:00 container

# Show logs until timestamp
docker logs --until 2024-01-01T23:59:59 container

# Show logs since duration
docker logs --since 5m container
docker logs --since 2h container

# Combine options
docker logs -f --tail 50 --since 10m container
```

### **Compose Logs:**
```bash
# All services
docker-compose logs

# Specific service
docker-compose logs app

# Follow logs
docker-compose logs -f

# Last N lines
docker-compose logs --tail=100

# Multiple services
docker-compose logs app nginx
```

---

## 🏗️ Centralized Logging Architecture

### **ELK Stack (Elasticsearch, Logstash, Kibana):**
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Application
  app:
    image: myapp:latest
    depends_on:
      - fluentd
    logging:
      driver: fluentd
      options:
        fluentd-address: localhost:24224
        tag: myapp.logs

  # Fluentd - Log collector
  fluentd:
    build: ./fluentd
    ports:
      - "24224:24224"
    volumes:
      - ./fluentd/conf:/fluentd/etc
    depends_on:
      - elasticsearch

  # Elasticsearch - Log storage
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  # Kibana - Log visualization
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch

volumes:
  es-data:
```

### **Fluentd Configuration:**
```ruby
# fluentd/conf/fluent.conf
<source>
  @type forward
  port 24224
  bind 0.0.0.0
</source>

<filter **>
  @type parser
  key_name log
  <parse>
    @type json
    time_key time
    time_format %Y-%m-%dT%H:%M:%S.%L%z
  </parse>
</filter>

<match **>
  @type elasticsearch
  host elasticsearch
  port 9200
  logstash_format true
  logstash_prefix docker
  include_tag_key true
  tag_key @log_name
  flush_interval 1s
</match>
```

```dockerfile
# fluentd/Dockerfile
FROM fluent/fluentd:v1.16-1

USER root

RUN apk add --no-cache --update --virtual .build-deps \
    sudo build-base ruby-dev \
 && sudo gem install fluent-plugin-elasticsearch \
 && sudo gem sources --clear-all \
 && apk del .build-deps \
 && rm -rf /tmp/* /var/tmp/* /usr/lib/ruby/gems/*/cache/*.gem

USER fluent
```

---

## 📈 Monitoring with Prometheus & Grafana

### **Complete Monitoring Stack:**
```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  # Your application with metrics endpoint
  app:
    image: myapp:latest
    ports:
      - "3000:3000"
    labels:
      - "prometheus-scrape.enabled=true"
      - "prometheus-scrape.port=3000"
      - "prometheus-scrape.path=/metrics"

  # Prometheus - Metrics collection
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/alerts.yml:/etc/prometheus/alerts.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=30d'

  # Node Exporter - Host metrics
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
      - '--path.sysfs=/host/sys'
      - '--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)'

  # cAdvisor - Container metrics
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
      - /dev/disk:/dev/disk:ro
    privileged: true

  # Grafana - Visualization
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources

  # Alertmanager - Alert handling
  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/config.yml:/etc/alertmanager/config.yml
      - alertmanager-data:/alertmanager
    command:
      - '--config.file=/etc/alertmanager/config.yml'
      - '--storage.path=/alertmanager'

volumes:
  prometheus-data:
  grafana-data:
  alertmanager-data:
```

### **Prometheus Configuration:**
```yaml
# prometheus/prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'docker-local'
    environment: 'production'

# Alerting configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Load alert rules
rule_files:
  - '/etc/prometheus/alerts.yml'

scrape_configs:
  # Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']

  # cAdvisor
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']

  # Application
  - job_name: 'myapp'
    static_configs:
      - targets: ['app:3000']
    metrics_path: '/metrics'
```

### **Alert Rules:**
```yaml
# prometheus/alerts.yml
groups:
  - name: container_alerts
    interval: 30s
    rules:
      # Container down
      - alert: ContainerDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Container {{ $labels.instance }} is down"
          description: "{{ $labels.instance }} has been down for more than 1 minute"

      # High memory usage
      - alert: HighMemoryUsage
        expr: (container_memory_usage_bytes / container_spec_memory_limit_bytes) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.name }}"
          description: "{{ $labels.name }} is using {{ $value | humanizePercentage }} of memory"

      # High CPU usage
      - alert: HighCPUUsage
        expr: rate(container_cpu_usage_seconds_total[5m]) > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.name }}"
          description: "{{ $labels.name }} CPU usage is {{ $value | humanizePercentage }}"

      # Disk space low
      - alert: DiskSpaceLow
        expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value | humanizePercentage }} disk space remaining"
```

### **Alertmanager Configuration:**
```yaml
# alertmanager/config.yml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'

route:
  group_by: ['alertname', 'cluster', 'service']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'slack-notifications'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty-critical'
    - match:
        severity: warning
      receiver: 'slack-notifications'

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'pagerduty-critical'
    pagerduty_configs:
      - service_key: 'YOUR_PAGERDUTY_KEY'
        description: '{{ .GroupLabels.alertname }}'
```

---

## 📱 Application Instrumentation

### **Node.js with Prom-Client:**
```javascript
// app.js
const express = require('express');
const promClient = require('prom-client');

const app = express();

// Create metrics registry
const register = new promClient.Registry();

// Default metrics (CPU, memory, etc.)
promClient.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections'
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);

// Middleware to track metrics
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration.labels(req.method, req.route?.path || req.path, res.statusCode).observe(duration);
    httpRequestTotal.labels(req.method, req.route?.path || req.path, res.statusCode).inc();
  });
  
  activeConnections.inc();
  res.on('close', () => activeConnections.dec());
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

---

## 🔍 Structured Logging

### **Winston Logger:**
```javascript
// logger.js
const winston = require('winston');

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: process.env.SERVICE_NAME || 'myapp',
    environment: process.env.NODE_ENV || 'development',
    hostname: require('os').hostname(),
    version: process.env.APP_VERSION || '1.0.0'
  },
  transports: [
    // Console output
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.printf(({ level, message, timestamp, ...meta }) => {
          return `${timestamp} [${level}]: ${message} ${
            Object.keys(meta).length ? JSON.stringify(meta, null, 2) : ''
          }`;
        })
      )
    })
  ]
});

module.exports = logger;

// Usage
const logger = require('./logger');

logger.info('Server started', { port: 3000 });
logger.error('Database connection failed', { 
  error: err.message, 
  stack: err.stack 
});
logger.warn('High memory usage', { 
  usage: process.memoryUsage() 
});
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the default Docker logging driver?**
   <details>
   <summary>Answer</summary>
   json-file
   </details>

2. **How do you follow container logs in real-time?**
   <details>
   <summary>Answer</summary>
   docker logs -f container
   </details>

3. **What's the ELK stack?**
   <details>
   <summary>Answer</summary>
   Elasticsearch (storage), Logstash (processing), Kibana (visualization)
   </details>

4. **What does Prometheus do?**
   <details>
   <summary>Answer</summary>
   Collects and stores time-series metrics data
   </details>

5. **Why use structured logging?**
   <details>
   <summary>Answer</summary>
   Easier to parse, search, filter, and analyze logs programmatically
   </details>

---

## 📝 Hands-on Exercise

**Set up complete monitoring:**

```bash
mkdir monitoring-lab
cd monitoring-lab

# 1. Create app with metrics
mkdir app
cat > app/package.json << 'EOF'
{
  "name": "monitored-app",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0",
    "prom-client": "^15.0.0"
  }
}
EOF

cat > app/server.js << 'EOF'
const express = require('express');
const promClient = require('prom-client');
const app = express();
const register = new promClient.Registry();

promClient.collectDefaultMetrics({ register });

const httpRequestsTotal = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
  labelNames: ['method', 'path', 'status']
});
register.registerMetric(httpRequestsTotal);

app.use((req, res, next) => {
  res.on('finish', () => {
    httpRequestsTotal.labels(req.method, req.path, res.statusCode).inc();
  });
  next();
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.get('/', (req, res) => res.send('Hello!'));
app.get('/health', (req, res) => res.json({ status: 'ok' }));

app.listen(3000, () => console.log('Server on 3000'));
EOF

cat > app/Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY server.js .
CMD ["node", "server.js"]
EOF

# 2. Create Prometheus config
mkdir prometheus
cat > prometheus/prometheus.yml << 'EOF'
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'app'
    static_configs:
      - targets: ['app:3000']
EOF

# 3. Create docker-compose
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  app:
    build: ./app
    ports:
      - "3000:3000"

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
EOF

# 4. Start stack
docker-compose up -d

# 5. Generate traffic
for i in {1..100}; do curl http://localhost:3000; done

# 6. View metrics
echo "App metrics: http://localhost:3000/metrics"
echo "Prometheus: http://localhost:9090"
echo "Grafana: http://localhost:3001 (admin/admin)"

# 7. Cleanup
cd ..
rm -rf monitoring-lab
```

---

## 📝 Key Takeaways

✅ **Configure logging drivers appropriately**  
✅ **Use centralized logging in production**  
✅ **Implement structured logging**  
✅ **Monitor containers with Prometheus**  
✅ **Visualize metrics with Grafana**  
✅ **Set up alerting for critical issues**  
✅ **Expose application metrics**  
✅ **Rotate and limit log files**  
✅ **Use labels and tags for filtering**  
✅ **Monitor resource usage continuously**  

---

## 🚀 Next Steps

**Scale and orchestrate:**

**Next lesson:** [18 - Docker Swarm Basics](18-docker-swarm-basics.md) - Container orchestration

You can't manage what you don't measure! 📊
