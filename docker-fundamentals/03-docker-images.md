---
render_with_liquid: false
---
# 03 - Docker Images Deep Dive

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand what Docker images are
- Pull images from Docker Hub
- List and inspect images
- Search for images
- Manage image tags and versions
- Remove unused images
- Understand image layers and caching

---

## 📦 What are Docker Images?

**Docker Image** = A read-only template containing everything needed to run a container.

**Real-World Analogy:**
Think of an image like a **recipe**:
- Recipe (Image) → tells you how to make a dish
- Cooked dish (Container) → the actual running result
- From one recipe, you can make many dishes

**Image Contents:**
- Base operating system (Alpine, Ubuntu, etc.)
- Application code
- Runtime (Python, Node.js, Java, etc.)
- Dependencies and libraries
- Configuration files
- Environment variables
- Metadata

---

## 🏗️ Image Layers

Images are built in **layers** (like a cake):

```
Image: my-app:latest
┌──────────────────────────────────┐
│  Layer 5: CMD ["npm", "start"]  │  ← Your app command
├──────────────────────────────────┤
│  Layer 4: COPY . /app            │  ← Your code
├──────────────────────────────────┤
│  Layer 3: RUN npm install        │  ← Dependencies
├──────────────────────────────────┤
│  Layer 2: WORKDIR /app           │  ← Working directory
├──────────────────────────────────┤
│  Layer 1: FROM node:18-alpine    │  ← Base image
└──────────────────────────────────┘
```

**Why Layers Matter:**
- **Efficient storage** - Shared layers save disk space
- **Fast builds** - Unchanged layers are cached
- **Quick downloads** - Only download new layers

**Example:**
```
Image A: nginx:alpine     Image B: myapp:nginx
┌────────────────┐        ┌────────────────┐
│  nginx config  │        │  my app files  │
├────────────────┤        ├────────────────┤
│  nginx binary  │  ←───→ │  nginx binary  │  (Shared!)
├────────────────┤        ├────────────────┤
│  alpine base   │  ←───→ │  alpine base   │  (Shared!)
└────────────────┘        └────────────────┘
```

---

## 📥 Pulling Images

### **Basic Pull:**
```bash
# Pull latest version
docker pull nginx
# Same as: docker pull nginx:latest

# Pull specific version
docker pull nginx:1.25
docker pull nginx:alpine
docker pull postgres:15.3

# Pull from specific registry
docker pull docker.io/library/nginx:latest
```

### **Image Naming Convention:**
```
[registry]/[username]/[repository]:[tag]

Examples:
docker.io/library/nginx:latest
  └─┬──┘ └──┬──┘ └─┬──┘ └─┬──┘
  Registry User  Name   Tag

ghcr.io/username/myapp:v1.0.0
quay.io/organization/service:prod
localhost:5000/internal/tool:dev
```

### **Pull Multiple Architectures:**
```bash
# Pull for specific platform
docker pull --platform linux/amd64 nginx:alpine
docker pull --platform linux/arm64 nginx:alpine

# View available platforms
docker manifest inspect nginx:alpine
```

---

## 📋 Listing Images

### **List All Images:**
```bash
# Basic list
docker images

# Alternative command
docker image ls

# Output:
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    605c77e624dd   2 weeks ago    141MB
postgres      15        3c6e8c5fb9e8   3 weeks ago    379MB
redis         alpine    3900abf41552   1 month ago    30MB
```

### **Filter Images:**
```bash
# Show only specific repository
docker images nginx

# Show images with specific tag
docker images nginx:alpine

# Show all tags for a repository
docker images nginx

# Show image IDs only
docker images -q

# Show with digests
docker images --digests

# No truncation
docker images --no-trunc
```

### **Format Output:**
```bash
# Custom format
docker images --format "table { {.Repository} }\t{ {.Tag} }\t{ {.Size} }"

# JSON format
docker images --format json

# Only show repository and tag
docker images --format "{ {.Repository} }:{ {.Tag} }"
```

---

## 🔍 Inspecting Images

### **Get Detailed Information:**
```bash
# Inspect image
docker image inspect nginx:alpine

# Get specific information
docker image inspect nginx:alpine --format='{ {.Architecture} }'
docker image inspect nginx:alpine --format='{ {.Os} }'
docker image inspect nginx:alpine --format='{ {.Size} }'
```

**Inspect Output** (JSON format):
```json
{
  "Id": "sha256:605c77e624dd...",
  "RepoTags": ["nginx:alpine"],
  "Created": "2024-01-15T10:30:45Z",
  "Architecture": "amd64",
  "Os": "linux",
  "Size": 41000000,
  "Config": {
    "Env": ["PATH=/usr/local/sbin:/usr/local/bin..."],
    "Cmd": ["nginx", "-g", "daemon off;"],
    "ExposedPorts": {"80/tcp": {}}
  },
  "RootFS": {
    "Type": "layers",
    "Layers": [...]
  }
}
```

### **View Image History:**
```bash
# See all layers and commands
docker image history nginx:alpine

# Output:
IMAGE          CREATED       CREATED BY                                      SIZE
605c77e624dd   2 weeks ago   CMD ["nginx" "-g" "daemon off;"]               0B
<missing>      2 weeks ago   EXPOSE 80                                       0B
<missing>      2 weeks ago   RUN /bin/sh -c apk add --no-cache nginx         12.5MB
<missing>      2 weeks ago   FROM alpine:3.18                                7.05MB
```

### **Show Image Layers:**
```bash
# Use dive tool (install separately)
dive nginx:alpine

# Or inspect layers
docker image inspect nginx:alpine --format='{ {.RootFS.Layers} }'
```

---

## 🔎 Searching Images

### **Search Docker Hub:**
```bash
# Basic search
docker search nginx

# Output:
NAME                  DESCRIPTION                                     STARS    OFFICIAL
nginx                 Official build of Nginx.                        18000    [OK]
nginxinc/nginx-unpri  Unprivileged NGINX Dockerfiles                 100
bitnami/nginx         Bitnami nginx Docker Image                     150

# Limit results
docker search --limit 5 nginx

# Filter by stars
docker search --filter stars=1000 nginx

# Filter official images only
docker search --filter is-official=true nginx

# No truncation
docker search --no-trunc nginx
```

### **Search on Docker Hub Website:**
```
Visit: https://hub.docker.com
- More detailed information
- README documentation
- Tags and versions
- Usage examples
- Security scans
```

---

## 🏷️ Image Tagging

### **Tag Images:**
```bash
# Tag an existing image
docker tag nginx:alpine my-nginx:v1.0
docker tag nginx:alpine localhost:5000/nginx:prod

# Tag during build
docker build -t myapp:v1.0 .
docker build -t myapp:latest -t myapp:v1.0 .

# Tag with multiple names
docker tag myapp:v1.0 myapp:stable
docker tag myapp:v1.0 myapp:production
```

### **Tagging Best Practices:**
```bash
# ❌ BAD - Using only 'latest'
docker tag myapp:latest

# ✅ GOOD - Semantic versioning
docker tag myapp:1.2.3
docker tag myapp:1.2    # Major.minor
docker tag myapp:1      # Major

# ✅ GOOD - Environment tags
docker tag myapp:dev
docker tag myapp:staging
docker tag myapp:prod

# ✅ GOOD - Git commit hash
docker tag myapp:git-a1b2c3d
docker tag myapp:sha-${GIT_COMMIT}

# ✅ GOOD - Date tags
docker tag myapp:2024-01-24
docker tag myapp:2024-01
```

---

## 🗑️ Removing Images

### **Remove Single Image:**
```bash
# By name:tag
docker rmi nginx:alpine
docker image rm nginx:alpine

# By image ID
docker rmi 605c77e624dd

# Force remove (even if container exists)
docker rmi -f nginx:alpine
```

### **Remove Multiple Images:**
```bash
# Remove multiple by name
docker rmi nginx:alpine redis:alpine postgres:15

# Remove all images for a repository
docker rmi $(docker images nginx -q)

# Remove all images
docker rmi $(docker images -q)
```

### **Remove Dangling Images:**
```bash
# List dangling images (untagged)
docker images -f dangling=true

# Remove dangling images
docker image prune

# Remove with confirmation prompt
docker image prune -a

# Remove without confirmation
docker image prune -a -f
```

### **Remove by Pattern:**
```bash
# Remove images older than 24h
docker image prune -a --filter "until=24h"

# Remove by label
docker image prune --filter "label=version=1.0"
```

---

## 💾 Saving and Loading Images

### **Save Image to File:**
```bash
# Save single image
docker save nginx:alpine > nginx-alpine.tar
docker save nginx:alpine -o nginx-alpine.tar

# Save multiple images
docker save -o images.tar nginx:alpine redis:alpine

# Save and compress
docker save nginx:alpine | gzip > nginx-alpine.tar.gz
```

### **Load Image from File:**
```bash
# Load image
docker load < nginx-alpine.tar
docker load -i nginx-alpine.tar

# Load compressed image
gunzip -c nginx-alpine.tar.gz | docker load
```

### **Export/Import (Containers, not Images):**
```bash
# Export running container to tar
docker export container_name > container.tar

# Import tar as image
docker import container.tar myimage:tag
```

**Difference:**
- **save/load** - Preserves layers and metadata
- **export/import** - Flattens to single layer, loses history

---

## 📊 Image Storage and Disk Usage

### **Check Disk Usage:**
```bash
# Overall Docker disk usage
docker system df

# Output:
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          10        5         2.5GB     1.2GB (48%)
Containers      15        3         500MB     400MB (80%)
Local Volumes   5         2         1GB       800MB (80%)
Build Cache     50        0         3GB       3GB (100%)

# Detailed view
docker system df -v
```

### **View Image Size:**
```bash
# Size of each image
docker images --format "{ {.Repository} }:{ {.Tag} } - { {.Size} }"

# Sort by size
docker images --format "table { {.Repository} }\t{ {.Tag} }\t{ {.Size} }" | sort -k 3 -h
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is a Docker image?**
   <details>
   <summary>Answer</summary>
   A read-only template containing application code, runtime, dependencies, and configuration needed to run a container
   </details>

2. **How do you pull an image?**
   <details>
   <summary>Answer</summary>
   docker pull image:tag (e.g., docker pull nginx:alpine)
   </details>

3. **What does 'latest' tag mean?**
   <details>
   <summary>Answer</summary>
   It's a convention for the most recent version, but not automatically updated - it's just a tag
   </details>

4. **How do you remove unused images?**
   <details>
   <summary>Answer</summary>
   docker image prune (for dangling) or docker image prune -a (for all unused)
   </details>

5. **Why are images layered?**
   <details>
   <summary>Answer</summary>
   For efficiency - shared layers save disk space and speed up builds/downloads
   </details>

---

## 📝 Hands-on Exercise

**Practice with images:**

```bash
# 1. Pull different versions
docker pull nginx:latest
docker pull nginx:alpine
docker pull nginx:1.25

# 2. List images
docker images nginx

# 3. Inspect an image
docker image inspect nginx:alpine

# 4. View image history
docker image history nginx:alpine

# 5. Search for images
docker search --filter is-official=true postgres

# 6. Tag an image
docker tag nginx:alpine my-nginx:v1.0

# 7. Save image to file
docker save nginx:alpine -o nginx.tar

# 8. Remove tagged image
docker rmi my-nginx:v1.0

# 9. Load image back
docker load -i nginx.tar

# 10. Check disk usage
docker system df

# 11. Clean up
docker image prune -a
```

---

## 📝 Key Takeaways

✅ **Images are read-only templates for containers**  
✅ **Pull images with `docker pull image:tag`**  
✅ **List images with `docker images` or `docker image ls`**  
✅ **Inspect images with `docker image inspect`**  
✅ **Images are built in layers for efficiency**  
✅ **Tag images for versioning and organization**  
✅ **Remove images with `docker rmi` or `docker image prune`**  
✅ **Save/load images for offline transfer**  
✅ **Use specific tags, avoid relying on 'latest'**  

---

## 🚀 Next Steps

**Ready to run containers?**

**Next lesson:** [04 - Docker Containers Fundamentals](04-docker-containers.md) - Run and manage containers

---

## 💡 Pro Tips

**Image Management:**
```bash
# Alias for quick cleanup
alias docker-cleanup='docker image prune -a -f && docker container prune -f'

# View dangling images
docker images -f dangling=true

# Remove images by age
docker image prune -a --filter "until=168h"  # 1 week

# Download before you need it
docker pull postgres:15 &  # Background download
```

**Tag Strategy:**
- Always use specific versions in production
- Use 'latest' only for development
- Implement semantic versioning
- Tag with git commit for traceability

**Storage Tips:**
- Regularly clean dangling images
- Use multi-stage builds to reduce size
- Choose minimal base images (alpine)
- Monitor disk usage with `docker system df`

Images are the foundation - master them! 📦
