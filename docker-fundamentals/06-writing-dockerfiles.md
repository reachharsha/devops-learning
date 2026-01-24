---
render_with_liquid: false
---
# 06 - Writing Dockerfiles

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand Dockerfile syntax and instructions
- Build custom Docker images from Dockerfiles
- Use best practices for efficient images
- Implement multi-stage builds
- Optimize image size and build time
- Handle build arguments and environment variables
- Debug Dockerfile builds

---

## 📄 What is a Dockerfile?

**Dockerfile** = A text file containing instructions to build a Docker image.

**Real-World Analogy:**
```
Dockerfile  = Recipe with step-by-step instructions
Docker Build = Following the recipe to make the dish
Image        = The prepared dish ready to serve
Container    = Actually serving and eating the dish
```

**Basic Structure:**
```dockerfile
# Comment
INSTRUCTION arguments
INSTRUCTION arguments
...
```

---

## 🏗️ Basic Dockerfile

### **Simple Example:**
```dockerfile
# Start from base image
FROM ubuntu:22.04

# Set working directory
WORKDIR /app

# Copy files
COPY . /app

# Install dependencies
RUN apt update && apt install -y python3

# Define what to run
CMD ["python3", "app.py"]
```

### **Build and Run:**
```bash
# Build image
docker build -t my-app:v1.0 .
# -t = tag (name)
# . = build context (current directory)

# Run container
docker run my-app:v1.0
```

---

## 📚 Dockerfile Instructions

### **1. FROM - Base Image**
```dockerfile
# Official image
FROM ubuntu:22.04

# Specific version
FROM python:3.11

# Minimal image
FROM alpine:3.18

# Multi-platform
FROM --platform=linux/amd64 node:18

# Multiple FROM (multi-stage)
FROM node:18 AS builder
FROM nginx:alpine
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Specific version
FROM python:3.11-slim

# ❌ BAD - Using 'latest'
FROM python:latest

# ✅ GOOD - Minimal base
FROM python:3.11-alpine

# ❌ BAD - Large base
FROM python:3.11
```

---

### **2. WORKDIR - Set Working Directory**
```dockerfile
# Set working directory
WORKDIR /app

# All subsequent commands run in /app
COPY . .  # Copies to /app
RUN ls    # Lists /app

# Create nested directories
WORKDIR /app/src/components
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Use WORKDIR
WORKDIR /app
COPY . .

# ❌ BAD - Using cd
RUN cd /app
COPY . /app
```

---

### **3. COPY - Copy Files**
```dockerfile
# Copy file
COPY app.py /app/

# Copy directory
COPY src/ /app/src/

# Copy multiple files
COPY package.json package-lock.json /app/

# Copy with wildcards
COPY *.py /app/

# Copy everything
COPY . /app/

# Copy with ownership
COPY --chown=user:group app.py /app/
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Copy only needed files
COPY package*.json ./
RUN npm install
COPY src/ ./src/

# ❌ BAD - Copy everything early
COPY . .
RUN npm install
```

---

### **4. ADD - Advanced Copy**
```dockerfile
# Same as COPY
ADD app.py /app/

# Auto-extract tar archives
ADD archive.tar.gz /app/

# Download from URL (not recommended)
ADD https://example.com/file.txt /app/
```

**When to Use:**
```dockerfile
# ✅ Use ADD for tar extraction
ADD configs.tar.gz /etc/

# ✅ Use COPY for everything else
COPY app.py /app/
```

---

### **5. RUN - Execute Commands**
```dockerfile
# Shell form (runs in /bin/sh)
RUN apt update && apt install -y curl

# Exec form (doesn't use shell)
RUN ["apt", "update"]

# Multiple commands
RUN apt update && \
    apt install -y curl wget && \
    rm -rf /var/lib/apt/lists/*

# Chain commands
RUN command1 && \
    command2 && \
    command3
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Chain commands (fewer layers)
RUN apt update && \
    apt install -y curl wget vim && \
    apt clean && \
    rm -rf /var/lib/apt/lists/*

# ❌ BAD - Multiple RUN (more layers)
RUN apt update
RUN apt install -y curl
RUN apt install -y wget
RUN apt clean
```

---

### **6. CMD - Default Command**
```dockerfile
# Exec form (preferred)
CMD ["python3", "app.py"]

# Shell form
CMD python3 app.py

# With ENTRYPOINT
ENTRYPOINT ["python3"]
CMD ["app.py"]
```

**Important:**
- Only **one CMD** per Dockerfile (last one wins)
- Can be overridden: `docker run image other-command`

---

### **7. ENTRYPOINT - Main Executable**
```dockerfile
# Exec form
ENTRYPOINT ["python3", "app.py"]

# With CMD for default args
ENTRYPOINT ["python3"]
CMD ["app.py"]

# Now you can:
# docker run image          → python3 app.py
# docker run image test.py  → python3 test.py
```

**ENTRYPOINT vs CMD:**
```dockerfile
# CMD - Easy to override
CMD ["python3", "app.py"]
# docker run image bash → runs bash

# ENTRYPOINT - Hard to override
ENTRYPOINT ["python3", "app.py"]
# docker run image bash → runs python3 app.py (bash ignored)

# BEST - Combine both
ENTRYPOINT ["python3"]
CMD ["app.py"]
# docker run image → python3 app.py
# docker run image test.py → python3 test.py
```

---

### **8. ENV - Environment Variables**
```dockerfile
# Set single variable
ENV NODE_ENV=production

# Set multiple variables
ENV NODE_ENV=production \
    PORT=3000 \
    DB_HOST=localhost

# Use in subsequent commands
ENV APP_HOME=/app
WORKDIR $APP_HOME
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Group related vars
ENV NODE_ENV=production \
    PORT=3000 \
    LOG_LEVEL=info

# ✅ GOOD - Use ARG for build-time
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}
```

---

### **9. EXPOSE - Document Ports**
```dockerfile
# Expose single port
EXPOSE 80

# Expose multiple ports
EXPOSE 80 443

# Expose with protocol
EXPOSE 8080/tcp
EXPOSE 53/udp

# Expose range
EXPOSE 8000-8010
```

**Important:**
- EXPOSE is **documentation only**
- Doesn't actually publish ports
- Still need `-p` when running container

---

### **10. VOLUME - Mount Points**
```dockerfile
# Create volume mount point
VOLUME /data

# Multiple volumes
VOLUME ["/data", "/logs", "/config"]

# Example
VOLUME /var/lib/mysql  # MySQL data
VOLUME /var/log        # Logs
```

**Usage:**
```bash
# Dockerfile has: VOLUME /data
docker run -v my-volume:/data image  # Uses named volume
docker run image                     # Creates anonymous volume
```

---

### **11. USER - Set User**
```dockerfile
# Run as specific user
USER username

# Create user first
RUN useradd -m -s /bin/bash appuser
USER appuser

# Switch to root, then back
USER root
RUN apt install -y package
USER appuser
```

**Security Best Practice:**
```dockerfile
# ✅ GOOD - Don't run as root
FROM node:18-alpine
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
WORKDIR /app
COPY --chown=appuser:appgroup . .
CMD ["node", "app.js"]

# ❌ BAD - Running as root
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]  # Runs as root!
```

---

### **12. ARG - Build Arguments**
```dockerfile
# Define build argument
ARG VERSION=1.0

# Use in FROM
ARG NODE_VERSION=18
FROM node:${NODE_VERSION}

# Use in RUN
ARG INSTALL_DEV=false
RUN if [ "$INSTALL_DEV" = "true" ]; then npm install; fi

# With default value
ARG PORT=8080
EXPOSE $PORT
```

**Build with Arguments:**
```bash
# Use default
docker build -t app .

# Override ARG
docker build -t app --build-arg NODE_VERSION=16 .
docker build -t app --build-arg INSTALL_DEV=true .
```

**ARG vs ENV:**
```dockerfile
# ARG - Build time only
ARG BUILD_VERSION=1.0
RUN echo $BUILD_VERSION  # Available
# Not available in running container

# ENV - Build time + Runtime
ENV APP_VERSION=1.0
RUN echo $APP_VERSION      # Available
CMD echo $APP_VERSION      # Available in container
```

---

### **13. LABEL - Metadata**
```dockerfile
# Add metadata
LABEL version="1.0"
LABEL description="My application"
LABEL maintainer="you@example.com"

# Multiple labels
LABEL version="1.0" \
      description="My app" \
      maintainer="you@example.com"
```

**View Labels:**
```bash
docker inspect image --format='{ {json .Config.Labels} }'
```

---

### **14. HEALTHCHECK - Container Health**
```dockerfile
# Basic health check
HEALTHCHECK CMD curl -f http://localhost/ || exit 1

# With options
HEALTHCHECK --interval=30s \
            --timeout=3s \
            --start-period=5s \
            --retries=3 \
            CMD curl -f http://localhost/ || exit 1

# Disable inherited health check
HEALTHCHECK NONE
```

---

## 🎯 Complete Example

### **Node.js Application:**
```dockerfile
# Use official Node.js image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files first (for caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY src/ ./src/
COPY public/ ./public/

# Create non-root user
RUN addgroup -S appgroup && \
    adduser -S appuser -G appgroup && \
    chown -R appuser:appgroup /app

# Switch to non-root user
USER appuser

# Expose port
EXPOSE 3000

# Set environment
ENV NODE_ENV=production

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD node healthcheck.js

# Start application
CMD ["node", "src/index.js"]
```

---

### **Python Application:**
```dockerfile
# Use Python slim image
FROM python:3.11-slim

# Set environment variables
ENV PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1 \
    PIP_NO_CACHE_DIR=1

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements first
COPY requirements.txt .

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create non-root user
RUN useradd -m -s /bin/bash appuser && \
    chown -R appuser:appuser /app
USER appuser

# Expose port
EXPOSE 8000

# Run application
CMD ["python", "app.py"]
```

---

## 🏗️ Building Images

### **Basic Build:**
```bash
# Build from current directory
docker build -t myapp:v1.0 .

# Build from specific Dockerfile
docker build -f Dockerfile.dev -t myapp:dev .

# Build with no cache
docker build --no-cache -t myapp:v1.0 .

# Build with build args
docker build --build-arg VERSION=2.0 -t myapp:v2.0 .
```

### **Build Context:**
```bash
# Current directory as context
docker build -t app .

# Specific directory as context
docker build -t app /path/to/context

# URL as context
docker build -t app https://github.com/user/repo.git

# Tar file as context
docker build -t app - < context.tar.gz
```

### **View Build Progress:**
```bash
# Default output
docker build -t app .

# Progress plain (no color)
docker build --progress=plain -t app .

# BuildKit output (better)
DOCKER_BUILDKIT=1 docker build -t app .
```

---

## 📝 Hands-on Exercise

**Create a simple web server:**

```bash
# 1. Create project directory
mkdir my-web-app
cd my-web-app

# 2. Create app file
cat > app.py << 'EOF'
from http.server import HTTPServer, SimpleHTTPRequestHandler

class MyHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        self.wfile.write(b'<h1>Hello from Docker!</h1>')

httpd = HTTPServer(('0.0.0.0', 8000), MyHandler)
print('Server running on port 8000...')
httpd.serve_forever()
EOF

# 3. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-slim

WORKDIR /app

COPY app.py .

RUN useradd -m appuser && \
    chown -R appuser:appuser /app

USER appuser

EXPOSE 8000

HEALTHCHECK CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000')" || exit 1

CMD ["python", "app.py"]
EOF

# 4. Build image
docker build -t my-web-app:v1.0 .

# 5. Run container
docker run -d -p 8080:8000 --name web my-web-app:v1.0

# 6. Test
curl http://localhost:8080

# 7. Check health
docker inspect web --format='{ {.State.Health.Status} }'

# 8. Cleanup
docker stop web
docker rm web
cd ..
rm -rf my-web-app
```

---

## 📝 Key Takeaways

✅ **Dockerfile defines image build instructions**  
✅ **FROM sets the base image**  
✅ **WORKDIR sets working directory**  
✅ **COPY/ADD copy files into image**  
✅ **RUN executes commands during build**  
✅ **CMD sets default command**  
✅ **ENTRYPOINT sets main executable**  
✅ **ENV sets environment variables**  
✅ **EXPOSE documents ports (doesn't publish)**  
✅ **USER sets execution user (security!)**  

---

## 🚀 Next Steps

**Learn optimization techniques:**

**Next lesson:** [07 - Dockerfile Best Practices](07-dockerfile-best-practices.md) - Optimize your images

Dockerfiles are the heart of Docker - practice writing them daily! 🐳
