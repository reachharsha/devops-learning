---
render_with_liquid: false
---
# 08 - Docker Volumes & Data Persistence

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand Docker's storage architecture
- Create and manage volumes
- Use bind mounts effectively
- Understand tmpfs mounts
- Share data between containers
- Backup and restore volumes
- Apply best practices for data persistence

---

## 💾 The Data Persistence Problem

### **Container Filesystem is Ephemeral:**
```bash
# Run container and create file
docker run -it --name temp ubuntu bash
# Inside: echo "important data" > /data.txt
# Inside: exit

# Start container again - data still there
docker start -ai temp
# Inside: cat /data.txt  → "important data"
# Inside: exit

# Remove container - DATA IS GONE!
docker rm temp

# Run new container from same image
docker run -it --name temp2 ubuntu bash
# Inside: cat /data.txt  → file not found
```

**The Problem:**
- Container filesystem is **temporary**
- Data lost when container is removed
- Can't share data between containers
- Database data would be lost!

**The Solution:**
- **Volumes** - Managed by Docker, best for most cases
- **Bind Mounts** - Direct host filesystem access
- **tmpfs Mounts** - In-memory, temporary data

---

## 📦 Docker Storage Types

### **Comparison:**
```
┌─────────────────────────────────────────────────┐
│                    Host                         │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────┐ │
│  │   Volume     │  │  Bind Mount  │  │ tmpfs│ │
│  │ /var/lib/    │  │  /host/path  │  │ RAM  │ │
│  │ docker/      │  │              │  │      │ │
│  │ volumes/     │  │              │  │      │ │
│  └──────┬───────┘  └──────┬───────┘  └───┬──┘ │
│         │                 │               │    │
│  ┌──────┴─────────────────┴───────────────┴──┐ │
│  │          Container Filesystem             │ │
│  │  /app/data  /config  /cache               │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

---

## 📁 Docker Volumes

### **What are Volumes?**
- **Managed by Docker** - Stored in `/var/lib/docker/volumes/`
- **Persistent** - Survive container deletion
- **Portable** - Easy to backup and migrate
- **Shared** - Multiple containers can use same volume
- **Better Performance** - Especially on macOS/Windows

### **Create Volume:**
```bash
# Create named volume
docker volume create my-data

# Create volume with driver options
docker volume create --driver local \
  --opt type=none \
  --opt o=bind \
  --opt device=/path/on/host \
  my-volume

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data
```

### **Use Volume:**
```bash
# Mount volume to container
docker run -d \
  --name web \
  -v my-data:/var/www/html \
  nginx

# Or using --mount (more explicit)
docker run -d \
  --name web \
  --mount source=my-data,target=/var/www/html \
  nginx
```

### **Volume Lifecycle:**
```bash
# 1. Create volume
docker volume create app-data

# 2. Use in container
docker run -d --name app -v app-data:/data ubuntu sleep infinity

# 3. Write data
docker exec app bash -c "echo 'persistent data' > /data/file.txt"

# 4. Remove container
docker stop app
docker rm app

# 5. Data still exists!
docker run --rm -v app-data:/data ubuntu cat /data/file.txt
# Output: persistent data

# 6. Clean up
docker volume rm app-data
```

---

## 🔗 Bind Mounts

### **What are Bind Mounts?**
- **Direct host filesystem access**
- **Any location** - Not managed by Docker
- **Development** - Great for live code editing
- **Configuration** - Share config files
- **Less portable** - Depends on host filesystem

### **Use Bind Mount:**
```bash
# Absolute path required
docker run -d \
  --name web \
  -v /home/user/website:/usr/share/nginx/html:ro \
  nginx

# Current directory
docker run -d \
  --name web \
  -v $(pwd)/html:/usr/share/nginx/html \
  nginx

# Using --mount (preferred)
docker run -d \
  --name web \
  --mount type=bind,source=/home/user/website,target=/usr/share/nginx/html,readonly \
  nginx
```

### **Development Workflow:**
```bash
# Project structure
my-app/
├── src/
│   └── index.js
└── package.json

# Run with bind mount
docker run -d \
  --name dev-server \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18 \
  npm start

# Edit src/index.js on host → Changes reflect immediately in container!
```

---

## 💨 tmpfs Mounts

### **What is tmpfs?**
- **In-memory storage** - Very fast
- **Temporary** - Lost on container stop
- **Secure** - Sensitive data not written to disk
- **Limited by RAM** - Don't use for large data

### **Use tmpfs:**
```bash
# Create tmpfs mount
docker run -d \
  --name app \
  --tmpfs /app/cache:rw,size=100m,mode=1777 \
  nginx

# Using --mount
docker run -d \
  --name app \
  --mount type=tmpfs,target=/app/cache,tmpfs-size=100000000 \
  nginx
```

### **Use Cases:**
- **Caching** - Temporary cache data
- **Secrets** - Sensitive data that shouldn't persist
- **Build artifacts** - Temporary compilation files
- **Session data** - User sessions

---

## 🔄 Volume vs Bind Mount

### **When to Use Volumes:**
```bash
# ✅ Production databases
docker run -d \
  --name postgres \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# ✅ Application data
docker run -d \
  --name app \
  -v app-data:/app/data \
  -v app-logs:/app/logs \
  myapp:latest

# ✅ Shared data between containers
docker run -d --name writer -v shared:/data ubuntu
docker run -d --name reader -v shared:/data ubuntu
```

### **When to Use Bind Mounts:**
```bash
# ✅ Development - live code reload
docker run -d \
  -v $(pwd)/src:/app/src \
  -v $(pwd)/public:/app/public \
  node:18

# ✅ Configuration files
docker run -d \
  -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro \
  nginx

# ✅ Log aggregation
docker run -d \
  -v /var/log/app:/logs \
  logstash
```

---

## 📊 Managing Volumes

### **List Volumes:**
```bash
# List all volumes
docker volume ls

# Filter by name
docker volume ls --filter name=app

# Filter dangling volumes (not used by any container)
docker volume ls --filter dangling=true

# Format output
docker volume ls --format "{ {.Name} }: { {.Driver} }"
```

### **Inspect Volume:**
```bash
# Full details
docker volume inspect my-data

# Output:
[
    {
        "CreatedAt": "2026-01-24T10:30:00Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-data/_data",
        "Name": "my-data",
        "Options": {},
        "Scope": "local"
    }
]

# Get specific info
docker volume inspect my-data --format '{ {.Mountpoint} }'
```

### **Remove Volumes:**
```bash
# Remove specific volume
docker volume rm my-data

# Remove multiple volumes
docker volume rm vol1 vol2 vol3

# Remove all unused volumes
docker volume prune

# Remove without confirmation
docker volume prune -f

# Remove volumes older than 24h
docker volume prune --filter "label!=keep"
```

---

## 🔐 Volume Permissions

### **Understanding Permissions:**
```bash
# Volume inherits container user's UID/GID
docker run -d \
  --name app \
  --user 1000:1000 \
  -v app-data:/data \
  ubuntu

# Check permissions
docker exec app ls -la /data
# drwxr-xr-x 2 1000 1000 ...

# Fix permissions
docker run --rm \
  -v app-data:/data \
  ubuntu \
  chown -R 1000:1000 /data
```

### **Best Practice:**
```dockerfile
# In Dockerfile
FROM node:18-alpine

# Create user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Create volume directory with correct permissions
RUN mkdir -p /app/data && \
    chown -R nodejs:nodejs /app

USER nodejs

VOLUME /app/data
```

---

## 🚀 Sharing Data Between Containers

### **Volumes from Another Container:**
```bash
# Create data container
docker run -d \
  --name data-provider \
  -v shared-data:/data \
  ubuntu sleep infinity

# Use volume from another container
docker run -d \
  --name app1 \
  --volumes-from data-provider \
  nginx

docker run -d \
  --name app2 \
  --volumes-from data-provider \
  nginx

# All three containers share /data
```

### **Shared Volume:**
```bash
# Create shared volume
docker volume create shared-storage

# Multiple containers use same volume
docker run -d --name writer -v shared-storage:/data alpine
docker run -d --name reader1 -v shared-storage:/data:ro alpine
docker run -d --name reader2 -v shared-storage:/data:ro alpine

# Writer writes, readers read
docker exec writer sh -c "echo 'data' > /data/file.txt"
docker exec reader1 cat /data/file.txt
docker exec reader2 cat /data/file.txt
```

---

## 💾 Backup and Restore

### **Backup Volume:**
```bash
# Method 1: Tar archive
docker run --rm \
  -v my-data:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/my-data-backup.tar.gz -C /data .

# Method 2: Using volume directly
docker run --rm \
  -v my-data:/data:ro \
  -v $(pwd):/backup \
  alpine \
  tar czf /backup/backup-$(date +%Y%m%d).tar.gz -C /data .
```

### **Restore Volume:**
```bash
# Create new volume
docker volume create my-data-restored

# Restore from backup
docker run --rm \
  -v my-data-restored:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/my-data-backup.tar.gz -C /data
```

### **Copy Volume:**
```bash
# Copy volume to another volume
docker volume create source-vol
docker volume create target-vol

docker run --rm \
  -v source-vol:/from:ro \
  -v target-vol:/to \
  alpine \
  sh -c "cp -av /from/. /to/"
```

---

## 🗄️ Database Volumes

### **PostgreSQL:**
```bash
# Create volume
docker volume create postgres-data

# Run PostgreSQL
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=secret \
  -v postgres-data:/var/lib/postgresql/data \
  postgres:15

# Backup database
docker exec postgres pg_dump -U postgres mydb > backup.sql

# Backup volume
docker run --rm \
  -v postgres-data:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/postgres-backup.tar.gz -C /data .
```

### **MySQL:**
```bash
# Create volume
docker volume create mysql-data

# Run MySQL
docker run -d \
  --name mysql \
  -e MYSQL_ROOT_PASSWORD=secret \
  -v mysql-data:/var/lib/mysql \
  mysql:8.0

# Backup
docker exec mysql mysqldump -u root -psecret --all-databases > backup.sql
```

### **MongoDB:**
```bash
# Create volume
docker volume create mongo-data

# Run MongoDB
docker run -d \
  --name mongo \
  -v mongo-data:/data/db \
  mongo:6.0

# Backup
docker exec mongo mongodump --archive=/backup.archive
docker cp mongo:/backup.archive ./mongo-backup.archive
```

---

## 🎯 Quick Check: Do You Understand?

1. **What happens to data in a container when it's removed?**
   <details>
   <summary>Answer</summary>
   Data is lost unless stored in a volume or bind mount
   </details>

2. **What's the difference between volume and bind mount?**
   <details>
   <summary>Answer</summary>
   Volume: managed by Docker, portable. Bind mount: direct host access, development
   </details>

3. **Where are Docker volumes stored on Linux?**
   <details>
   <summary>Answer</summary>
   /var/lib/docker/volumes/
   </details>

4. **How do you make a bind mount read-only?**
   <details>
   <summary>Answer</summary>
   Add :ro flag: -v /host/path:/container/path:ro
   </details>

5. **What is tmpfs used for?**
   <details>
   <summary>Answer</summary>
   In-memory temporary storage for cache, secrets, temporary files
   </details>

---

## 📝 Hands-on Exercise

**Complete volume workflow:**

```bash
# 1. Create volume
docker volume create my-app-data

# 2. Run container with volume
docker run -d \
  --name app \
  -v my-app-data:/data \
  ubuntu sleep infinity

# 3. Write data
docker exec app bash -c "echo 'Hello Volume' > /data/message.txt"
docker exec app bash -c "date > /data/timestamp.txt"

# 4. Verify data
docker exec app cat /data/message.txt

# 5. Inspect volume
docker volume inspect my-app-data

# 6. Backup volume
docker run --rm \
  -v my-app-data:/data:ro \
  -v $(pwd):/backup \
  ubuntu \
  tar czf /backup/app-backup.tar.gz -C /data .

# 7. Remove container
docker stop app
docker rm app

# 8. Data still exists - create new container
docker run --rm \
  -v my-app-data:/data \
  ubuntu \
  cat /data/message.txt

# 9. Restore to new volume
docker volume create restored-data
docker run --rm \
  -v restored-data:/data \
  -v $(pwd):/backup \
  ubuntu \
  tar xzf /backup/app-backup.tar.gz -C /data

# 10. Verify restore
docker run --rm \
  -v restored-data:/data \
  ubuntu \
  ls -la /data

# 11. Cleanup
docker volume rm my-app-data restored-data
rm app-backup.tar.gz
```

**Challenge - Database persistence:**
```bash
# 1. Run PostgreSQL with volume
docker volume create pgdata
docker run -d \
  --name postgres \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# 2. Create database and table
docker exec -it postgres psql -U postgres -c "CREATE DATABASE testdb;"
docker exec -it postgres psql -U postgres -d testdb -c \
  "CREATE TABLE users (id SERIAL PRIMARY KEY, name VARCHAR(50));"
docker exec -it postgres psql -U postgres -d testdb -c \
  "INSERT INTO users (name) VALUES ('Alice'), ('Bob');"

# 3. Verify data
docker exec -it postgres psql -U postgres -d testdb -c "SELECT * FROM users;"

# 4. Remove container
docker stop postgres
docker rm postgres

# 5. Create new container with same volume
docker run -d \
  --name postgres-new \
  -e POSTGRES_PASSWORD=mysecret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:15

# 6. Data persisted!
sleep 5  # Wait for startup
docker exec -it postgres-new psql -U postgres -d testdb -c "SELECT * FROM users;"

# 7. Cleanup
docker stop postgres-new
docker rm postgres-new
docker volume rm pgdata
```

---

## 📝 Key Takeaways

✅ **Container filesystems are ephemeral by default**  
✅ **Volumes are managed by Docker - best for production**  
✅ **Bind mounts map host directories - best for development**  
✅ **tmpfs uses RAM - best for temporary data**  
✅ **Volumes survive container deletion**  
✅ **Multiple containers can share volumes**  
✅ **Use :ro for read-only mounts**  
✅ **Backup volumes regularly**  
✅ **Use `docker volume prune` to clean up**  
✅ **Volumes have better performance on Docker Desktop**  

---

## 🚀 Next Steps

**Ready to connect containers?**

**Next lesson:** [09 - Docker Networking](09-docker-networking.md) - Container communication

---

## 💡 Pro Tips

**Volume Naming Convention:**
```bash
# ✅ GOOD - Descriptive names
docker volume create myapp-postgres-data
docker volume create myapp-logs
docker volume create myapp-uploads

# ❌ BAD - Generic names
docker volume create data
docker volume create vol1
```

**Development vs Production:**
```bash
# Development - bind mount for live reload
docker run -v $(pwd)/src:/app/src node:18

# Production - volumes for persistence
docker run -v app-data:/app/data myapp:prod
```

**Backup Script:**
```bash
#!/bin/bash
# backup-volumes.sh
VOLUME_NAME=$1
BACKUP_DIR="./backups"
mkdir -p $BACKUP_DIR

docker run --rm \
  -v ${VOLUME_NAME}:/data:ro \
  -v ${BACKUP_DIR}:/backup \
  alpine \
  tar czf /backup/${VOLUME_NAME}-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

echo "Backup complete: ${BACKUP_DIR}/${VOLUME_NAME}-*.tar.gz"
```

**Cleanup Automation:**
```bash
# Clean up unused volumes weekly
0 0 * * 0 docker volume prune -f

# Or add to script
docker volume prune --filter "label!=keep" -f
```

Volumes are crucial for production - master them! 💾
