---
render_with_liquid: false
---
# 11 - Docker Compose Advanced

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Use advanced Compose file features
- Work with multiple Compose files
- Implement health checks properly
- Use build arguments and secrets
- Configure resource limits
- Set up development vs production environments
- Use Compose with CI/CD pipelines
- Debug complex multi-container applications

---

## 🎛️ Advanced Service Configuration

### **Health Checks:**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s      # Check every 10 seconds
      timeout: 5s        # Max 5 seconds per check
      retries: 5         # Retry 5 times before unhealthy
      start_period: 30s  # Grace period on startup

  app:
    image: myapp:latest
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 40s
    depends_on:
      postgres:
        condition: service_healthy  # Wait for healthy status
```

### **Resource Limits:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '0.50'      # Max 50% of one CPU
          memory: 512M      # Max 512 MB
        reservations:
          cpus: '0.25'      # Reserve 25% of one CPU
          memory: 256M      # Reserve 256 MB
    
  # Alternative (v2 syntax)
  database:
    image: postgres:15
    mem_limit: 1g
    mem_reservation: 512m
    cpus: 1.5
    cpu_shares: 1024
```

### **Logging Configuration:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
        max-size: "10m"      # Max log file size
        max-file: "3"        # Keep 3 files
        
  nginx:
    image: nginx:alpine
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://192.168.0.42:123"
        tag: "nginx"
        
  api:
    image: api:latest
    logging:
      driver: "fluentd"
      options:
        fluentd-address: localhost:24224
        tag: api.logs
```

---

## 📚 Multiple Compose Files

### **Base Configuration:**
```yaml
# docker-compose.yml (base)
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin

  redis:
    image: redis:alpine

  app:
    build: .
    environment:
      DB_HOST: postgres
      REDIS_HOST: redis
```

### **Development Override:**
```yaml
# docker-compose.override.yml (automatically loaded)
version: '3.8'

services:
  app:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src      # Live code reload
    environment:
      DEBUG: "true"
      LOG_LEVEL: debug
    ports:
      - "8000:8000"
      - "9229:9229"         # Debugger port

  postgres:
    ports:
      - "5432:5432"         # Expose for local tools
```

### **Production Configuration:**
```yaml
# docker-compose.prod.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: production
    restart: unless-stopped
    environment:
      DEBUG: "false"
      LOG_LEVEL: info
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M

  postgres:
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    volumes:
      - postgres-prod-data:/var/lib/postgresql/data

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  postgres-prod-data:
    driver: local
```

### **Using Multiple Files:**
```bash
# Development (base + override)
docker-compose up -d

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d

# Testing
docker-compose -f docker-compose.yml -f docker-compose.test.yml up

# Custom environment
docker-compose -f docker-compose.yml -f docker-compose.staging.yml up -d
```

---

## 🔐 Secrets Management

### **File-based Secrets:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    secrets:
      - db_password
      - api_key
    environment:
      DB_PASSWORD_FILE: /run/secrets/db_password
      API_KEY_FILE: /run/secrets/api_key

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

```bash
# Create secrets directory
mkdir secrets
echo "mySecretPassword123" > secrets/db_password.txt
echo "sk-1234567890abcdef" > secrets/api_key.txt

# Add to .gitignore
echo "secrets/" >> .gitignore
```

### **External Secrets:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    secrets:
      - db_password

secrets:
  db_password:
    external: true  # Secret created outside of Compose
```

```bash
# Create external secret (Docker Swarm)
echo "secret123" | docker secret create db_password -
```

---

## 🏗️ Build Arguments

### **Passing Build Args:**
```yaml
version: '3.8'

services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NODE_VERSION: 18
        APP_ENV: production
        BUILD_DATE: ${BUILD_DATE}
      target: production
      cache_from:
        - myapp:latest
```

### **Dockerfile Using Args:**
```dockerfile
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}-alpine

ARG APP_ENV=development
ARG BUILD_DATE

ENV APP_ENV=${APP_ENV}
LABEL build_date=${BUILD_DATE}

RUN echo "Building for ${APP_ENV} environment"
```

### **Build and Pass Args:**
```bash
# Set build args
BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
docker-compose build --build-arg BUILD_DATE=$BUILD_DATE

# Or in .env file
echo "BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')" > .env
docker-compose build
```

---

## 🌐 Advanced Networking

### **Custom Network Configuration:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    networks:
      frontend:
        aliases:
          - api-server
      backend:
        ipv4_address: 172.20.0.10

networks:
  frontend:
    driver: bridge
    driver_opts:
      com.docker.network.bridge.name: my-bridge
      com.docker.network.bridge.enable_icc: "true"
      com.docker.network.bridge.enable_ip_masquerade: "true"
    ipam:
      driver: default
      config:
        - subnet: 172.18.0.0/16
          gateway: 172.18.0.1

  backend:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### **External Networks:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    networks:
      - existing-network

networks:
  existing-network:
    external: true
    name: my-pre-existing-network
```

---

## 🔄 Extending Services

### **Using `extends`:**
```yaml
# common.yml
version: '3.8'

services:
  base-service:
    image: alpine:latest
    restart: unless-stopped
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "3"
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  app:
    extends:
      file: common.yml
      service: base-service
    image: myapp:latest
    ports:
      - "8000:8000"
```

### **YAML Anchors (Better Alternative):**
```yaml
version: '3.8'

x-common-config: &common-config
  restart: unless-stopped
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

x-healthcheck: &healthcheck
  interval: 30s
  timeout: 3s
  retries: 3
  start_period: 40s

services:
  app1:
    <<: *common-config
    image: app1:latest
    healthcheck:
      <<: *healthcheck
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]

  app2:
    <<: *common-config
    image: app2:latest
    healthcheck:
      <<: *healthcheck
      test: ["CMD", "curl", "-f", "http://localhost:9000/health"]
```

---

## 🎯 Development Workflow

### **Complete Dev Setup:**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: dev_db
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: dev
    ports:
      - "5432:5432"
    volumes:
      - postgres-dev-data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  redis:
    image: redis:alpine
    ports:
      - "6379:6379"

  mailhog:  # Email testing
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

  app:
    build:
      context: .
      target: development
    volumes:
      - ./src:/app/src
      - ./tests:/app/tests
      - /app/node_modules  # Don't override node_modules
    environment:
      NODE_ENV: development
      DEBUG: "app:*"
      DB_HOST: postgres
      REDIS_HOST: redis
      SMTP_HOST: mailhog
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    command: npm run dev
    stdin_open: true   # Keep STDIN open
    tty: true          # Allocate TTY

volumes:
  postgres-dev-data:
```

---

## 🚀 Production Best Practices

### **Production-Ready Compose:**
```yaml
# docker-compose.prod.yml
version: '3.8'

x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

services:
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/postgres_password
    secrets:
      - postgres_password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    logging: *default-logging
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M

  app:
    image: myapp:${VERSION:-latest}
    restart: unless-stopped
    environment:
      NODE_ENV: production
      DB_HOST: postgres
    secrets:
      - app_secrets
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
      start_period: 60s
    logging: *default-logging
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '0.50'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M

  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - nginx-cache:/var/cache/nginx
    depends_on:
      - app
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    logging: *default-logging

secrets:
  postgres_password:
    file: ./secrets/postgres_password.txt
  app_secrets:
    file: ./secrets/app_secrets.env

volumes:
  postgres-data:
    driver: local
  nginx-cache:
    driver: local
```

---

## 🐛 Debugging Complex Apps

### **View All Logs:**
```bash
# All services
docker-compose logs -f

# Specific services
docker-compose logs -f app postgres

# Last 100 lines
docker-compose logs --tail=100 app

# Since timestamp
docker-compose logs --since 2026-01-24T10:00:00

# With timestamps
docker-compose logs -t
```

### **Inspect Services:**
```bash
# View configuration
docker-compose config

# Show running processes
docker-compose top

# Show resource usage
docker-compose ps

# Detailed service info
docker inspect $(docker-compose ps -q app)
```

### **Network Debugging:**
```bash
# Get network name
docker-compose config | grep networks: -A 5

# Inspect network
docker network inspect myapp_default

# Test connectivity
docker-compose exec app ping postgres
docker-compose exec app nslookup postgres
docker-compose exec app wget -O- http://nginx
```

### **Debug Build Issues:**
```bash
# Build with verbose output
docker-compose build --progress=plain

# Build without cache
docker-compose build --no-cache

# Build specific service
docker-compose build app

# See build args
docker-compose config --services
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you use multiple Compose files?**
   <details>
   <summary>Answer</summary>
   docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
   </details>

2. **What's the purpose of health checks?**
   <details>
   <summary>Answer</summary>
   Monitor service health and control dependency startup order
   </details>

3. **How do you limit container memory in Compose?**
   <details>
   <summary>Answer</summary>
   Use deploy.resources.limits.memory or mem_limit
   </details>

4. **What are YAML anchors used for?**
   <details>
   <summary>Answer</summary>
   Reuse configuration blocks across multiple services
   </details>

5. **How do you pass secrets securely?**
   <details>
   <summary>Answer</summary>
   Use secrets section with file or external references
   </details>

---

## 📝 Hands-on Exercise

**Build a complete production-ready stack:**

```bash
mkdir advanced-compose
cd advanced-compose

# Create structure
mkdir -p secrets nginx

# Create secret
echo "SuperSecret123!" > secrets/db_password.txt

# Create nginx config
cat > nginx/nginx.conf << 'EOF'
events {
    worker_connections 1024;
}

http {
    upstream app {
        server app:3000;
    }

    server {
        listen 80;
        
        location / {
            proxy_pass http://app;
            proxy_set_header Host $host;
        }
        
        location /health {
            return 200 "healthy\n";
        }
    }
}
EOF

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

x-logging: &default-logging
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    secrets:
      - db_password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - pg-data:/var/lib/postgresql/data
    logging: *default-logging

  app:
    image: nginx:alpine
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:80"]
      interval: 30s
    logging: *default-logging

  nginx:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    logging: *default-logging

secrets:
  db_password:
    file: ./secrets/db_password.txt

volumes:
  pg-data:
EOF

# Start stack
docker-compose up -d

# Check health
docker-compose ps

# View logs
docker-compose logs

# Test
curl http://localhost:8080/health

# Cleanup
docker-compose down -v
cd ..
rm -rf advanced-compose
```

---

## 📝 Key Takeaways

✅ **Health checks ensure service readiness**  
✅ **Multiple Compose files for different environments**  
✅ **Resource limits prevent resource exhaustion**  
✅ **Secrets keep sensitive data secure**  
✅ **YAML anchors reduce configuration duplication**  
✅ **Logging configuration manages log volume**  
✅ **Build args customize image builds**  
✅ **Proper dependencies prevent race conditions**  
✅ **Network configuration controls isolation**  
✅ **Production configs need restart policies and limits**  

---

## 🚀 Next Steps

**See real-world applications:**

**Next lesson:** [12 - Real-World Compose Projects](12-real-world-compose.md) - Complete application examples

Master these advanced features for production! 🎯
