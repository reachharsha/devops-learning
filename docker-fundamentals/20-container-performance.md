---
render_with_liquid: false
---
# 20 - Container Performance Optimization

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Optimize Docker images for size and speed
- Improve container startup time
- Reduce build time with caching
- Optimize resource usage
- Profile and benchmark containers
- Implement performance best practices
- Use BuildKit features

---

## 📦 Image Size Optimization

### **Choose Minimal Base Images:**
```dockerfile
# ❌ BAD - Full Ubuntu (78MB compressed)
FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3
COPY app.py .
CMD ["python3", "app.py"]

# ✅ BETTER - Slim variant (50MB compressed)
FROM python:3.11-slim
COPY app.py .
CMD ["python", "app.py"]

# ✅ BEST - Alpine (15MB compressed)
FROM python:3.11-alpine
COPY app.py .
CMD ["python", "app.py"]

# ⚡ OPTIMAL - Distroless (No shell, minimal size)
FROM gcr.io/distroless/python3
COPY app.py .
CMD ["app.py"]
```

### **Image Size Comparison:**
```bash
# Compare image sizes
docker images | grep python

# REPOSITORY          TAG            SIZE
# python              3.11           1.01GB
# python              3.11-slim      130MB
# python              3.11-alpine    52MB
# distroless/python3  latest         52MB
```

### **Multi-Stage Builds:**
```dockerfile
# Before: 1.2GB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["npm", "start"]

# After: 150MB (87% reduction!)
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
CMD ["node", "dist/index.js"]
```

### **Remove Unnecessary Files:**
```dockerfile
# ✅ Clean up in same layer
FROM python:3.11-alpine
RUN apk add --no-cache --virtual .build-deps \
    gcc \
    musl-dev \
    && pip install --no-cache-dir numpy \
    && apk del .build-deps

# ✅ Use .dockerignore
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
README.md
.env
.vscode
*.md
tests/
docs/
```

---

## ⚡ Build Performance

### **Layer Caching Strategy:**
```dockerfile
# ❌ BAD - Cache invalidated on any file change
FROM node:18-alpine
WORKDIR /app
COPY . .              # Copies everything - cache breaks here!
RUN npm install
CMD ["npm", "start"]

# ✅ GOOD - Leverage layer caching
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./  # Only copy dependency files first
RUN npm ci             # Cache this layer unless package.json changes
COPY . .               # Copy source code last
CMD ["npm", "start"]
```

### **BuildKit Features:**
```bash
# Enable BuildKit (faster builds, better caching)
export DOCKER_BUILDKIT=1

# Or in docker-compose.yml
export COMPOSE_DOCKER_CLI_BUILD=1
export DOCKER_BUILDKIT=1

# Permanent (add to ~/.bashrc or ~/.zshrc)
echo 'export DOCKER_BUILDKIT=1' >> ~/.zshrc
```

```dockerfile
# syntax=docker/dockerfile:1

# BuildKit cache mounts
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./

# Cache npm packages
RUN --mount=type=cache,target=/root/.npm \
    npm ci --prefer-offline

COPY . .
RUN npm run build
```

### **Parallel Builds:**
```bash
# Build with BuildKit and see progress
DOCKER_BUILDKIT=1 docker build \
  --progress=plain \
  -t myapp:latest .

# Build multiple images in parallel
docker build -t app1:latest ./app1 &
docker build -t app2:latest ./app2 &
docker build -t app3:latest ./app3 &
wait

echo "All builds complete!"
```

### **Build Secrets (No Layer Exposure):**
```dockerfile
# syntax=docker/dockerfile:1

FROM alpine:latest

# Secret only available during build, not in image
RUN --mount=type=secret,id=github_token \
    TOKEN=$(cat /run/secrets/github_token) && \
    git clone https://$TOKEN@github.com/private/repo.git

# Or with ssh
RUN --mount=type=ssh \
    git clone git@github.com:private/repo.git
```

```bash
# Build with secret
DOCKER_BUILDKIT=1 docker build \
  --secret id=github_token,src=./token.txt \
  -t myapp:latest .

# Build with SSH
DOCKER_BUILDKIT=1 docker build \
  --ssh default \
  -t myapp:latest .
```

---

## 🚀 Runtime Performance

### **Optimize Container Startup:**
```dockerfile
# ❌ SLOW - Installs at runtime
FROM python:3.11-alpine
COPY requirements.txt .
CMD pip install -r requirements.txt && python app.py

# ✅ FAST - Pre-installed in image
FROM python:3.11-alpine
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY app.py .
CMD ["python", "app.py"]
```

### **Use Exec Form for CMD:**
```dockerfile
# ❌ SLOW - Shell form (spawns shell process)
CMD python app.py

# ✅ FAST - Exec form (direct execution)
CMD ["python", "app.py"]

# Same for ENTRYPOINT
ENTRYPOINT ["python", "app.py"]
```

### **Minimize Layers:**
```dockerfile
# ❌ BAD - Many layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y vim
RUN apt-get install -y git
RUN apt-get clean

# ✅ GOOD - Single layer
RUN apt-get update && apt-get install -y \
    curl \
    vim \
    git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
```

---

## 📊 Performance Monitoring

### **Benchmark Image Build:**
```bash
# Time the build
time docker build -t myapp:latest .

# Build with BuildKit timing
DOCKER_BUILDKIT=1 docker build \
  --progress=plain \
  -t myapp:latest . 2>&1 | grep 'done'

# Compare build times
echo "=== Without BuildKit ==="
DOCKER_BUILDKIT=0 time docker build -t myapp:v1 .

echo "=== With BuildKit ==="
DOCKER_BUILDKIT=1 time docker build -t myapp:v2 .
```

### **Container Startup Time:**
```bash
# Measure startup time
time docker run --rm myapp:latest echo "Started"

# Benchmark with hyperfine
brew install hyperfine

hyperfine 'docker run --rm myapp:latest echo "Started"'

# Compare different images
hyperfine \
  'docker run --rm python:3.11 python --version' \
  'docker run --rm python:3.11-slim python --version' \
  'docker run --rm python:3.11-alpine python --version'
```

### **Resource Usage Profiling:**
```bash
# Monitor resource usage
docker stats myapp --no-stream

# Continuous monitoring
docker stats myapp

# Memory usage over time
while true; do
  docker stats myapp --no-stream | grep myapp
  sleep 1
done

# CPU profiling (Node.js example)
docker run --rm \
  --cpus=1 \
  -v $(pwd):/app \
  node:18-alpine \
  node --prof app.js

# Analyze profile
docker run --rm \
  -v $(pwd):/app \
  node:18-alpine \
  node --prof-process isolate-*.log
```

---

## 🔧 Application-Level Optimization

### **Node.js Optimization:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy source
COPY . .

# Build
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app

# Production dependencies only
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force

# Copy built app
COPY --from=builder /app/dist ./dist

# Use production mode
ENV NODE_ENV=production

# Optimize V8
ENV NODE_OPTIONS="--max-old-space-size=512"

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js || exit 1

USER node
CMD ["node", "dist/index.js"]
```

### **Python Optimization:**
```dockerfile
FROM python:3.11-alpine AS builder
WORKDIR /app

# Install build dependencies
RUN apk add --no-cache gcc musl-dev

# Install Python dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-alpine
WORKDIR /app

# Copy installed packages
COPY --from=builder /root/.local /root/.local

# Copy application
COPY app.py .

# Update PATH
ENV PATH=/root/.local/bin:$PATH

# Optimize Python
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

USER nobody
CMD ["python", "app.py"]
```

### **Go Optimization:**
```dockerfile
# Stage 1: Build
FROM golang:1.21-alpine AS builder
WORKDIR /app

# Cache dependencies
COPY go.mod go.sum ./
RUN go mod download

# Build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags="-w -s" -o main .

# Stage 2: Minimal runtime
FROM scratch
COPY --from=builder /app/main /main
EXPOSE 8080
ENTRYPOINT ["/main"]

# Final image: ~5MB!
```

---

## 🎯 Network Performance

### **DNS Optimization:**
```bash
# Custom DNS servers
docker run -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  nginx

# In docker-compose.yml
services:
  app:
    image: myapp:latest
    dns:
      - 8.8.8.8
      - 8.8.4.4
```

### **Host Network Mode (Linux Only):**
```bash
# Use host network for maximum performance
# ⚠️ Less isolation, use carefully
docker run -d \
  --network host \
  nginx

# In compose
services:
  app:
    image: myapp:latest
    network_mode: "host"
```

---

## 📈 Best Practices Summary

### **Optimal Dockerfile Pattern:**
```dockerfile
# syntax=docker/dockerfile:1

# ========================================
# Stage 1: Dependencies
# ========================================
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# ========================================
# Stage 2: Build
# ========================================
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci
COPY . .
RUN npm run build && npm run test

# ========================================
# Stage 3: Production
# ========================================
FROM node:18-alpine AS production

# Security: Non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy dependencies and build
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --chown=nodejs:nodejs package*.json ./

# Environment
ENV NODE_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=512"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js || exit 1

# Switch to non-root
USER nodejs

# Expose port
EXPOSE 3000

# Start
CMD ["node", "dist/index.js"]
```

### **.dockerignore Template:**
```
# Dependencies
node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Testing
coverage
.nyc_output
*.test.js
*.spec.js
__tests__

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
.travis.yml

# Documentation
README.md
CHANGELOG.md
LICENSE
docs

# Environment
.env
.env.local
.env.*.local

# Build artifacts
dist
build
*.log

# OS
.DS_Store
Thumbs.db
```

---

## 🎯 Quick Check: Do You Understand?

1. **Why use Alpine images?**
   <details>
   <summary>Answer</summary>
   Smaller size (5-10x smaller), faster downloads, reduced attack surface
   </details>

2. **What's the benefit of multi-stage builds?**
   <details>
   <summary>Answer</summary>
   Separate build and runtime dependencies, resulting in much smaller final images
   </details>

3. **How does BuildKit improve builds?**
   <details>
   <summary>Answer</summary>
   Better caching, parallel builds, build secrets, improved performance
   </details>

4. **Why use exec form for CMD?**
   <details>
   <summary>Answer</summary>
   Faster startup (no shell process), signals handled correctly
   </details>

5. **What should go in .dockerignore?**
   <details>
   <summary>Answer</summary>
   Tests, docs, IDE files, node_modules, .git - anything not needed in image
   </details>

---

## 📝 Hands-on Exercise

**Optimize an application:**

```bash
# 1. Create unoptimized app
mkdir optimize-demo
cd optimize-demo

cat > app.js << 'EOF'
const express = require('express');
const app = express();
app.get('/', (req, res) => res.send('Hello!'));
app.listen(3000, () => console.log('Server started'));
EOF

cat > package.json << 'EOF'
{
  "name": "demo",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

# 2. Unoptimized Dockerfile
cat > Dockerfile.unoptimized << 'EOF'
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD npm start
EOF

# 3. Build and measure
echo "=== Unoptimized ==="
time docker build -f Dockerfile.unoptimized -t demo:unoptimized .
docker images demo:unoptimized

# 4. Optimized Dockerfile
cat > Dockerfile << 'EOF'
# syntax=docker/dockerfile:1
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm npm ci --only=production

FROM node:18-alpine
RUN addgroup -g 1001 nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .
USER nodejs
CMD ["node", "app.js"]
EOF

# 5. Build optimized
echo "=== Optimized ==="
DOCKER_BUILDKIT=1 time docker build -t demo:optimized .
docker images demo:optimized

# 6. Compare sizes
docker images | grep demo

# 7. Compare startup times
echo "=== Startup Times ==="
hyperfine \
  'docker run --rm -d demo:unoptimized' \
  'docker run --rm -d demo:optimized'

# 8. Cleanup
cd ..
rm -rf optimize-demo
docker rmi demo:unoptimized demo:optimized
```

---

## 📝 Key Takeaways

✅ **Use Alpine or distroless base images**  
✅ **Multi-stage builds reduce image size dramatically**  
✅ **Order Dockerfile commands for optimal caching**  
✅ **Enable BuildKit for better build performance**  
✅ **Use .dockerignore to exclude unnecessary files**  
✅ **Combine RUN commands to reduce layers**  
✅ **Use exec form for CMD and ENTRYPOINT**  
✅ **Install only production dependencies**  
✅ **Clean up package manager caches**  
✅ **Monitor and benchmark regularly**  

---

## 🚀 Next Steps

**Troubleshoot like a pro:**

**Next lesson:** [21 - Advanced Troubleshooting](21-advanced-troubleshooting.md) - Debug production issues

Performance matters - optimize early, optimize often! ⚡
