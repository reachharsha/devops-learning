---
render_with_liquid: false
---
# 15 - Production Deployment

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Deploy containers to production safely
- Implement zero-downtime deployments
- Configure health checks and monitoring
- Handle environment-specific configuration
- Implement proper logging
- Use Docker in production orchestrators
- Follow deployment best practices

---

## 🚀 Production Readiness Checklist

### **Before Going to Production:**
```
Infrastructure:
□ Load balancer configured
□ SSL/TLS certificates installed
□ DNS configured
□ Firewall rules set
□ Backup strategy defined

Application:
□ Health checks implemented
□ Graceful shutdown handling
□ Proper error handling
□ Logging configured
□ Metrics exposed

Container:
□ Non-root user configured
□ Security scan passed
□ Resource limits set
□ Secrets externalized
□ Multi-stage build used

Operations:
□ Monitoring set up
□ Alerting configured
□ Rollback plan ready
□ Documentation updated
□ Team trained
```

---

## 🏗️ Production Dockerfile

### **Optimized Production Build:**
```dockerfile
# syntax=docker/dockerfile:1

###################
# Base Stage
###################
FROM node:18-alpine AS base
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*
WORKDIR /app

###################
# Dependencies Stage
###################
FROM base AS dependencies
COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

###################
# Build Stage
###################
FROM base AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build && \
    npm run test

###################
# Production Stage
###################
FROM base AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy dependencies
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy built application
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --from=build --chown=nodejs:nodejs /app/package*.json ./

# Set environment
ENV NODE_ENV=production \
    PORT=3000

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "dist/index.js"]
```

---

## 🎯 Health Checks

### **Application Health Check:**
```javascript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  console.log(`STATUS: ${res.statusCode}`);
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

request.on('error', (err) => {
  console.log('ERROR', err);
  process.exit(1);
});

request.end();
```

```javascript
// Health endpoint in Express
app.get('/health', (req, res) => {
  // Check database
  const dbHealthy = await checkDatabase();
  
  // Check dependencies
  const redisHealthy = await checkRedis();
  
  if (dbHealthy && redisHealthy) {
    res.status(200).json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime()
    });
  } else {
    res.status(503).json({
      status: 'unhealthy',
      database: dbHealthy,
      redis: redisHealthy
    });
  }
});

// Readiness endpoint
app.get('/ready', async (req, res) => {
  // Check if app is ready to serve traffic
  if (await isReady()) {
    res.status(200).send('Ready');
  } else {
    res.status(503).send('Not Ready');
  }
});
```

### **Compose Health Checks:**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

  redis:
    image: redis:alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  app:
    build: .
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 60s
```

---

## 🔄 Graceful Shutdown

### **Signal Handling:**
```javascript
// app.js
const express = require('express');
const app = express();

// Track active connections
let connections = new Set();
let isShuttingDown = false;

const server = app.listen(3000, () => {
  console.log('Server started on port 3000');
});

// Track connections
server.on('connection', (conn) => {
  connections.add(conn);
  conn.on('close', () => {
    connections.delete(conn);
  });
});

// Graceful shutdown
const gracefulShutdown = async (signal) => {
  console.log(`${signal} received, starting graceful shutdown`);
  
  if (isShuttingDown) {
    return;
  }
  isShuttingDown = true;

  // Stop accepting new connections
  server.close(async () => {
    console.log('HTTP server closed');

    try {
      // Close database connections
      await db.close();
      console.log('Database connections closed');

      // Close Redis
      await redis.quit();
      console.log('Redis connection closed');

      // Wait for active requests to complete
      console.log(`Waiting for ${connections.size} connections to close`);
      
      process.exit(0);
    } catch (err) {
      console.error('Error during shutdown:', err);
      process.exit(1);
    }
  });

  // Force shutdown after 30 seconds
  setTimeout(() => {
    console.error('Forced shutdown after timeout');
    process.exit(1);
  }, 30000);

  // Close all active connections
  for (const conn of connections) {
    conn.end();
  }
};

// Handle signals
process.on('SIGTERM', () => gracefulShutdown('SIGTERM'));
process.on('SIGINT', () => gracefulShutdown('SIGINT'));

// Handle uncaught errors
process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err);
  gracefulShutdown('UNCAUGHT_EXCEPTION');
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  gracefulShutdown('UNHANDLED_REJECTION');
});
```

---

## 🔧 Environment Configuration

### **Multi-Environment Setup:**
```yaml
# docker-compose.base.yml
version: '3.8'

services:
  app:
    image: myapp:${VERSION:-latest}
    restart: unless-stopped
    
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
```

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src
    environment:
      NODE_ENV: development
      LOG_LEVEL: debug
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port

  postgres:
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: dev
```

```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    image: registry.company.com/myapp:${VERSION}
    environment:
      NODE_ENV: production
      LOG_LEVEL: info
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    secrets:
      - db_password
      - api_key

  postgres:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres-data-prod:/var/lib/postgresql/data

secrets:
  db_password:
    external: true
  api_key:
    external: true

volumes:
  postgres-data-prod:
    driver: local
```

```bash
# Development
docker-compose -f docker-compose.base.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.base.yml -f docker-compose.prod.yml up -d
```

---

## 📊 Logging

### **Structured Logging:**
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
    service: 'myapp',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION
  },
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    })
  ]
});

module.exports = logger;

// Usage
logger.info('Server started', { port: 3000 });
logger.error('Database connection failed', { error: err.message });
logger.warn('High memory usage', { usage: process.memoryUsage() });
```

### **Logging Configuration:**
```yaml
# docker-compose.yml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
    labels: "service,environment"

services:
  app:
    image: myapp:latest
    logging: *default-logging
    labels:
      service: "myapp"
      environment: "production"

  # Send logs to external service
  app-with-fluentd:
    image: myapp:latest
    logging:
      driver: fluentd
      options:
        fluentd-address: fluentd:24224
        tag: myapp.logs
```

---

## 🔄 Zero-Downtime Deployment

### **Rolling Update Strategy:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro

  app:
    image: myapp:${VERSION}
    deploy:
      replicas: 3
      update_config:
        parallelism: 1        # Update 1 at a time
        delay: 10s            # Wait 10s between updates
        failure_action: rollback
        monitor: 60s
        order: start-first    # Start new before stopping old
      rollback_config:
        parallelism: 1
        delay: 5s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 10s
      timeout: 3s
      retries: 3
      start_period: 40s
```

### **Blue-Green Deployment:**
```bash
# Current (blue) version running
docker-compose -p myapp-blue up -d

# Deploy new (green) version
docker-compose -p myapp-green up -d

# Test green version
curl http://localhost:8080/health

# Switch traffic to green (update load balancer)
./switch-to-green.sh

# Monitor for issues
# If problems, switch back to blue
./switch-to-blue.sh

# If successful, remove blue
docker-compose -p myapp-blue down
```

### **Canary Deployment:**
```nginx
# nginx.conf - Route 10% to new version
upstream app_v1 {
    server app-v1:3000 weight=9;
}

upstream app_v2 {
    server app-v2:3000 weight=1;
}

server {
    listen 80;

    location / {
        proxy_pass http://app_v1;
        # 10% traffic to v2
    }
}
```

---

## 🎯 Production Deployment Script

### **Automated Deployment:**
```bash
#!/bin/bash
# deploy.sh

set -e

ENV=${1:-production}
VERSION=${2:-latest}

echo "Deploying version $VERSION to $ENV"

# Load environment
source .env.$ENV

# Pre-deployment checks
echo "Running pre-deployment checks..."
./scripts/check-health.sh

# Pull latest images
echo "Pulling images..."
docker-compose -f docker-compose.yml -f docker-compose.$ENV.yml pull

# Backup database
echo "Backing up database..."
./scripts/backup-db.sh

# Deploy with zero downtime
echo "Starting deployment..."
docker-compose -f docker-compose.yml -f docker-compose.$ENV.yml up -d --no-deps --build app

# Wait for health check
echo "Waiting for health checks..."
for i in {1..30}; do
  if curl -f http://localhost:3000/health > /dev/null 2>&1; then
    echo "Health check passed!"
    break
  fi
  if [ $i -eq 30 ]; then
    echo "Health check failed! Rolling back..."
    docker-compose -f docker-compose.yml -f docker-compose.$ENV.yml up -d --no-deps --force-recreate app
    exit 1
  fi
  echo "Attempt $i/30..."
  sleep 2
done

# Run smoke tests
echo "Running smoke tests..."
./scripts/smoke-test.sh

# Update load balancer (if needed)
# ./scripts/update-lb.sh

echo "Deployment successful!"

# Cleanup old images
docker image prune -f

# Send notification
curl -X POST $SLACK_WEBHOOK -d "{\"text\":\"Deployed $VERSION to $ENV\"}"
```

---

## 🐳 Docker Swarm Production

### **Initialize Swarm:**
```bash
# On manager node
docker swarm init --advertise-addr <MANAGER-IP>

# On worker nodes (use token from init output)
docker swarm join --token <TOKEN> <MANAGER-IP>:2377

# View nodes
docker node ls
```

### **Deploy Stack:**
```yaml
# stack.yml
version: '3.8'

services:
  app:
    image: registry.company.com/myapp:${VERSION}
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 10s
        failure_action: rollback
      rollback_config:
        parallelism: 2
        delay: 5s
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    networks:
      - app-network
    secrets:
      - db_password

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    deploy:
      replicas: 2
      placement:
        constraints:
          - node.role == manager
    networks:
      - app-network

networks:
  app-network:
    driver: overlay

secrets:
  db_password:
    external: true
```

```bash
# Deploy stack
docker stack deploy -c stack.yml myapp

# List stacks
docker stack ls

# List services
docker stack services myapp

# View service logs
docker service logs -f myapp_app

# Update service
docker service update --image myapp:v2 myapp_app

# Remove stack
docker stack rm myapp
```

---

## 📝 Hands-on Exercise

**Complete production deployment:**

```bash
mkdir prod-deploy
cd prod-deploy

# 1. Create production Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine AS base
RUN apk update && apk upgrade
WORKDIR /app

FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production

FROM base AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM base AS prod
RUN addgroup -g 1001 nodejs && adduser -S nodejs -u 1001
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./
USER nodejs
EXPOSE 3000
HEALTHCHECK CMD node healthcheck.js
CMD ["node", "dist/index.js"]
EOF

# 2. Create docker-compose.prod.yml
cat > docker-compose.prod.yml << 'EOF'
version: '3.8'

x-logging: &logging
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"

services:
  app:
    image: myapp:${VERSION:-latest}
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
    logging: *logging
    networks:
      - app-net

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    restart: unless-stopped
    logging: *logging
    networks:
      - app-net

networks:
  app-net:
EOF

# 3. Create deployment script
cat > deploy.sh << 'EOF'
#!/bin/bash
set -e

VERSION=${1:-latest}
echo "Deploying version: $VERSION"

# Build
docker build -t myapp:$VERSION .

# Deploy
VERSION=$VERSION docker-compose -f docker-compose.prod.yml up -d

# Health check
sleep 5
for i in {1..10}; do
  if curl -f http://localhost/health; then
    echo "Deployment successful!"
    exit 0
  fi
  sleep 2
done

echo "Health check failed!"
exit 1
EOF

chmod +x deploy.sh

# Cleanup
cd ..
rm -rf prod-deploy
```

---

## 📝 Key Takeaways

✅ **Always use multi-stage builds for production**  
✅ **Implement proper health checks**  
✅ **Handle graceful shutdown**  
✅ **Use environment-specific configurations**  
✅ **Implement structured logging**  
✅ **Plan for zero-downtime deployments**  
✅ **Monitor everything**  
✅ **Have rollback plan ready**  
✅ **Automate deployment process**  
✅ **Security scan before production**  

---

## 🚀 Next Steps

**Optimize resource usage:**

**Next lesson:** [16 - Resource Management](16-resource-management.md) - CPU, memory, and disk optimization

Production is where it counts - deploy with confidence! 🎯
