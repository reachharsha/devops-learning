---
render_with_liquid: false
---
# 13 - Container Security

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand container security fundamentals
- Implement least privilege principles
- Scan images for vulnerabilities
- Use Docker secrets safely
- Configure security options
- Apply security best practices
- Harden container deployments

---

## 🔒 Container Security Fundamentals

### **The Security Model:**
```
┌────────────────────────────────────┐
│     Application Code               │  ← Your responsibility
├────────────────────────────────────┤
│     Container Runtime              │  ← Shared responsibility
├────────────────────────────────────┤
│     Docker Daemon                  │  ← Docker's responsibility
├────────────────────────────────────┤
│     Host OS Kernel                 │  ← System responsibility
└────────────────────────────────────┘
```

### **Attack Surface:**
```bash
# Potential vulnerabilities:
1. Vulnerable base images
2. Exposed secrets in images
3. Running as root
4. Excessive capabilities
5. Unrestricted network access
6. Exposed Docker socket
7. Outdated dependencies
8. Misconfigured permissions
```

---

## 👤 Never Run as Root

### **The Problem:**
```dockerfile
# ❌ DANGEROUS - Running as root
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
# Container runs as root (UID 0)
# If compromised, attacker has root access!
```

### **The Solution:**
```dockerfile
# ✅ SECURE - Non-root user
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy files with correct ownership
COPY --chown=nodejs:nodejs package*.json ./
RUN npm ci --only=production

COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

EXPOSE 3000
CMD ["node", "app.js"]
```

### **Verify User:**
```bash
# Check running user
docker exec container whoami
docker exec container id

# Inspect container
docker inspect container --format='{ {.Config.User} }'
```

---

## 🔍 Image Vulnerability Scanning

### **Using Docker Scout (Built-in):**
```bash
# Scan local image
docker scout cves nginx:latest

# Scan with detailed output
docker scout cves --only-package-type os nginx:latest

# Compare with another image
docker scout compare nginx:latest --to nginx:alpine

# Quick health check
docker scout quickview nginx:latest

# Get recommendations
docker scout recommendations nginx:latest
```

### **Using Trivy:**
```bash
# Install Trivy
# macOS
brew install aquasecurity/trivy/trivy

# Linux
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo "deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt update && sudo apt install trivy

# Scan image
trivy image nginx:latest

# Only show HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL nginx:latest

# Scan and fail on vulnerabilities
trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest

# Scan Dockerfile
trivy config Dockerfile

# Scan file system
trivy fs .

# Generate report
trivy image --format json --output report.json nginx:latest
```

### **Using Snyk:**
```bash
# Install Snyk
npm install -g snyk

# Authenticate
snyk auth

# Scan image
snyk container test nginx:latest

# Monitor image
snyk container monitor nginx:latest

# Test Dockerfile
snyk container test --file=Dockerfile nginx:latest
```

### **Automated Scanning in CI/CD:**
```yaml
# .github/workflows/security-scan.yml
name: Security Scan

on: [push, pull_request]

jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Run Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
      
      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'
```

---

## 🔐 Secrets Management

### **What NOT to Do:**
```dockerfile
# ❌ NEVER DO THIS!
FROM node:18-alpine
ENV API_KEY="sk-1234567890abcdef"  # Exposed in image!
ENV DB_PASSWORD="mypassword"        # Anyone can see this!
RUN echo "secret" > /app/secret.txt # Baked into layer!
```

```bash
# ❌ NEVER DO THIS!
docker run -e DB_PASSWORD="secret123" app  # Shows in docker inspect!
```

### **Docker Secrets (Swarm Mode):**
```bash
# Create secret
echo "mySecretPassword" | docker secret create db_password -

# From file
docker secret create db_password ./password.txt

# List secrets
docker secret ls

# Inspect secret (value hidden)
docker secret inspect db_password

# Use in service
docker service create \
  --name myapp \
  --secret db_password \
  myapp:latest

# Remove secret
docker secret rm db_password
```

### **Secrets in Dockerfile:**
```dockerfile
# Use build secrets (BuildKit)
# syntax=docker/dockerfile:1

FROM alpine:latest

# Secret available during build only
RUN --mount=type=secret,id=github_token \
    GITHUB_TOKEN=$(cat /run/secrets/github_token) && \
    # Use token to clone private repo
    echo "Token: $GITHUB_TOKEN"
```

```bash
# Build with secret
DOCKER_BUILDKIT=1 docker build \
  --secret id=github_token,src=./github_token.txt \
  -t myapp:latest .
```

### **Environment Variables at Runtime:**
```bash
# Better - pass at runtime
docker run --env-file .env myapp:latest

# .env file (add to .gitignore!)
DB_PASSWORD=secret123
API_KEY=sk-1234567890
```

### **Using Vault (Production):**
```yaml
# docker-compose.yml with Vault
version: '3.8'

services:
  vault:
    image: vault:latest
    cap_add:
      - IPC_LOCK
    environment:
      VAULT_DEV_ROOT_TOKEN_ID: myroot
    ports:
      - "8200:8200"

  app:
    image: myapp:latest
    environment:
      VAULT_ADDR: http://vault:8200
      VAULT_TOKEN: myroot
    command: >
      sh -c "
        export DB_PASSWORD=$(vault kv get -field=password secret/db)
        npm start
      "
    depends_on:
      - vault
```

---

## 🛡️ Security Options

### **Read-only Root Filesystem:**
```bash
# Make root filesystem read-only
docker run -d \
  --read-only \
  --tmpfs /tmp \
  --tmpfs /var/run \
  nginx:alpine

# In Compose
services:
  app:
    image: myapp:latest
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
```

### **Drop Capabilities:**
```bash
# Drop all capabilities, add only needed ones
docker run -d \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  nginx:alpine

# In Compose
services:
  app:
    image: myapp:latest
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
      - CHOWN
```

### **No New Privileges:**
```bash
# Prevent privilege escalation
docker run -d \
  --security-opt=no-new-privileges:true \
  myapp:latest

# In Compose
services:
  app:
    image: myapp:latest
    security_opt:
      - no-new-privileges:true
```

### **AppArmor/SELinux:**
```bash
# Use AppArmor profile
docker run -d \
  --security-opt apparmor=docker-default \
  myapp:latest

# Custom AppArmor profile
docker run -d \
  --security-opt apparmor=my-profile \
  myapp:latest

# SELinux label
docker run -d \
  --security-opt label=level:s0:c100,c200 \
  myapp:latest
```

---

## 🌐 Network Security

### **Isolate Networks:**
```yaml
version: '3.8'

services:
  # Public-facing web server
  web:
    image: nginx:alpine
    networks:
      - frontend
    ports:
      - "80:80"

  # Application server (both networks)
  app:
    image: myapp:latest
    networks:
      - frontend
      - backend

  # Database (backend only - isolated)
  db:
    image: postgres:15
    networks:
      - backend
    # No ports exposed!

networks:
  frontend:
  backend:
    internal: true  # No external access
```

### **Limit Port Exposure:**
```bash
# ❌ BAD - Expose to all interfaces
docker run -p 5432:5432 postgres

# ✅ GOOD - Expose to localhost only
docker run -p 127.0.0.1:5432:5432 postgres
```

### **Use Internal Networks:**
```bash
# Create internal network
docker network create --internal backend

# Containers can talk to each other, but not internet
docker run -d --network backend --name db postgres
docker run -d --network backend --name app myapp
```

---

## 📝 Content Trust

### **Enable Docker Content Trust:**
```bash
# Enable content trust
export DOCKER_CONTENT_TRUST=1

# Now only signed images can be pulled/run
docker pull nginx:latest

# Generate signing keys
docker trust key generate mykey

# Sign image
docker trust sign myimage:tag

# View signatures
docker trust inspect nginx:latest
```

---

## 🔒 Security Best Practices

### **1. Minimal Base Images:**
```dockerfile
# ✅ GOOD - Minimal attack surface
FROM alpine:3.18
FROM node:18-alpine
FROM python:3.11-slim

# ❌ BAD - Large attack surface
FROM ubuntu:latest
FROM node:18  # Full variant
```

### **2. Multi-stage Builds:**
```dockerfile
# Build stage - includes build tools
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage - minimal
FROM node:18-alpine
RUN addgroup -g 1001 nodejs && \
    adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
USER nodejs
CMD ["node", "dist/index.js"]
```

### **3. Security Scanning in Dockerfile:**
```dockerfile
FROM node:18-alpine

# Update packages
RUN apk update && apk upgrade

# Remove package manager cache
RUN rm -rf /var/cache/apk/*

# Rest of Dockerfile...
```

### **4. Secure Compose File:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    
    # Security options
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    
    # User
    user: "1001:1001"
    
    # Resources
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    
    # Tmpfs for writable dirs
    tmpfs:
      - /tmp
      - /var/run
    
    # Network isolation
    networks:
      - internal
    
    # Restart policy
    restart: unless-stopped

networks:
  internal:
    internal: true
```

---

## 🛠️ Docker Daemon Security

### **Secure Daemon Configuration:**
```json
// /etc/docker/daemon.json
{
  "icc": false,              // Disable inter-container communication
  "userns-remap": "default", // User namespace remapping
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  },
  "live-restore": true,
  "userland-proxy": false,
  "no-new-privileges": true
}
```

### **TLS for Remote Access:**
```bash
# Generate certificates
openssl genrsa -out ca-key.pem 4096
openssl req -new -x509 -days 365 -key ca-key.pem -sha256 -out ca.pem

# Configure daemon
# /etc/docker/daemon.json
{
  "tls": true,
  "tlscert": "/etc/docker/server-cert.pem",
  "tlskey": "/etc/docker/server-key.pem",
  "tlsverify": true,
  "tlscacert": "/etc/docker/ca.pem",
  "hosts": ["tcp://0.0.0.0:2376"]
}
```

---

## 🎯 Quick Check: Do You Understand?

1. **Why shouldn't containers run as root?**
   <details>
   <summary>Answer</summary>
   If compromised, attacker has root access; violates least privilege principle
   </details>

2. **What tool can scan Docker images for vulnerabilities?**
   <details>
   <summary>Answer</summary>
   Docker Scout, Trivy, Snyk, Clair, or Anchore
   </details>

3. **How should secrets be passed to containers?**
   <details>
   <summary>Answer</summary>
   At runtime via environment variables, Docker secrets, or secret management tools (Vault)
   </details>

4. **What does --read-only do?**
   <details>
   <summary>Answer</summary>
   Makes root filesystem read-only, preventing writes except to mounted volumes
   </details>

5. **Why use minimal base images?**
   <details>
   <summary>Answer</summary>
   Smaller attack surface, fewer vulnerabilities, smaller image size
   </details>

---

## 📝 Hands-on Exercise

**Create a secure container:**

```bash
# 1. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM python:3.11-alpine

# Update and upgrade
RUN apk update && apk upgrade && \
    rm -rf /var/cache/apk/*

# Create non-root user
RUN addgroup -g 1001 appgroup && \
    adduser -D -u 1001 -G appgroup appuser

WORKDIR /app

# Copy and install dependencies
COPY --chown=appuser:appgroup requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy app
COPY --chown=appuser:appgroup app.py .

# Switch to non-root user
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD python -c "import urllib.request; urllib.request.urlopen('http://localhost:8000/health')" || exit 1

CMD ["python", "app.py"]
EOF

# 2. Create simple app
cat > app.py << 'EOF'
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.send_header('Content-type', 'text/html')
        self.end_headers()
        if self.path == '/health':
            self.wfile.write(b'OK')
        else:
            self.wfile.write(b'Hello from secure container!')

httpd = HTTPServer(('0.0.0.0', 8000), Handler)
print('Server running on port 8000...')
httpd.serve_forever()
EOF

cat > requirements.txt << 'EOF'
# No dependencies needed for this example
EOF

# 3. Build image
docker build -t secure-app:latest .

# 4. Scan for vulnerabilities
trivy image secure-app:latest

# 5. Run securely
docker run -d \
  --name secure-app \
  --read-only \
  --tmpfs /tmp \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges:true \
  -p 127.0.0.1:8000:8000 \
  secure-app:latest

# 6. Verify user
docker exec secure-app whoami  # Should be "appuser"
docker exec secure-app id      # Should show UID 1001

# 7. Test
curl http://localhost:8000

# 8. Inspect security
docker inspect secure-app | grep -A 10 SecurityOpt

# 9. Cleanup
docker stop secure-app
docker rm secure-app
docker rmi secure-app:latest
rm Dockerfile app.py requirements.txt
```

---

## 📝 Key Takeaways

✅ **Never run containers as root**  
✅ **Scan images for vulnerabilities regularly**  
✅ **Never hardcode secrets in images**  
✅ **Use minimal base images (alpine, slim)**  
✅ **Drop unnecessary capabilities**  
✅ **Enable read-only root filesystem**  
✅ **Isolate networks appropriately**  
✅ **Keep images and dependencies updated**  
✅ **Use multi-stage builds for smaller images**  
✅ **Enable security options (no-new-privileges)**  

---

## 🚀 Next Steps

**Learn to manage your own registry:**

**Next lesson:** [14 - Docker Registry](14-docker-registry.md) - Private image storage

---

## 💡 Pro Tips

**Security Checklist:**
```bash
# Before deploying to production:
□ Image scanned for vulnerabilities
□ No secrets in image layers
□ Running as non-root user
□ Read-only root filesystem
□ Capabilities dropped
□ Resource limits set
□ Health checks configured
□ Network isolation implemented
□ Logging enabled
□ Regular updates scheduled
```

**Automated Security:**
```yaml
# .gitlab-ci.yml
security_scan:
  stage: test
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - trivy image --exit-code 1 --severity HIGH,CRITICAL $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - merge_requests
    - main
```

**Security Monitoring:**
```bash
# Use Falco for runtime security
docker run -d \
  --name falco \
  --privileged \
  -v /var/run/docker.sock:/host/var/run/docker.sock \
  -v /dev:/host/dev \
  -v /proc:/host/proc:ro \
  falcosecurity/falco
```

Security is not optional - make it a habit! 🔒
