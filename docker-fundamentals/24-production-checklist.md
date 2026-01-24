---
render_with_liquid: false
---
# 24 - Production Checklist & Best Practices

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Apply Docker best practices consistently
- Use the production-ready checklist
- Optimize your Docker workflow
- Understand security fundamentals
- Plan your next learning steps
- Apply Docker in real-world scenarios

---

## ✅ Production Deployment Checklist

### **🏗️ Image Build:**
```
□ Use specific base image tags (not :latest)
□ Multi-stage builds implemented
□ Minimal base images (Alpine/Distroless)
□ .dockerignore configured
□ Layers ordered for optimal caching
□ No secrets in image layers
□ BuildKit enabled
□ Image scanned for vulnerabilities
□ Image size optimized (<200MB ideal)
□ Build reproducible (same inputs = same output)
```

### **🔒 Security:**
```
□ Running as non-root user
□ Read-only root filesystem where possible
□ Dropped unnecessary capabilities
□ Security options configured (no-new-privileges)
□ Secrets managed externally (not in env vars)
□ Regular security scans (Trivy, Snyk)
□ Base images updated regularly
□ TLS/HTTPS enabled for communication
□ Network policies configured
□ Resource limits set
```

### **⚙️ Configuration:**
```
□ Environment variables for configuration
□ Health checks implemented
□ Graceful shutdown handling
□ Proper signal handling (SIGTERM)
□ Logging to stdout/stderr
□ Structured logging (JSON)
□ Timezone configured if needed
□ Restart policies set
□ Resource limits configured
□ Dependencies version-pinned
```

### **🌐 Networking:**
```
□ Only required ports exposed
□ Port binding to localhost when possible
□ Custom networks for isolation
□ DNS resolution tested
□ Service discovery configured
□ Load balancing implemented
□ SSL/TLS certificates valid
□ Network policies defined
□ Ingress/egress rules configured
```

### **💾 Data Management:**
```
□ Volumes for persistent data
□ Volume backup strategy
□ Database migrations automated
□ Data retention policy defined
□ Backup verification tested
□ Disaster recovery plan
□ Volume permissions correct
□ Storage driver appropriate
```

### **📊 Monitoring & Logging:**
```
□ Container metrics collected (Prometheus)
□ Logs centralized (ELK, Loki)
□ Dashboards created (Grafana)
□ Alerts configured
□ Health endpoints exposed
□ APM integrated (if applicable)
□ Error tracking (Sentry, etc.)
□ Log retention configured
□ Distributed tracing (Jaeger)
```

### **🚀 Deployment:**
```
□ CI/CD pipeline configured
□ Automated testing in place
□ Staging environment matches production
□ Rollback procedure documented
□ Zero-downtime deployment
□ Blue-green or canary deployment
□ Database migration strategy
□ Feature flags implemented
□ Version tagging strategy
□ Deployment documented
```

### **📈 Performance:**
```
□ Resource limits tuned
□ Image size optimized
□ Startup time minimized
□ Build time optimized
□ Cache configured properly
□ CDN for static assets
□ Database indexes created
□ Connection pooling configured
□ Horizontal scaling tested
```

### **📚 Documentation:**
```
□ README with setup instructions
□ Environment variables documented
□ API documentation current
□ Architecture diagrams
□ Runbooks for common issues
□ Deployment procedures
□ Disaster recovery plan
□ Dependencies documented
□ Changelog maintained
```

---

## 🎨 Dockerfile Best Practices

### **Perfect Production Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1

# ==========================================
# Base Stage - Common dependencies
# ==========================================
FROM node:18-alpine AS base

# Install security updates
RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

# Set working directory
WORKDIR /app

# ==========================================
# Dependencies Stage - Production deps
# ==========================================
FROM base AS dependencies

# Copy dependency files
COPY package*.json ./

# Install production dependencies with cache mount
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production && \
    npm cache clean --force

# ==========================================
# Build Stage - Compile application
# ==========================================
FROM base AS build

# Copy dependency files
COPY package*.json ./

# Install all dependencies (including dev)
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Copy source code
COPY . .

# Run tests
RUN npm run test

# Build application
RUN npm run build

# ==========================================
# Production Stage - Final image
# ==========================================
FROM base AS production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy production dependencies
COPY --from=dependencies --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy built application
COPY --from=build --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

# Set environment
ENV NODE_ENV=production \
    NODE_OPTIONS="--max-old-space-size=512" \
    PORT=3000

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js || exit 1

# Start application
CMD ["node", "dist/index.js"]
```

### **.dockerignore Template:**
```
# Dependencies
node_modules
npm-debug.log*
yarn-error.log*
package-lock.json
yarn.lock

# Testing
coverage
.nyc_output
*.test.js
*.spec.js
__tests__
__mocks__

# IDE
.vscode
.idea
*.swp
*.swo
*~

# Git
.git
.gitignore
.gitattributes

# CI/CD
.github
.gitlab-ci.yml
.circleci
.travis.yml

# Documentation
README.md
CHANGELOG.md
LICENSE
docs
*.md

# Environment
.env
.env.*
!.env.example

# Build artifacts
dist
build
*.log
tmp
temp

# OS
.DS_Store
Thumbs.db
Desktop.ini

# Docker
Dockerfile*
docker-compose*
.dockerignore
```

---

## 🐳 Docker Compose Best Practices

### **Production-Ready Compose:**
```yaml
version: '3.8'

# Reusable configurations
x-logging: &default-logging
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
    labels: "service,environment"

x-healthcheck: &default-healthcheck
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s

services:
  app:
    image: ${REGISTRY}/myapp:${VERSION:-latest}
    
    # Restart policy
    restart: unless-stopped
    
    # Resources
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
    
    # Security
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    user: "1001:1001"
    
    # Tmpfs for writable directories
    tmpfs:
      - /tmp:size=64M,mode=1777
      - /var/run:size=64M,mode=755
    
    # Environment
    environment:
      NODE_ENV: production
      DB_HOST: postgres
    
    # Secrets (avoid plain text)
    secrets:
      - db_password
      - api_key
    
    # Volumes
    volumes:
      - app-data:/app/data:rw
    
    # Networking
    networks:
      - frontend
      - backend
    ports:
      - "3000:3000"
    
    # Dependencies
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    
    # Logging
    logging: *default-logging
    
    # Health check
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
    
    # Labels
    labels:
      com.example.service: "app"
      com.example.environment: "production"

  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD_FILE: /run/secrets/db_password
    
    secrets:
      - db_password
    
    volumes:
      - postgres-data:/var/lib/postgresql/data
    
    networks:
      - backend
    
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD-SHELL", "pg_isready -U myuser"]
    
    logging: *default-logging
    
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G

  redis:
    image: redis:alpine
    restart: unless-stopped
    
    command: >
      redis-server
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --requirepass ${REDIS_PASSWORD}
    
    networks:
      - backend
    
    healthcheck:
      <<: *default-healthcheck
      test: ["CMD", "redis-cli", "ping"]
    
    logging: *default-logging
    
    volumes:
      - redis-data:/data

networks:
  frontend:
  backend:
    internal: true

volumes:
  app-data:
  postgres-data:
  redis-data:

secrets:
  db_password:
    file: ./secrets/db_password.txt
  api_key:
    file: ./secrets/api_key.txt
```

---

## 🛠️ Essential Commands Reference

### **Daily Commands:**
```bash
# Build
docker build -t myapp:latest .
DOCKER_BUILDKIT=1 docker build -t myapp:latest .

# Run
docker run -d --name myapp -p 3000:3000 myapp:latest

# Logs
docker logs -f myapp
docker logs --tail 100 myapp

# Stats
docker stats myapp
docker stats --no-stream

# Exec
docker exec -it myapp sh
docker exec myapp ps aux

# Inspect
docker inspect myapp
docker inspect --format='{ {.State.Status} }' myapp

# Cleanup
docker system prune -a
docker volume prune
```

### **Compose Commands:**
```bash
# Start
docker-compose up -d

# Scale
docker-compose up -d --scale app=3

# Logs
docker-compose logs -f app

# Restart
docker-compose restart app

# Stop
docker-compose stop

# Down (remove)
docker-compose down
docker-compose down -v  # With volumes

# Build
docker-compose build
docker-compose build --no-cache
```

### **Debugging Commands:**
```bash
# Check health
docker inspect --format='{ {.State.Health.Status} }' myapp

# Network debug
docker exec myapp ping -c 3 db
docker exec myapp nslookup db

# Process debug
docker top myapp
docker exec myapp ps aux

# File debug
docker exec myapp ls -la /app
docker cp myapp:/app/logs/error.log ./

# Stats
docker stats myapp --no-stream
```

---

## 🎓 Learning Path Forward

### **Next Steps:**
```
You've completed Docker Fundamentals! 🎉

Recommended Learning Path:

1. ☸️ Kubernetes Deep Dive
   - Pod management
   - Deployments & StatefulSets
   - Services & Ingress
   - ConfigMaps & Secrets
   - Helm charts
   - Operators

2. 🔄 Advanced CI/CD
   - GitOps with ArgoCD
   - Tekton Pipelines
   - Spinnaker
   - Advanced GitHub Actions

3. 📊 Observability
   - Prometheus & Grafana
   - Jaeger tracing
   - ELK Stack
   - OpenTelemetry

4. 🔒 Security
   - Container security
   - Image signing
   - Policy enforcement (OPA)
   - Falco runtime security

5. 🏗️ Cloud Native
   - Service mesh (Istio, Linkerd)
   - Serverless (Knative)
   - Event-driven (NATS, Kafka)
   - Cloud platforms (AWS ECS, GKE, AKS)
```

### **Practice Projects:**
```
Build These to Master Docker:

1. 📝 Blog Platform
   - Frontend (React)
   - Backend API (Node.js/Python)
   - Database (PostgreSQL)
   - Cache (Redis)
   - Full CI/CD pipeline

2. 🛒 E-commerce Microservices
   - User service
   - Product catalog
   - Order management
   - Payment gateway
   - API Gateway

3. 📊 Monitoring Stack
   - Prometheus
   - Grafana
   - AlertManager
   - Node Exporter
   - Custom metrics

4. 🔄 CI/CD Platform
   - Jenkins
   - GitLab Runner
   - SonarQube
   - Nexus/Artifactory
   - Automated testing

5. 🎮 Real-time Application
   - WebSocket server
   - Redis pub/sub
   - Load balancer
   - Horizontal scaling
```

---

## 💡 Common Mistakes to Avoid

### **❌ Don't Do This:**
```dockerfile
# Using :latest tag
FROM node:latest

# Running as root
# (no USER directive)

# Hardcoding secrets
ENV API_KEY="secret123"

# Not cleaning up
RUN apt-get update
RUN apt-get install curl

# Copying everything
COPY . .

# No health check
# (missing HEALTHCHECK)
```

### **✅ Do This Instead:**
```dockerfile
# Specific version
FROM node:18-alpine

# Non-root user
RUN adduser -D myuser
USER myuser

# External secrets
# Use --secret or environment at runtime

# Clean up in same layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Selective copying
COPY package*.json ./
RUN npm ci
COPY src ./src

# Health check
HEALTHCHECK CMD curl -f http://localhost/health || exit 1
```

---

## 🎯 Quick Reference Card

### **Essential Docker Commands:**
```bash
# Images
docker images                    # List images
docker build -t name:tag .       # Build image
docker pull image:tag            # Pull image
docker push image:tag            # Push image
docker rmi image                 # Remove image

# Containers
docker ps                        # List running
docker ps -a                     # List all
docker run -d image              # Run detached
docker stop container            # Stop container
docker rm container              # Remove container
docker logs -f container         # Follow logs
docker exec -it container sh     # Interactive shell

# System
docker system df                 # Disk usage
docker system prune -a           # Clean all
docker stats                     # Resource usage
docker info                      # Docker info

# Networks
docker network ls                # List networks
docker network create net        # Create network
docker network inspect net       # Inspect network

# Volumes
docker volume ls                 # List volumes
docker volume create vol         # Create volume
docker volume inspect vol        # Inspect volume
docker volume prune              # Remove unused
```

---

## 🎊 Congratulations!

**You've completed the Docker Fundamentals course!** 🎉

### **What You've Learned:**
```
✅ Docker architecture and concepts
✅ Images and containers
✅ Dockerfile best practices
✅ Docker Compose
✅ Networking and volumes
✅ Security fundamentals
✅ Production deployment
✅ Monitoring and logging
✅ CI/CD integration
✅ Microservices architecture
✅ Kubernetes introduction
✅ Troubleshooting techniques
```

### **You're Now Ready To:**
```
🚀 Build production-ready containers
🚀 Deploy microservices applications
🚀 Implement CI/CD pipelines
🚀 Optimize for performance
🚀 Secure containerized applications
🚀 Monitor and troubleshoot
🚀 Scale applications
🚀 Mentor others
```

---

## 📚 Additional Resources

### **Official Documentation:**
- [Docker Docs](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Docker Compose Reference](https://docs.docker.com/compose/)

### **Books:**
- "Docker Deep Dive" by Nigel Poulton
- "Kubernetes in Action" by Marko Lukša
- "The DevOps Handbook"
- "Site Reliability Engineering" (Google)

### **Online Platforms:**
- [Play with Docker](https://labs.play-with-docker.com/)
- [Katacoda](https://www.katacoda.com/)
- [Docker Tutorials](https://www.docker.com/101-tutorial)
- [KodeKloud](https://kodekloud.com/)

### **Communities:**
- Docker Community Slack
- r/docker on Reddit
- Stack Overflow
- Docker Forums

---

## 🙏 Thank You!

Thank you for completing this comprehensive Docker course!

**Keep Learning. Keep Building. Keep Shipping.** 🚢

**Remember:** The best way to learn is by doing. Start building projects, contribute to open source, and share your knowledge with others.

**Good luck on your DevOps journey!** 🎯

---

**Course Complete! 🎓**

Return to: [00 - START HERE](00-START-HERE.md) | [Docker Fundamentals Homepage](../README.md)
