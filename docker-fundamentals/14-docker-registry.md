---
render_with_liquid: false
---
# 14 - Docker Registry

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand Docker registries and repositories
- Use Docker Hub effectively
- Set up private Docker registries
- Push and pull images from registries
- Configure registry authentication
- Implement registry security
- Use cloud registries (AWS ECR, GCP GCR, Azure ACR)

---

## 📦 What is a Docker Registry?

**Docker Registry** = A service that stores and distributes Docker images.

### **Registry Hierarchy:**
```
Registry (docker.io)
  └── Repository (library/nginx)
       ├── Tag: latest
       ├── Tag: 1.25
       ├── Tag: alpine
       └── Tag: 1.25-alpine
```

### **Image Naming:**
```
[registry]/[namespace]/[repository]:[tag]

Examples:
docker.io/library/nginx:latest          # Docker Hub (official)
docker.io/username/myapp:v1.0           # Docker Hub (user)
ghcr.io/username/myapp:latest           # GitHub Container Registry
localhost:5000/myapp:dev                # Local registry
myregistry.com:5000/team/app:prod       # Private registry
```

---

## 🌍 Docker Hub

### **Public Repository:**
```bash
# Login to Docker Hub
docker login
# Username: yourusername
# Password: yourpassword

# Tag image for Docker Hub
docker tag myapp:latest yourusername/myapp:latest
docker tag myapp:latest yourusername/myapp:v1.0

# Push to Docker Hub
docker push yourusername/myapp:latest
docker push yourusername/myapp:v1.0

# Pull from Docker Hub
docker pull yourusername/myapp:latest

# Search Docker Hub
docker search nginx
docker search --filter stars=1000 nginx
```

### **Private Repository:**
```bash
# Create private repo on hub.docker.com

# Push to private repo
docker tag myapp:latest yourusername/private-app:latest
docker push yourusername/private-app:latest

# Pull requires authentication
docker login
docker pull yourusername/private-app:latest
```

### **Automated Builds:**
```yaml
# Link GitHub/Bitbucket repository
# Docker Hub automatically builds on push

# Configure build rules:
Source: main
Docker Tag: latest
Dockerfile: Dockerfile

Source: /^v[0-9.]+$/
Docker Tag: {sourceref}
Dockerfile: Dockerfile.prod
```

---

## 🏢 Private Registry

### **Run Local Registry:**
```bash
# Start registry container
docker run -d \
  -p 5000:5000 \
  --name registry \
  --restart=always \
  registry:2

# Test registry
curl http://localhost:5000/v2/_catalog

# Push image to local registry
docker tag myapp:latest localhost:5000/myapp:latest
docker push localhost:5000/myapp:latest

# Pull from local registry
docker pull localhost:5000/myapp:latest

# List images in registry
curl http://localhost:5000/v2/_catalog
# {"repositories":["myapp"]}

# List tags
curl http://localhost:5000/v2/myapp/tags/list
# {"name":"myapp","tags":["latest","v1.0"]}
```

### **Persistent Storage:**
```bash
# Create volume for registry data
docker volume create registry-data

# Run with volume
docker run -d \
  -p 5000:5000 \
  --name registry \
  --restart=always \
  -v registry-data:/var/lib/registry \
  registry:2
```

### **With Docker Compose:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
    volumes:
      - registry-data:/data
    restart: unless-stopped

volumes:
  registry-data:
```

---

## 🔐 Secure Registry

### **Basic Authentication:**
```bash
# Create password file
mkdir -p auth
docker run --rm \
  --entrypoint htpasswd \
  httpd:2 -Bbn username password > auth/htpasswd

# Run registry with auth
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  -v registry-data:/var/lib/registry \
  registry:2

# Login to registry
docker login localhost:5000
# Username: username
# Password: password

# Push image
docker push localhost:5000/myapp:latest
```

### **TLS/HTTPS:**
```bash
# Generate self-signed certificate
mkdir -p certs
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout certs/domain.key -x509 -days 365 \
  -out certs/domain.crt \
  -subj "/CN=myregistry.local"

# Run registry with TLS
docker run -d \
  -p 5000:5000 \
  --name registry \
  -v $(pwd)/certs:/certs \
  -v registry-data:/var/lib/registry \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
  registry:2

# Use registry with custom cert
# Copy cert to Docker
sudo mkdir -p /etc/docker/certs.d/myregistry.local:5000
sudo cp certs/domain.crt /etc/docker/certs.d/myregistry.local:5000/ca.crt

# Or add to system trust store
sudo cp certs/domain.crt /usr/local/share/ca-certificates/
sudo update-ca-certificates
```

### **Complete Secure Setup:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      # TLS
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
      
      # Authentication
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      
      # Storage
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
      
      # Delete enabled
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - ./certs:/certs:ro
      - ./auth:/auth:ro
      - registry-data:/data
    restart: unless-stopped

volumes:
  registry-data:
```

---

## 🖼️ Registry UI

### **Docker Registry UI:**
```yaml
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_STORAGE_FILESYSTEM_ROOTDIRECTORY: /data
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - registry-data:/data
    restart: unless-stopped

  ui:
    image: joxit/docker-registry-ui:latest
    ports:
      - "8080:80"
    environment:
      REGISTRY_TITLE: My Private Registry
      REGISTRY_URL: http://registry:5000
      DELETE_IMAGES: "true"
      SHOW_CONTENT_DIGEST: "true"
      NGINX_PROXY_PASS_URL: http://registry:5000
      SINGLE_REGISTRY: "true"
    depends_on:
      - registry

volumes:
  registry-data:
```

### **Harbor (Enterprise Registry):**
```bash
# Download Harbor installer
wget https://github.com/goharbor/harbor/releases/download/v2.9.0/harbor-offline-installer-v2.9.0.tgz
tar xzvf harbor-offline-installer-v2.9.0.tgz
cd harbor

# Configure Harbor
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
# Edit: hostname, https, admin password

# Install
sudo ./install.sh

# Access Harbor
# https://your-harbor-domain
# Username: admin
# Password: (from harbor.yml)
```

---

## ☁️ Cloud Registries

### **Amazon ECR (Elastic Container Registry):**
```bash
# Install AWS CLI
aws configure

# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.us-east-1.amazonaws.com

# Create repository
aws ecr create-repository --repository-name myapp

# Tag image
docker tag myapp:latest \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Push image
docker push \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# Pull image
docker pull \
  123456789012.dkr.ecr.us-east-1.amazonaws.com/myapp:latest

# List images
aws ecr list-images --repository-name myapp

# Delete image
aws ecr batch-delete-image \
  --repository-name myapp \
  --image-ids imageTag=latest
```

### **Google GCR (Container Registry):**
```bash
# Install gcloud CLI
gcloud auth configure-docker

# Tag image
docker tag myapp:latest \
  gcr.io/my-project-id/myapp:latest

# Push image
docker push gcr.io/my-project-id/myapp:latest

# Pull image
docker pull gcr.io/my-project-id/myapp:latest

# List images
gcloud container images list

# Delete image
gcloud container images delete \
  gcr.io/my-project-id/myapp:latest
```

### **Azure ACR (Container Registry):**
```bash
# Install Azure CLI
az login

# Create registry
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Basic

# Login to ACR
az acr login --name myregistry

# Tag image
docker tag myapp:latest \
  myregistry.azurecr.io/myapp:latest

# Push image
docker push myregistry.azurecr.io/myapp:latest

# Pull image
docker pull myregistry.azurecr.io/myapp:latest

# List repositories
az acr repository list --name myregistry

# List tags
az acr repository show-tags \
  --name myregistry \
  --repository myapp
```

### **GitHub Container Registry:**
```bash
# Create Personal Access Token (PAT)
# Settings → Developer settings → Personal access tokens
# Scope: write:packages, read:packages, delete:packages

# Login to GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag image
docker tag myapp:latest \
  ghcr.io/username/myapp:latest

# Push image
docker push ghcr.io/username/myapp:latest

# Pull image
docker pull ghcr.io/username/myapp:latest

# Make package public
# Go to package settings on GitHub
```

---

## 🗑️ Managing Registry Images

### **Delete Images from Registry:**
```bash
# List repositories
curl http://localhost:5000/v2/_catalog

# List tags
curl http://localhost:5000/v2/myapp/tags/list

# Get digest
curl -I -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
  http://localhost:5000/v2/myapp/manifests/latest

# Delete by digest
curl -X DELETE \
  http://localhost:5000/v2/myapp/manifests/sha256:abc123...

# Run garbage collection
docker exec registry bin/registry garbage-collect \
  /etc/docker/registry/config.yml
```

### **Cleanup Script:**
```bash
#!/bin/bash
# cleanup-registry.sh

REGISTRY="localhost:5000"
REPO="myapp"

# Get all tags
TAGS=$(curl -s http://${REGISTRY}/v2/${REPO}/tags/list | jq -r '.tags[]')

# Keep only last 5 tags
echo "$TAGS" | head -n -5 | while read tag; do
  # Get digest
  DIGEST=$(curl -I -s -H "Accept: application/vnd.docker.distribution.manifest.v2+json" \
    http://${REGISTRY}/v2/${REPO}/manifests/${tag} | \
    grep Docker-Content-Digest | awk '{print $2}' | tr -d '\r')
  
  # Delete
  curl -X DELETE http://${REGISTRY}/v2/${REPO}/manifests/${DIGEST}
  echo "Deleted ${REPO}:${tag}"
done

# Garbage collect
docker exec registry bin/registry garbage-collect /etc/docker/registry/config.yml
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the default Docker registry?**
   <details>
   <summary>Answer</summary>
   Docker Hub (docker.io)
   </details>

2. **How do you run a local registry?**
   <details>
   <summary>Answer</summary>
   docker run -d -p 5000:5000 registry:2
   </details>

3. **What's the format for tagging images for a registry?**
   <details>
   <summary>Answer</summary>
   registry/namespace/repository:tag
   </details>

4. **How do you enable image deletion in registry?**
   <details>
   <summary>Answer</summary>
   Set REGISTRY_STORAGE_DELETE_ENABLED=true
   </details>

5. **Why use a private registry?**
   <details>
   <summary>Answer</summary>
   Security, privacy, control, faster pulls in private network, cost savings
   </details>

---

## 📝 Hands-on Exercise

**Set up complete private registry:**

```bash
# 1. Create project directory
mkdir my-registry
cd my-registry

# 2. Create auth credentials
mkdir auth
docker run --rm \
  --entrypoint htpasswd \
  httpd:2 -Bbn admin secret123 > auth/htpasswd

# 3. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  registry:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_STORAGE_DELETE_ENABLED: "true"
    volumes:
      - ./auth:/auth:ro
      - registry-data:/var/lib/registry
    restart: unless-stopped

  ui:
    image: joxit/docker-registry-ui:latest
    ports:
      - "8080:80"
    environment:
      REGISTRY_TITLE: My Private Registry
      REGISTRY_URL: http://registry:5000
      DELETE_IMAGES: "true"
    depends_on:
      - registry

volumes:
  registry-data:
EOF

# 4. Start registry
docker-compose up -d

# 5. Login to registry
docker login localhost:5000
# Username: admin
# Password: secret123

# 6. Build and push test image
docker pull alpine:latest
docker tag alpine:latest localhost:5000/test/alpine:latest
docker push localhost:5000/test/alpine:latest

# 7. Verify in UI
echo "Open http://localhost:8080 in browser"

# 8. Test pull
docker rmi localhost:5000/test/alpine:latest
docker pull localhost:5000/test/alpine:latest

# 9. Check catalog
curl http://admin:secret123@localhost:5000/v2/_catalog

# 10. Cleanup
docker-compose down -v
cd ..
rm -rf my-registry
```

---

## 📝 Key Takeaways

✅ **Registries store and distribute Docker images**  
✅ **Docker Hub is the default public registry**  
✅ **Private registries provide security and control**  
✅ **Use authentication to secure registries**  
✅ **TLS/HTTPS encrypts registry traffic**  
✅ **Cloud registries integrate with cloud platforms**  
✅ **Tag images properly for registry push**  
✅ **Enable deletion for image cleanup**  
✅ **Harbor provides enterprise features**  
✅ **Regular cleanup prevents storage bloat**  

---

## 🚀 Next Steps

**Deploy to production:**

**Next lesson:** [15 - Production Deployment](15-production-deployment.md) - Deploy containers to production

---

## 💡 Pro Tips

**Registry Best Practices:**
```bash
# Use semantic versioning
docker tag myapp:latest registry.local/myapp:1.2.3
docker tag myapp:latest registry.local/myapp:1.2
docker tag myapp:latest registry.local/myapp:1
docker tag myapp:latest registry.local/myapp:latest

# Tag with git commit
docker tag myapp:latest registry.local/myapp:git-$(git rev-parse --short HEAD)

# Tag with date
docker tag myapp:latest registry.local/myapp:$(date +%Y%m%d)
```

**CI/CD Integration:**
```yaml
# .github/workflows/build-push.yml
name: Build and Push

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Login to Registry
        run: echo "${{ secrets.REGISTRY_PASSWORD }}" | 
          docker login registry.local -u admin --password-stdin
      
      - name: Build and Push
        run: |
          docker build -t registry.local/myapp:${{ github.sha }} .
          docker push registry.local/myapp:${{ github.sha }}
          
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            docker tag registry.local/myapp:${{ github.sha }} registry.local/myapp:latest
            docker push registry.local/myapp:latest
          fi
```

**Mirror Docker Hub:**
```yaml
# Use registry as pull-through cache
version: '3.8'

services:
  registry-mirror:
    image: registry:2
    ports:
      - "5000:5000"
    environment:
      REGISTRY_PROXY_REMOTEURL: https://registry-1.docker.io
    volumes:
      - mirror-cache:/var/lib/registry

volumes:
  mirror-cache:
```

Registries are essential infrastructure - set them up right! 📦
