---
render_with_liquid: false
---
# 07 - Dockerfile Best Practices & Optimization

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Write efficient and secure Dockerfiles
- Optimize image size and build time
- Implement multi-stage builds
- Use .dockerignore effectively
- Apply layer caching strategies
- Follow security best practices
- Debug build issues

---

## 🎯 Layer Optimization

### **Understand Layers:**
Every Dockerfile instruction creates a new layer:

```dockerfile
FROM ubuntu:22.04        # Layer 1
RUN apt update           # Layer 2
RUN apt install curl     # Layer 3
RUN apt install wget     # Layer 4
COPY app.py /app/        # Layer 5
```

**Problem:** Each RUN creates a layer, even if cleaning up in next layer:
```dockerfile
# ❌ BAD - 3 layers, keeps intermediate data
RUN apt update                          # Layer 1 (80 MB)
RUN apt install -y package              # Layer 2 (50 MB)
RUN rm -rf /var/lib/apt/lists/*         # Layer 3 (0 MB, but previous layers still exist!)
# Total: 130 MB

# ✅ GOOD - 1 layer, cleaned immediately
RUN apt update && \
    apt install -y package && \
    rm -rf /var/lib/apt/lists/*         # Layer 1 (50 MB)
# Total: 50 MB
```

---

## 📦 Minimize Layers

### **Chain Commands:**
```dockerfile
# ❌ BAD - Many layers
RUN apt update
RUN apt install -y curl
RUN apt install -y wget
RUN apt clean
RUN rm -rf /var/lib/apt/lists/*

# ✅ GOOD - Single layer
RUN apt update && \
    apt install -y \
        curl \
        wget && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*
```

### **Combine Related Operations:**
```dockerfile
# ❌ BAD - Separate operations
RUN mkdir /app
RUN mkdir /app/logs
RUN mkdir /app/config
RUN chmod 755 /app

# ✅ GOOD - Combined
RUN mkdir -p /app/logs /app/config && \
    chmod 755 /app
```

---

## 🗂️ Order Layers by Change Frequency

### **Layer Caching:**
Docker caches layers. If a layer changes, all subsequent layers rebuild.

```dockerfile
# ❌ BAD - Code changes invalidate dependency cache
FROM node:18-alpine
WORKDIR /app
COPY . .                    # Changes often → invalidates next line
RUN npm install             # Reinstalls deps every code change!
CMD ["node", "app.js"]

# ✅ GOOD - Dependencies cached separately
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./       # Changes rarely
RUN npm install             # Cached unless package.json changes
COPY . .                    # Changes often, but doesn't affect npm install
CMD ["node", "app.js"]
```

### **Optimal Order:**
```dockerfile
FROM base-image             # 1. Base (changes rarely)
RUN install-system-deps     # 2. System packages (changes rarely)
COPY requirements.txt .     # 3. Dependency files (changes occasionally)
RUN install-app-deps        # 4. App dependencies (changes occasionally)
COPY . .                    # 5. Application code (changes frequently)
CMD ["app"]                 # 6. Default command (changes rarely)
```

---

## 🎯 Use Specific Tags

### **Version Pinning:**
```dockerfile
# ❌ BAD - Unpredictable
FROM python:latest
FROM node
FROM ubuntu

# ✅ GOOD - Explicit versions
FROM python:3.11-slim
FROM node:18-alpine
FROM ubuntu:22.04

# ✅ BETTER - With digest (immutable)
FROM python:3.11-slim@sha256:abc123...

# ✅ BEST - Multi-stage with specific versions
FROM node:18-alpine AS builder
FROM nginx:1.25-alpine
```

---

## 🪶 Choose Minimal Base Images

### **Image Size Comparison:**
```dockerfile
# Ubuntu: ~77 MB
FROM ubuntu:22.04

# Debian slim: ~50 MB
FROM debian:11-slim

# Python standard: ~920 MB
FROM python:3.11

# Python slim: ~180 MB
FROM python:3.11-slim

# Python alpine: ~50 MB
FROM python:3.11-alpine

# Scratch (empty): ~0 MB
FROM scratch
```

### **When to Use Each:**
```dockerfile
# Full OS (ubuntu, debian)
# Use when: Need many system tools, compatibility
FROM ubuntu:22.04

# Slim variants
# Use when: Need basic tools, balance size/compatibility
FROM python:3.11-slim

# Alpine
# Use when: Minimize size, simple dependencies
FROM python:3.11-alpine

# Scratch
# Use when: Compiled binaries (Go, Rust), minimal attack surface
FROM scratch
```

### **Alpine Considerations:**
```dockerfile
# Alpine uses musl instead of glibc
# Some packages may not work or need compilation

# ✅ Works well:
FROM node:18-alpine        # JavaScript (no compilation)
FROM nginx:alpine          # Static binaries

# ⚠️ May have issues:
FROM python:3.11-alpine    # Some packages need build tools
# Solution: Install build dependencies
RUN apk add --no-cache build-base linux-headers
```

---

## 🏗️ Multi-Stage Builds

### **Basic Multi-Stage:**
```dockerfile
# ❌ BAD - Single stage, includes build tools
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install             # Includes dev dependencies
COPY . .
RUN npm run build           # Build tools in final image
CMD ["node", "dist/app.js"]
# Final image: ~1.5 GB

# ✅ GOOD - Multi-stage, minimal final image
# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/app.js"]
# Final image: ~150 MB
```

### **Advanced Multi-Stage:**
```dockerfile
# Test stage
FROM node:18 AS test
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm test

# Build stage
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force
USER node
CMD ["node", "dist/app.js"]

# Build specific stage:
# docker build --target test .
# docker build --target production .
```

### **Copy from Multiple Stages:**
```dockerfile
FROM golang:1.20 AS backend-builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o server

FROM node:18 AS frontend-builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM alpine:3.18
WORKDIR /app
COPY --from=backend-builder /app/server ./
COPY --from=frontend-builder /app/dist ./public
CMD ["./server"]
```

---

## 🚫 Use .dockerignore

### **Why .dockerignore:**
- Reduces build context size
- Speeds up builds
- Prevents sensitive files in image
- Keeps image clean

### **Example .dockerignore:**
```
# Version control
.git
.gitignore
.github

# Dependencies
node_modules
vendor
venv
__pycache__

# Build artifacts
dist
build
*.pyc
*.pyo
*.so
*.o

# IDE
.vscode
.idea
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs

# Tests
test
tests
*.test.js
coverage

# Documentation
README.md
docs
*.md

# Environment
.env
.env.local
*.key
*.pem

# Docker
Dockerfile*
docker-compose*.yml
.dockerignore

# CI/CD
.travis.yml
.gitlab-ci.yml
Jenkinsfile
```

### **Patterns:**
```
# Ignore all markdown except README
*.md
!README.md

# Ignore directory
node_modules

# Ignore specific files
.env
secret.key

# Wildcards
**/*.log
temp/*
*/temp/*

# Comments
# This is a comment
```

---

## 🔐 Security Best Practices

### **1. Don't Run as Root:**
```dockerfile
# ❌ BAD - Runs as root
FROM node:18
WORKDIR /app
COPY . .
CMD ["node", "app.js"]  # Running as root!

# ✅ GOOD - Non-root user
FROM node:18-alpine
WORKDIR /app
COPY --chown=node:node . .
USER node
CMD ["node", "app.js"]

# ✅ BETTER - Custom user
FROM python:3.11-slim
RUN useradd -m -u 1001 -s /bin/bash appuser
WORKDIR /app
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python", "app.py"]
```

### **2. No Secrets in Dockerfile:**
```dockerfile
# ❌ NEVER DO THIS
FROM python:3.11
ENV API_KEY="sk-secret123456"        # ← EXPOSED IN IMAGE!
ENV DB_PASSWORD="mypassword"         # ← EXPOSED IN IMAGE!

# ✅ GOOD - Use runtime environment
FROM python:3.11
# Pass secrets at runtime:
# docker run -e API_KEY=secret image

# ✅ BETTER - Use Docker secrets
FROM python:3.11
# docker secret create api_key key.txt
# docker service create --secret api_key image
```

### **3. Use COPY Instead of ADD:**
```dockerfile
# ❌ AVOID - ADD has side effects
ADD app.tar.gz /app/     # Auto-extracts
ADD http://url/file /app/ # Downloads (security risk)

# ✅ GOOD - COPY is explicit
COPY app.tar.gz /app/
RUN tar -xzf /app/app.tar.gz
```

### **4. Scan Images for Vulnerabilities:**
```bash
# Using Docker Scout (built-in)
docker scout cves my-image:latest

# Using Trivy
trivy image my-image:latest

# Using Snyk
snyk container test my-image:latest

# Using Clair
docker scan my-image:latest
```

### **5. Minimal Attack Surface:**
```dockerfile
# ✅ GOOD - Remove unnecessary tools
FROM python:3.11-slim
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        # Only install what you need
        libpq5 && \
    # Remove package manager
    apt-get purge -y --auto-remove && \
    rm -rf /var/lib/apt/lists/*
```

---

## ⚡ Build Performance

### **Use BuildKit:**
```bash
# Enable BuildKit (better caching, parallel builds)
export DOCKER_BUILDKIT=1
docker build -t app .

# Or inline
DOCKER_BUILDKIT=1 docker build -t app .

# Permanently enable (add to ~/.bashrc)
echo 'export DOCKER_BUILDKIT=1' >> ~/.bashrc
```

### **Parallel Downloads:**
```dockerfile
# BuildKit enables parallel layer downloads
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install  # Downloads dependencies in parallel
```

### **Mount Caches:**
```dockerfile
# Cache package manager downloads
FROM python:3.11-slim
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r requirements.txt

FROM node:18-alpine
RUN --mount=type=cache,target=/root/.npm \
    npm install

FROM golang:1.20
RUN --mount=type=cache,target=/go/pkg/mod \
    go mod download
```

---

## 🎯 Complete Optimized Example

### **Node.js Production:**
```dockerfile
# ===================
# Stage 1: Dependencies
# ===================
FROM node:18-alpine AS deps
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install production dependencies only
RUN npm ci --only=production && \
    npm cache clean --force

# ===================
# Stage 2: Build
# ===================
FROM node:18-alpine AS builder
WORKDIR /app

# Copy package files
COPY package.json package-lock.json ./

# Install all dependencies (including dev)
RUN npm ci

# Copy source code
COPY src ./src
COPY tsconfig.json ./

# Build application
RUN npm run build

# ===================
# Stage 3: Production
# ===================
FROM node:18-alpine AS production

# Install security updates
RUN apk --no-cache upgrade

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Set working directory
WORKDIR /app

# Copy production dependencies
COPY --from=deps --chown=nodejs:nodejs /app/node_modules ./node_modules

# Copy built application
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist

# Copy necessary files
COPY --chown=nodejs:nodejs package.json ./

# Switch to non-root user
USER nodejs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s \
  CMD node healthcheck.js || exit 1

# Set environment
ENV NODE_ENV=production

# Start application
CMD ["node", "dist/index.js"]
```

---

## 🐛 Debugging Builds

### **Build with Progress:**
```bash
# See detailed output
DOCKER_BUILDKIT=1 docker build --progress=plain -t app .

# No cache (force rebuild)
docker build --no-cache -t app .

# Build specific stage
docker build --target builder -t app:builder .
```

### **Inspect Layers:**
```bash
# View image history
docker history my-image:latest

# Check layer sizes
docker history --no-trunc my-image:latest

# Use dive for interactive inspection
dive my-image:latest
```

### **Debug Failed Builds:**
```bash
# Build until failed step
docker build -t app . --target=failed-stage

# Or run intermediate container
docker run -it <intermediate-image-id> sh
```

---

## 📊 Before/After Comparison

### **Before Optimization:**
```dockerfile
FROM ubuntu:22.04
RUN apt update
RUN apt install -y python3 python3-pip
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python3", "app.py"]
# Image size: 800 MB
# Build time: 120 seconds
```

### **After Optimization:**
```dockerfile
# Multi-stage with Alpine
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip wheel --no-cache-dir --wheel-dir /app/wheels -r requirements.txt

FROM python:3.11-alpine
RUN apk --no-cache upgrade
RUN adduser -D appuser
WORKDIR /app
COPY --from=builder /app/wheels /wheels
RUN pip install --no-cache /wheels/*
COPY --chown=appuser:appuser . .
USER appuser
CMD ["python", "app.py"]
# Image size: 120 MB (85% reduction)
# Build time: 25 seconds (79% faster)
```

---

## 🎯 Quick Check: Do You Understand?

1. **Why chain RUN commands?**
   <details>
   <summary>Answer</summary>
   To reduce layers and image size; cleanup in same layer removes files
   </details>

2. **Why copy package.json before source code?**
   <details>
   <summary>Answer</summary>
   To cache dependency installation; only reinstalls if package.json changes
   </details>

3. **What's the benefit of multi-stage builds?**
   <details>
   <summary>Answer</summary>
   Smaller final image; build tools and intermediate files not in production image
   </details>

4. **Why use alpine images?**
   <details>
   <summary>Answer</summary>
   Minimal size (5-50 MB vs 100-1000 MB); fewer packages; smaller attack surface
   </details>

5. **Why avoid running as root?**
   <details>
   <summary>Answer</summary>
   Security; container escape could compromise host; principle of least privilege
   </details>

---

## 📝 Hands-on Exercise

**Optimize this Dockerfile:**

```bash
mkdir optimize-demo
cd optimize-demo

# Create inefficient Dockerfile
cat > Dockerfile.bad << 'EOF'
FROM ubuntu:22.04
RUN apt update
RUN apt install -y python3 python3-pip curl wget
RUN apt install -y gcc
COPY . /app
WORKDIR /app
RUN pip install flask
RUN pip install requests
CMD ["python3", "app.py"]
EOF

# Create optimized Dockerfile
cat > Dockerfile.good << 'EOF'
FROM python:3.11-alpine AS builder
WORKDIR /app
COPY requirements.txt .
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install --user -r requirements.txt

FROM python:3.11-alpine
RUN apk --no-cache upgrade && \
    adduser -D appuser
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY --chown=appuser:appuser app.py .
USER appuser
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
EOF

# Create requirements.txt
cat > requirements.txt << 'EOF'
flask==2.3.2
requests==2.31.0
EOF

# Create simple app
cat > app.py << 'EOF'
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Optimized Docker!</h1>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
EOF

# Build both
docker build -f Dockerfile.bad -t app:bad .
docker build -f Dockerfile.good -t app:good .

# Compare sizes
docker images | grep app

# Cleanup
cd ..
rm -rf optimize-demo
```

---

## 📝 Key Takeaways

✅ **Chain RUN commands to minimize layers**  
✅ **Order layers by change frequency**  
✅ **Use specific image tags, not 'latest'**  
✅ **Choose minimal base images (alpine, slim)**  
✅ **Use multi-stage builds for production**  
✅ **Create .dockerignore to reduce context**  
✅ **Never run containers as root**  
✅ **Never hardcode secrets in Dockerfile**  
✅ **Use BuildKit for better performance**  
✅ **Scan images for vulnerabilities**  

---

## 🚀 Next Steps

**Ready to compose multiple containers?**

**Next lesson:** [08 - Docker Volumes](08-docker-volumes.md) - Persistent data management

---

## 💡 Pro Tips

**Dockerfile Template:**
```dockerfile
# syntax=docker/dockerfile:1

# Build stage
FROM base:version AS builder
WORKDIR /app
COPY package-files ./
RUN install-deps
COPY source ./
RUN build-app

# Production stage
FROM base:slim-version
RUN security-updates && create-user
WORKDIR /app
COPY --from=builder --chown=user:group /app/dist ./
USER user
HEALTHCHECK CMD health-check
CMD ["start-app"]
```

**Build Automation:**
```bash
# Build script with best practices
#!/bin/bash
DOCKER_BUILDKIT=1 docker build \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg VCS_REF=$(git rev-parse --short HEAD) \
  --tag myapp:$(git describe --tags --always) \
  --tag myapp:latest \
  .
```

Master optimization - it's crucial for production! 🚀
