---
render_with_liquid: false
---
# 04 - Docker Containers Fundamentals

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand what containers are and how they work
- Run containers in various modes
- Manage container lifecycle
- Interact with running containers
- Map ports and volumes
- Set environment variables
- Understand container networking basics

---

## 🎯 What is a Container?

**Container** = A running instance of an image.

**Real-World Analogy:**
```
Image (Class)     →  Container (Object)
Recipe (Document) →  Cooked Dish (Actual Food)
Program (.exe)    →  Running Process
DVD Movie         →  Playing on TV
```

**Key Concepts:**
- **From one image** → Create many containers
- **Container runs** → Isolated from host and other containers
- **Container stops** → Data inside is lost (unless using volumes)

**Example:**
```bash
# From one nginx image
docker run nginx  # Container 1
docker run nginx  # Container 2 (independent)
docker run nginx  # Container 3 (independent)

# Each container:
- Has its own filesystem
- Has its own processes
- Has its own network
- Cannot see other containers (by default)
```

---

## 🚀 Running Containers

### **Basic Container:**
```bash
# Run container (foreground)
docker run nginx

# What happens:
1. Checks if nginx image exists locally
2. If not, pulls from Docker Hub
3. Creates container from image
4. Starts the container
5. Attaches to container output (blocks terminal)
```

### **Background (Detached) Mode:**
```bash
# Run in background
docker run -d nginx
# Output: container_id (e.g., a1b2c3d4e5f6...)

# Run with custom name
docker run -d --name my-nginx nginx

# Now you can reference it by name:
docker stop my-nginx
docker start my-nginx
```

**When to use:**
- `-d` (detached) → Servers, long-running services
- Foreground → One-time commands, interactive tasks

---

## 📋 Listing Containers

### **View Running Containers:**
```bash
# Show only running containers
docker ps

# Output:
CONTAINER ID   IMAGE    COMMAND                  CREATED        STATUS        PORTS     NAMES
a1b2c3d4e5f6   nginx    "/docker-entrypoint.…"   2 min ago      Up 2 min      80/tcp    my-nginx

# Alternative command
docker container ls
```

### **View All Containers:**
```bash
# Show all (running + stopped)
docker ps -a
docker container ls -a

# Show only container IDs
docker ps -q

# Show last 3 containers created
docker ps -n 3

# Show latest created container
docker ps -l

# Custom format
docker ps --format "table { {.Names} }\t{ {.Status} }\t{ {.Ports} }"
```

---

## 🎮 Container Lifecycle

### **Start/Stop Containers:**
```bash
# Stop running container
docker stop my-nginx
docker stop a1b2c3d4e5f6  # By ID

# Start stopped container
docker start my-nginx

# Restart container
docker restart my-nginx

# Pause container (freeze processes)
docker pause my-nginx

# Unpause container
docker unpause my-nginx
```

### **Lifecycle States:**
```
Created → Running → Paused → Stopped → Removed
   ↓         ↓        ↓         ↓          ↓
  run      pause   unpause    start       rm
         stop/kill            
```

---

## 🗑️ Removing Containers

### **Remove Stopped Container:**
```bash
# Remove by name
docker rm my-nginx

# Remove by ID
docker rm a1b2c3d4e5f6

# Force remove running container
docker rm -f my-nginx

# Remove multiple containers
docker rm container1 container2 container3
```

### **Auto-remove on Exit:**
```bash
# Remove container automatically when it stops
docker run --rm nginx

# Useful for temporary tasks:
docker run --rm alpine echo "Hello World"
# Container runs, prints message, and is automatically deleted
```

### **Remove All Stopped Containers:**
```bash
# Prune stopped containers
docker container prune

# Remove without confirmation
docker container prune -f

# Remove all containers (running + stopped)
docker rm -f $(docker ps -aq)
```

---

## 🌐 Port Mapping

### **Expose Container Ports:**
```bash
# Map container port to host port
docker run -d -p 8080:80 nginx
# Host:Container (localhost:8080 → container:80)

# Access in browser: http://localhost:8080

# Map to specific host IP
docker run -d -p 127.0.0.1:8080:80 nginx

# Random host port
docker run -d -P nginx  # Maps all exposed ports to random host ports

# Multiple port mappings
docker run -d -p 8080:80 -p 8443:443 nginx
```

### **Understanding Port Mapping:**
```
Your Computer (Host)          Container
┌──────────────────┐         ┌──────────┐
│  Browser         │         │          │
│  localhost:8080 ─┼────────→│ :80      │
│                  │         │  nginx   │
└──────────────────┘         └──────────┘

# Command:
docker run -d -p 8080:80 nginx
              └─┬─┘ └┬┘
              Host  Container
```

**Check Port Mappings:**
```bash
# View mapped ports
docker port my-nginx

# Output:
80/tcp -> 0.0.0.0:8080
```

---

## 💾 Volume Mounting

### **Mount Host Directory:**
```bash
# Bind mount (absolute path required)
docker run -d -p 8080:80 -v /path/on/host:/usr/share/nginx/html nginx

# Mount current directory
docker run -d -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx
docker run -d -p 8080:80 -v ${PWD}/html:/usr/share/nginx/html nginx  # Windows

# Read-only mount
docker run -d -v /host/path:/container/path:ro nginx
```

### **Named Volumes:**
```bash
# Create named volume
docker volume create my-data

# Use named volume
docker run -d -v my-data:/var/lib/mysql mysql

# List volumes
docker volume ls

# Inspect volume
docker volume inspect my-data

# Remove volume
docker volume rm my-data
```

### **Volume Types:**
```
1. Bind Mount:
   -v /host/path:/container/path
   → Direct mapping to host filesystem

2. Named Volume:
   -v volume-name:/container/path
   → Managed by Docker, better for persistence

3. Anonymous Volume:
   -v /container/path
   → Created automatically, hard to reference
```

---

## 🔧 Environment Variables

### **Set Variables:**
```bash
# Single variable
docker run -d -e MYSQL_ROOT_PASSWORD=secret mysql

# Multiple variables
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -e MYSQL_USER=admin \
  mysql

# From file
docker run -d --env-file .env mysql
```

**`.env` file example:**
```env
MYSQL_ROOT_PASSWORD=secret
MYSQL_DATABASE=myapp
MYSQL_USER=admin
MYSQL_PASSWORD=admin123
```

---

## 💬 Interactive Containers

### **Interactive Mode:**
```bash
# Run with interactive terminal
docker run -it ubuntu bash
# -i = Keep STDIN open
# -t = Allocate pseudo-TTY (terminal)

# Now you're inside the container:
root@container-id:/# ls
root@container-id:/# pwd
root@container-id:/# cat /etc/os-release
root@container-id:/# exit  # Exit container
```

### **Execute Commands in Running Container:**
```bash
# Start a bash shell in running container
docker exec -it my-nginx bash

# Run single command
docker exec my-nginx ls /usr/share/nginx/html

# Run as specific user
docker exec -u root my-nginx whoami

# Run with environment variable
docker exec -e MY_VAR=value my-nginx env
```

**Difference:**
- `docker run` → Creates new container
- `docker exec` → Runs command in existing container

---

## 📊 Container Logs

### **View Logs:**
```bash
# Show all logs
docker logs my-nginx

# Follow logs (live)
docker logs -f my-nginx

# Show last 50 lines
docker logs --tail 50 my-nginx

# Show logs since specific time
docker logs --since 2024-01-24 my-nginx
docker logs --since 30m my-nginx  # Last 30 minutes

# Show timestamps
docker logs -t my-nginx

# Combination
docker logs -f --tail 20 -t my-nginx
```

---

## 📈 Container Stats

### **Monitor Resources:**
```bash
# Show live resource usage
docker stats

# Show specific container
docker stats my-nginx

# No streaming (snapshot)
docker stats --no-stream

# Output:
CONTAINER   CPU %   MEM USAGE / LIMIT     MEM %   NET I/O        BLOCK I/O
my-nginx    0.5%    10MB / 2GB           0.5%    1MB / 500KB    0B / 0B
```

---

## 🔍 Inspecting Containers

### **Get Container Details:**
```bash
# Full inspection
docker inspect my-nginx

# Get specific information
docker inspect my-nginx --format='{ {.State.Status} }'
docker inspect my-nginx --format='{ {.NetworkSettings.IPAddress} }'
docker inspect my-nginx --format='{ {.Config.Image} }'

# Get multiple fields
docker inspect my-nginx --format='Name: { {.Name} }, IP: { {.NetworkSettings.IPAddress} }'
```

---

## 🛑 Stopping Containers Gracefully

### **Stop vs Kill:**
```bash
# Graceful stop (SIGTERM, wait 10s, then SIGKILL)
docker stop my-nginx

# Force kill immediately (SIGKILL)
docker kill my-nginx

# Stop with custom timeout
docker stop -t 30 my-nginx  # Wait 30 seconds before SIGKILL
```

**What happens:**
```
docker stop:
1. Sends SIGTERM (graceful shutdown signal)
2. Waits 10 seconds (default)
3. If still running, sends SIGKILL (force)

docker kill:
1. Sends SIGKILL immediately (no grace period)
```

---

## 🎯 Common Run Patterns

### **Web Server:**
```bash
docker run -d \
  --name web-server \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html:ro \
  nginx:alpine
```

### **Database:**
```bash
docker run -d \
  --name mysql-db \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=myapp \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0
```

### **One-time Command:**
```bash
docker run --rm \
  -v $(pwd):/workspace \
  -w /workspace \
  node:18-alpine \
  npm install
```

### **Development Environment:**
```bash
docker run -it --rm \
  -v $(pwd):/app \
  -w /app \
  -p 3000:3000 \
  node:18 \
  bash
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between an image and a container?**
   <details>
   <summary>Answer</summary>
   Image is a read-only template; container is a running instance of an image
   </details>

2. **What does `-d` flag do?**
   <details>
   <summary>Answer</summary>
   Runs container in detached mode (background)
   </details>

3. **How do you access container port 80 on host port 8080?**
   <details>
   <summary>Answer</summary>
   docker run -p 8080:80 image
   </details>

4. **What happens to data when container is removed?**
   <details>
   <summary>Answer</summary>
   Data is lost unless stored in volumes
   </details>

5. **How do you run a command in an existing container?**
   <details>
   <summary>Answer</summary>
   docker exec container_name command
   </details>

---

## 📝 Hands-on Exercise

**Complete container workflow:**

```bash
# 1. Run nginx in background
docker run -d --name web -p 8080:80 nginx

# 2. Verify it's running
docker ps
curl http://localhost:8080

# 3. View logs
docker logs web

# 4. Execute command inside
docker exec web ls /usr/share/nginx/html

# 5. Get shell access
docker exec -it web bash
# Inside: cat /etc/nginx/nginx.conf
# Inside: exit

# 6. Check resource usage
docker stats --no-stream web

# 7. Inspect container
docker inspect web --format='{ {.State.Status} }'

# 8. Stop container
docker stop web

# 9. Start it again
docker start web

# 10. Remove container
docker stop web
docker rm web

# 11. Run with volume
mkdir -p html
echo "<h1>Hello from volume!</h1>" > html/index.html
docker run -d --name web -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html:ro nginx

# 12. Test
curl http://localhost:8080

# 13. Cleanup
docker stop web
docker rm web
```

**Challenge - Run MySQL:**
```bash
# Run MySQL with volume and environment
docker run -d \
  --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=mypassword \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# Connect to database
docker exec -it mysql-test mysql -u root -p
# Enter password: mypassword
# Inside MySQL: SHOW DATABASES;
# Inside MySQL: exit;

# Cleanup
docker stop mysql-test
docker rm mysql-test
docker volume rm mysql-data
```

---

## 📝 Key Takeaways

✅ **Container = running instance of image**  
✅ **`docker run` creates and starts container**  
✅ **Use `-d` for background, `-it` for interactive**  
✅ **Map ports with `-p host:container`**  
✅ **Mount volumes with `-v host:container`**  
✅ **Set env variables with `-e KEY=value`**  
✅ **`docker exec` runs commands in running container**  
✅ **`docker logs` shows container output**  
✅ **`--rm` auto-removes container on exit**  
✅ **Data is ephemeral unless using volumes**  

---

## 🚀 Next Steps

**Master advanced container operations:**

**Next lesson:** [05 - Container Operations](05-container-operations.md) - Advanced container management

---

## 💡 Pro Tips

**Container Naming:**
```bash
# ✅ GOOD - Descriptive names
docker run --name web-prod-nginx ...
docker run --name mysql-dev-db ...

# ❌ BAD - Generic names
docker run --name container1 ...
docker run --name test ...
```

**Resource Limits:**
```bash
# Limit memory
docker run -m 512m nginx

# Limit CPU
docker run --cpus=".5" nginx  # 50% of one CPU

# Limit both
docker run -m 512m --cpus=".5" nginx
```

**Cleanup Aliases:**
```bash
# Add to ~/.bashrc or ~/.zshrc
alias dps='docker ps'
alias dpsa='docker ps -a'
alias drm='docker rm'
alias drmi='docker rmi'
alias dstop='docker stop'
alias dstart='docker start'
alias dlogs='docker logs -f'
alias dexec='docker exec -it'
```

**Quick Debugging:**
```bash
# Quick alpine shell
docker run -it --rm alpine sh

# Quick ubuntu with networking tools
docker run -it --rm nicolaka/netshoot

# Quick test web server
docker run -d --rm -p 8080:80 nginx:alpine
```

Containers are Docker's superpower - practice daily! 🚀
