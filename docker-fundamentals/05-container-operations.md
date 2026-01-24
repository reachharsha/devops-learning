---
render_with_liquid: false
---
# 05 - Container Operations & Advanced Management

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Copy files between host and containers
- Commit container changes to new images
- Export and import containers
- View container differences
- Rename and update containers
- Attach to running containers
- Manage container resources
- Troubleshoot common container issues

---

## 📁 Copying Files

### **Copy from Host to Container:**
```bash
# Copy file
docker cp /path/on/host/file.txt my-container:/path/in/container/

# Copy directory
docker cp /path/on/host/folder my-container:/app/

# Copy multiple files
docker cp file1.txt my-container:/app/
docker cp file2.txt my-container:/app/

# Example - Copy config file to nginx
docker cp nginx.conf web-server:/etc/nginx/nginx.conf
docker restart web-server  # Restart to apply changes
```

### **Copy from Container to Host:**
```bash
# Copy file from container
docker cp my-container:/app/logs/error.log ./logs/

# Copy directory from container
docker cp my-container:/var/log/nginx ./nginx-logs/

# Example - Backup database
docker cp mysql-db:/var/lib/mysql/backup.sql ./backups/
```

### **Real-World Use Cases:**
```bash
# 1. Update website content
docker cp index.html web-server:/usr/share/nginx/html/

# 2. Extract logs for analysis
docker cp app-container:/app/logs ./analysis/

# 3. Deploy configuration
docker cp app-config.yml app-container:/etc/app/config.yml

# 4. Backup database
docker cp postgres:/var/lib/postgresql/data ./db-backup/

# 5. Copy SSL certificates
docker cp ssl-cert.pem web-server:/etc/ssl/certs/
```

---

## 💾 Committing Container Changes

### **Create Image from Container:**
```bash
# Basic commit
docker commit my-container my-new-image:v1.0

# With message and author
docker commit -m "Added custom config" -a "Your Name" my-container my-new-image:v1.0

# Pause container during commit (safer)
docker commit -p my-container my-new-image:v1.0
```

### **When to Use Commits:**
```bash
# ✅ GOOD Use Cases:
- Quick prototyping/testing
- Creating debug images
- Temporary customizations
- Learning/experimenting

# ❌ AVOID in Production:
- Not reproducible
- No version control
- No documentation
- Hard to maintain
- Better to use Dockerfile
```

### **Example Workflow:**
```bash
# 1. Run base container
docker run -it --name custom-ubuntu ubuntu bash

# 2. Inside container - make changes
apt update
apt install -y curl vim git
echo "Custom setup complete" > /root/README.txt
exit

# 3. Commit changes
docker commit custom-ubuntu my-ubuntu:dev

# 4. Use new image
docker run -it my-ubuntu:dev bash

# 5. Verify changes
ls /root/
which curl vim git
```

---

## 📦 Export and Import

### **Export Container to Tar:**
```bash
# Export container filesystem
docker export my-container > container-backup.tar
docker export my-container -o container-backup.tar

# View tar contents
tar -tvf container-backup.tar

# Extract tar
mkdir extracted
tar -xf container-backup.tar -C extracted/
```

### **Import Tar as Image:**
```bash
# Import tar file
docker import container-backup.tar my-imported-image:v1.0

# Import with metadata
docker import -m "Imported from backup" container-backup.tar my-image:v1.0

# Import from URL
docker import http://example.com/container.tar my-image:latest
```

### **Export vs Save:**
```bash
# EXPORT (container) - Flattens to single layer
docker export my-container > container.tar
# → Loses history, metadata, layers
# → Creates filesystem snapshot only

# SAVE (image) - Preserves all layers
docker save my-image > image.tar
# → Keeps history, metadata, layers
# → Can be loaded exactly as original
```

**When to Use:**
```
docker export/import:
✅ Minimal image size
✅ Fresh start without history
✅ Flatten multilayer images
❌ Loses Dockerfile instructions
❌ No layer caching benefits

docker save/load:
✅ Perfect image replica
✅ Preserves all metadata
✅ Layer caching intact
❌ Larger file size
```

---

## 🔍 Container Differences

### **View Changes:**
```bash
# Show filesystem changes
docker diff my-container

# Output:
A /app/logs/access.log      # Added
C /etc/nginx/nginx.conf     # Changed
D /tmp/cache/old-file.txt   # Deleted
```

**Change Types:**
- `A` = Added file/directory
- `C` = Changed file/directory
- `D` = Deleted file/directory

### **Track Changes Example:**
```bash
# 1. Run container
docker run -d --name web nginx

# 2. Make changes
docker exec web touch /tmp/newfile.txt
docker exec web bash -c "echo 'test' > /usr/share/nginx/html/test.html"
docker exec web rm /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh

# 3. View differences
docker diff web

# Output:
A /tmp/newfile.txt
C /usr/share/nginx/html
A /usr/share/nginx/html/test.html
D /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
```

---

## 🏷️ Renaming Containers

### **Rename Container:**
```bash
# Rename container
docker rename old-name new-name

# Example
docker run -d --name web nginx
docker rename web prod-web-server

# Verify
docker ps --format "{ {.Names} }"
```

**Why Rename:**
- Better organization
- Clearer naming conventions
- Fix naming mistakes
- Match environment (dev → prod)

---

## 🔄 Updating Containers

### **Update Resource Limits:**
```bash
# Update memory limit
docker update --memory 512m my-container

# Update CPU limit
docker update --cpus 2 my-container

# Update restart policy
docker update --restart=always my-container

# Update multiple containers
docker update --memory 512m container1 container2 container3

# Update all running containers
docker update --restart=unless-stopped $(docker ps -q)
```

**Restart Policies:**
```bash
# No automatic restart
docker update --restart=no my-container

# Always restart
docker update --restart=always my-container

# Restart unless manually stopped
docker update --restart=unless-stopped my-container

# Restart on failure (max 5 times)
docker update --restart=on-failure:5 my-container
```

---

## 🔗 Attaching to Containers

### **Attach to Running Container:**
```bash
# Attach to container output
docker attach my-container

# Detach without stopping: Ctrl+P, Ctrl+Q

# Attach to specific container process
docker attach --sig-proxy=false my-container
```

**Attach vs Exec:**
```
docker attach:
- Connects to container's main process (PID 1)
- Sees the same output as docker logs
- Exiting stops the container

docker exec:
- Starts new process in container
- Independent from main process
- Exiting doesn't stop container
```

### **Example:**
```bash
# Run container with interactive shell
docker run -d -it --name ubuntu-test ubuntu bash

# Attach to it
docker attach ubuntu-test
# Now you're in the bash shell

# Detach: Ctrl+P, Ctrl+Q
# Container keeps running

# Or use exec (better for most cases)
docker exec -it ubuntu-test bash
```

---

## 💪 Container Top/Processes

### **View Container Processes:**
```bash
# Show running processes
docker top my-container

# Output (similar to 'ps'):
PID    USER    TIME    COMMAND
1234   root    0:00    nginx: master process
1235   nginx   0:00    nginx: worker process

# Show with custom format
docker top my-container aux

# Compare with ps inside container
docker exec my-container ps aux
```

---

## 📊 Resource Management

### **Set Resource Limits:**
```bash
# Memory limit
docker run -d --memory="512m" --name web nginx

# CPU limit (50% of one CPU)
docker run -d --cpus="0.5" --name web nginx

# Memory with swap
docker run -d --memory="512m" --memory-swap="1g" --name web nginx

# CPU shares (relative weight)
docker run -d --cpu-shares=512 --name web1 nginx
docker run -d --cpu-shares=1024 --name web2 nginx

# Limit disk I/O
docker run -d --device-read-bps /dev/sda:1mb --name web nginx
docker run -d --device-write-bps /dev/sda:1mb --name web nginx
```

### **Monitor Resources:**
```bash
# Real-time stats
docker stats

# Specific container
docker stats my-container

# All containers (no streaming)
docker stats --no-stream --all

# Custom format
docker stats --format "table { {.Name} }\t{ {.CPUPerc} }\t{ {.MemUsage} }"
```

---

## 🔧 Container Events

### **Monitor Events:**
```bash
# Watch all Docker events
docker events

# Filter by container
docker events --filter container=my-container

# Filter by event type
docker events --filter event=start
docker events --filter event=stop
docker events --filter event=die

# Filter by time
docker events --since '2024-01-24'
docker events --since '30m'  # Last 30 minutes

# Multiple filters
docker events --filter container=my-container --filter event=start
```

**Event Types:**
- attach, commit, copy, create, destroy, detach, die
- exec_create, exec_detach, exec_start, export
- kill, oom, pause, rename, resize, restart
- start, stop, top, unpause, update

---

## 🩺 Container Health Checks

### **Check Container Health:**
```bash
# View health status
docker inspect my-container --format='{ {.State.Health.Status} }'

# Possible values:
- starting  # Health check in progress
- healthy   # All checks passed
- unhealthy # Checks failed

# View health check history
docker inspect my-container --format='{ {json .State.Health} }' | jq
```

### **Run with Health Check:**
```bash
# Simple health check
docker run -d \
  --name web \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-retries=3 \
  nginx

# Check status
docker ps  # Shows (healthy) or (unhealthy)
```

---

## 🐛 Troubleshooting Containers

### **Debug Failing Container:**
```bash
# 1. Check if it's running
docker ps -a

# 2. View logs
docker logs my-container
docker logs --tail 50 my-container

# 3. Inspect container
docker inspect my-container

# 4. Check exit code
docker inspect my-container --format='{ {.State.ExitCode} }'

# Common exit codes:
# 0   - Success
# 1   - Application error
# 125 - Docker daemon error
# 126 - Command cannot execute
# 127 - Command not found
# 137 - SIGKILL (killed)
# 139 - Segmentation fault
# 143 - SIGTERM (graceful stop)

# 5. Check resource usage
docker stats --no-stream my-container

# 6. View events
docker events --filter container=my-container --since 60m
```

### **Container Won't Start:**
```bash
# Try running interactively
docker run -it --rm image-name bash

# Override entrypoint
docker run -it --rm --entrypoint bash image-name

# Check port conflicts
netstat -tuln | grep 8080
lsof -i :8080

# Check volume permissions
ls -la /host/volume/path

# Run with more verbose output
docker run --log-level debug image-name
```

### **Container is Slow:**
```bash
# Check resource limits
docker inspect my-container | grep -i memory
docker inspect my-container | grep -i cpu

# Monitor in real-time
docker stats my-container

# Check disk I/O
docker stats --format "{ {.BlockIO} }"

# View processes
docker top my-container

# Check logs for errors
docker logs my-container | grep -i error
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you copy a file from container to host?**
   <details>
   <summary>Answer</summary>
   docker cp container:/path/in/container /path/on/host
   </details>

2. **What's the difference between commit and save?**
   <details>
   <summary>Answer</summary>
   commit creates image from container; save exports existing image
   </details>

3. **How do you view filesystem changes in a container?**
   <details>
   <summary>Answer</summary>
   docker diff container-name
   </details>

4. **How do you update container memory limit?**
   <details>
   <summary>Answer</summary>
   docker update --memory 512m container-name
   </details>

5. **What does exit code 137 mean?**
   <details>
   <summary>Answer</summary>
   Container was killed (SIGKILL), often due to OOM (out of memory)
   </details>

---

## 📝 Hands-on Exercise

**Complete operations workflow:**

```bash
# 1. Run container with resource limits
docker run -d \
  --name webapp \
  --memory="256m" \
  --cpus="0.5" \
  -p 8080:80 \
  nginx

# 2. Copy custom file
echo "<h1>Custom Page</h1>" > index.html
docker cp index.html webapp:/usr/share/nginx/html/

# 3. View changes
docker diff webapp

# 4. Check stats
docker stats --no-stream webapp

# 5. View processes
docker top webapp

# 6. Make more changes
docker exec webapp touch /tmp/test.txt
docker exec webapp bash -c "echo 'log entry' > /tmp/app.log"

# 7. Commit to new image
docker commit webapp custom-nginx:v1.0

# 8. Export container
docker export webapp > webapp-backup.tar

# 9. Update resource limits
docker update --memory="512m" webapp

# 10. Rename container
docker rename webapp production-web

# 11. Verify
docker inspect production-web --format='{ {.HostConfig.Memory} }'

# 12. Cleanup
docker stop production-web
docker rm production-web
docker rmi custom-nginx:v1.0
rm webapp-backup.tar index.html
```

**Challenge - Debug failing container:**
```bash
# Run container that exits immediately
docker run --name failing-app alpine sh -c "exit 1"

# Debug it
docker ps -a  # See it exited
docker logs failing-app
docker inspect failing-app --format='{ {.State.ExitCode} }'

# Fix and run again
docker run --name working-app alpine sh -c "echo 'Success!' && sleep infinity"
docker logs working-app

# Cleanup
docker stop working-app
docker rm failing-app working-app
```

---

## 📝 Key Takeaways

✅ **`docker cp` copies files between host and container**  
✅ **`docker commit` creates image from container changes**  
✅ **`docker export` creates tar from container filesystem**  
✅ **`docker import` creates image from tar**  
✅ **`docker diff` shows filesystem changes**  
✅ **`docker rename` changes container name**  
✅ **`docker update` modifies resource limits**  
✅ **`docker top` shows container processes**  
✅ **`docker stats` monitors resource usage**  
✅ **Exit codes help diagnose failures**  

---

## 🚀 Next Steps

**Ready to build your own images?**

**Next lesson:** [06 - Writing Dockerfiles](06-writing-dockerfiles.md) - Create custom Docker images

---

## 💡 Pro Tips

**Efficient File Copying:**
```bash
# Copy multiple files at once
docker cp . my-container:/app/  # Copy all from current dir

# Use tar for large transfers
tar -czf - files/ | docker exec -i my-container tar -xzf - -C /app/
```

**Container Debugging Tools:**
```bash
# Install debugging tools in running container
docker exec -it my-container bash -c "apt update && apt install -y vim curl wget"

# Use nsenter to enter container namespace (advanced)
PID=$(docker inspect --format { {.State.Pid} } my-container)
nsenter -t $PID -n ip addr  # View container network
```

**Resource Management:**
```bash
# Prevent OOM killer
docker run --memory="512m" --memory-reservation="256m" --oom-kill-disable nginx

# CPU pinning
docker run --cpuset-cpus="0,1" nginx  # Use only CPU 0 and 1
```

**Backup Strategy:**
```bash
# Backup script
#!/bin/bash
CONTAINER=$1
BACKUP_DIR="./backups/$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

docker export $CONTAINER > $BACKUP_DIR/$CONTAINER.tar
docker cp $CONTAINER:/app/data $BACKUP_DIR/data
echo "Backup complete: $BACKUP_DIR"
```

Master these operations - they're essential for daily Docker work! 🛠️
