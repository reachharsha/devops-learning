---
render_with_liquid: false
---
# 21 - Advanced Troubleshooting

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Debug container startup failures
- Troubleshoot networking issues
- Investigate performance problems
- Analyze container logs effectively
- Use Docker debugging tools
- Resolve common Docker issues
- Implement troubleshooting workflows

---

## 🔍 Container Won't Start

### **Check Container Status:**
```bash
# List all containers (including stopped)
docker ps -a

# Common exit codes:
# 0   - Success
# 1   - Application error
# 125 - Docker daemon error
# 126 - Command cannot be invoked
# 127 - Command not found
# 137 - SIGKILL (OOM or manual kill)
# 139 - SIGSEGV (segmentation fault)
# 143 - SIGTERM (graceful termination)

# Check exit code
docker inspect container --format='{ {.State.ExitCode} }'

# View complete state
docker inspect container | jq '.[0].State'
```

### **Check Logs:**
```bash
# View container logs
docker logs container

# Last 100 lines
docker logs --tail 100 container

# With timestamps
docker logs -t container

# Follow logs
docker logs -f container

# Logs since specific time
docker logs --since 2024-01-24T10:00:00 container
docker logs --since 30m container
```

### **Interactive Debugging:**
```bash
# Run container with interactive shell
docker run -it myapp:latest /bin/sh

# Override entrypoint to debug
docker run -it --entrypoint /bin/sh myapp:latest

# Run with debug mode
docker run -it -e DEBUG=true myapp:latest

# Check what command is being executed
docker inspect myapp:latest | jq '.[0].Config.Cmd'
docker inspect myapp:latest | jq '.[0].Config.Entrypoint'
```

### **Common Startup Issues:**

#### **Issue 1: Command Not Found**
```dockerfile
# ❌ Problem: Command doesn't exist
FROM alpine:latest
CMD ["nodejs", "app.js"]
# Error: executable file not found
```

```dockerfile
# ✅ Solution: Install Node.js
FROM node:18-alpine
COPY app.js .
CMD ["node", "app.js"]
```

#### **Issue 2: Permission Denied**
```bash
# Problem: Script not executable
# Error: permission denied

# Solution 1: Make script executable in Dockerfile
RUN chmod +x /app/start.sh

# Solution 2: Use sh to run script
CMD ["sh", "/app/start.sh"]
```

#### **Issue 3: Missing Files**
```bash
# Check what files exist in container
docker run --rm myapp:latest ls -la /app

# Check build context
docker build --progress=plain --no-cache -t myapp:latest .

# Verify .dockerignore isn't excluding needed files
cat .dockerignore
```

---

## 🌐 Network Troubleshooting

### **Container Can't Connect to Network:**
```bash
# Check container network
docker inspect container | jq '.[0].NetworkSettings.Networks'

# List networks
docker network ls

# Inspect network
docker network inspect bridge

# Check if container is on network
docker network inspect my-network | jq '.[0].Containers'

# Connect container to network
docker network connect my-network container

# Disconnect from network
docker network disconnect my-network container
```

### **DNS Issues:**
```bash
# Test DNS resolution inside container
docker exec container nslookup google.com

# Check DNS servers
docker exec container cat /etc/resolv.conf

# Override DNS
docker run -d \
  --dns 8.8.8.8 \
  --dns 8.8.4.4 \
  nginx

# Test connectivity
docker exec container ping -c 3 google.com
docker exec container wget -O- http://google.com
```

### **Port Mapping Issues:**
```bash
# Check port mappings
docker port container

# Verify port is listening inside container
docker exec container netstat -tlnp
# or
docker exec container ss -tlnp

# Test from host
curl http://localhost:8080

# Check if port is in use on host
sudo lsof -i :8080
sudo netstat -tlnp | grep 8080

# Bind to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx  # localhost only
docker run -d -p 0.0.0.0:8080:80 nginx    # all interfaces
```

### **Service Discovery Issues:**
```bash
# Check if services can resolve each other
docker exec web ping -c 3 api

# View container hostname
docker exec container hostname

# Check /etc/hosts
docker exec container cat /etc/hosts

# Inspect DNS in custom network
docker network inspect my-network | jq '.[0].IPAM'
```

### **Debugging with Network Tools:**
```bash
# Install debugging tools in container
docker exec -it container sh
apk add curl wget netcat-openbsd bind-tools

# Test connectivity
curl http://api:3000
nc -zv api 3000

# Trace route
traceroute api

# DNS lookup
nslookup api
dig api
```

---

## 💾 Volume and Storage Issues

### **Data Not Persisting:**
```bash
# Check volumes
docker volume ls

# Inspect volume
docker volume inspect my-volume

# Check where volume is mounted
docker inspect container | jq '.[0].Mounts'

# Verify data in volume
docker run --rm \
  -v my-volume:/data \
  alpine ls -la /data

# Check volume permissions
docker run --rm \
  -v my-volume:/data \
  alpine ls -ld /data
```

### **Permission Issues:**
```bash
# Check file ownership
docker exec container ls -la /app/data

# Fix permissions (run as root)
docker exec -u root container chown -R 1001:1001 /app/data

# Or rebuild with correct permissions
COPY --chown=1001:1001 . /app
```

### **Disk Space Issues:**
```bash
# Check disk usage
docker system df

# Detailed view
docker system df -v

# Check container filesystem usage
docker exec container df -h

# Clean up
docker system prune -a
docker volume prune
docker image prune -a
```

---

## 🐛 Application-Level Debugging

### **Memory Issues:**
```bash
# Check memory usage
docker stats container --no-stream

# Check if OOM killed
docker inspect container | jq '.[0].State.OOMKilled'

# View dmesg for OOM events
dmesg | grep -i oom

# Increase memory limit
docker run -d --memory="2g" myapp:latest

# Check memory inside container
docker exec container free -m
docker exec container cat /proc/meminfo
```

### **CPU Issues:**
```bash
# Check CPU usage
docker stats container

# See processes inside container
docker exec container ps aux

# Top processes
docker exec container top

# Limit CPU
docker run -d --cpus="0.5" myapp:latest

# Check CPU usage over time
while true; do
  docker stats container --no-stream | ts
  sleep 5
done
```

### **Process Debugging:**
```bash
# Check running processes
docker exec container ps aux

# See process tree
docker exec container ps auxf

# Check PID 1 (main process)
docker exec container ps -p 1

# Check if process is zombie
docker exec container ps aux | grep Z

# View process details
docker exec container cat /proc/1/status
docker exec container cat /proc/1/cmdline
```

---

## 🔬 Advanced Debugging Tools

### **strace - System Call Tracing:**
```bash
# Run container with strace
docker run --rm -it \
  --cap-add=SYS_PTRACE \
  --security-opt seccomp=unconfined \
  alpine sh

# Inside container, install strace
apk add strace

# Trace process
strace -p 1

# Trace specific system calls
strace -e trace=open,read,write -p 1

# Count system calls
strace -c -p 1
```

### **tcpdump - Network Packet Analysis:**
```bash
# Run container with network debugging
docker run --rm -it \
  --cap-add=NET_ADMIN \
  --cap-add=NET_RAW \
  nicolaka/netshoot

# Capture packets
tcpdump -i eth0

# Capture HTTP traffic
tcpdump -i eth0 -A 'tcp port 80'

# Save to file
tcpdump -i eth0 -w capture.pcap

# Read capture file
tcpdump -r capture.pcap
```

### **nsenter - Enter Container Namespace:**
```bash
# Get container PID
PID=$(docker inspect --format { {.State.Pid} } container)

# Enter network namespace
sudo nsenter -t $PID -n

# Now you can use host tools
ip addr
netstat -tlnp
tcpdump -i eth0
```

### **Debugging Sidecar:**
```bash
# Run debugging container in same namespace
docker run --rm -it \
  --network container:myapp \
  --pid container:myapp \
  nicolaka/netshoot

# Now debug the target container
ps aux
netstat -tlnp
curl localhost:3000
```

---

## 📊 Log Analysis

### **Structured Log Parsing:**
```bash
# JSON logs
docker logs container | jq '.level, .message'

# Filter error logs
docker logs container | jq 'select(.level=="error")'

# Count log levels
docker logs container | jq -r '.level' | sort | uniq -c

# Find specific errors
docker logs container | grep "ERROR"
docker logs container | grep -i "exception"
```

### **Log Aggregation:**
```bash
# Export logs to file
docker logs container > app.log 2>&1

# Analyze with standard tools
cat app.log | grep ERROR | wc -l
cat app.log | awk '{print $1}' | sort | uniq -c

# Follow multiple containers
docker-compose logs -f web api db
```

---

## 🔧 Build Troubleshooting

### **Build Failures:**
```bash
# Build with detailed output
DOCKER_BUILDKIT=0 docker build --progress=plain --no-cache -t myapp:latest .

# Build up to specific stage
docker build --target builder -t myapp:debug .

# Interactive debugging of build
docker run -it --rm $(docker build -q --target builder .) sh

# Check build context
docker build --no-cache -t myapp:latest . 2>&1 | grep "Sending build context"
```

### **Layer Analysis:**
```bash
# View layer history
docker history myapp:latest

# Detailed layer info
docker history --no-trunc myapp:latest

# Find large layers
docker history myapp:latest | sort -k7 -h

# Analyze with dive
docker run --rm -it \
  -v /var/run/docker.sock:/var/run/docker.sock \
  wagoodman/dive:latest myapp:latest
```

---

## 🎯 Common Issues & Solutions

### **Issue: Container Exits Immediately**
```bash
# Problem: Application not running in foreground
# Solution: Ensure main process doesn't exit

# ❌ Wrong
CMD npm start &  # Runs in background, container exits

# ✅ Correct
CMD ["npm", "start"]  # Runs in foreground

# Debug
docker run --rm -it --entrypoint sh myapp:latest
```

### **Issue: "Cannot connect to Docker daemon"**
```bash
# Check if Docker is running
sudo systemctl status docker

# Start Docker
sudo systemctl start docker

# Check permissions
sudo usermod -aG docker $USER
newgrp docker

# Verify socket
ls -l /var/run/docker.sock
```

### **Issue: "No space left on device"**
```bash
# Check disk usage
df -h
docker system df

# Clean up
docker system prune -a --volumes

# Clean specific items
docker container prune
docker image prune -a
docker volume prune
docker network prune
```

### **Issue: "Conflict. Container already exists"**
```bash
# Remove old container
docker rm container-name

# Force remove
docker rm -f container-name

# Remove and recreate
docker-compose up -d --force-recreate
```

### **Issue: "Port already in use"**
```bash
# Find what's using the port
sudo lsof -i :8080
sudo netstat -tlnp | grep 8080

# Kill process
sudo kill -9 PID

# Use different port
docker run -d -p 8081:80 nginx
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does exit code 137 mean?**
   <details>
   <summary>Answer</summary>
   Container was killed (SIGKILL), often due to OOM (Out Of Memory)
   </details>

2. **How do you check which network a container is on?**
   <details>
   <summary>Answer</summary>
   docker inspect container | jq '.[0].NetworkSettings.Networks'
   </details>

3. **What command shows container resource usage?**
   <details>
   <summary>Answer</summary>
   docker stats container
   </details>

4. **How do you override the entrypoint for debugging?**
   <details>
   <summary>Answer</summary>
   docker run -it --entrypoint /bin/sh myapp:latest
   </details>

5. **How do you check if a container was OOM killed?**
   <details>
   <summary>Answer</summary>
   docker inspect container | jq '.[0].State.OOMKilled'
   </details>

---

## 📝 Hands-on Exercise

**Debug a failing container:**

```bash
# 1. Create broken app
mkdir debug-lab
cd debug-lab

cat > app.js << 'EOF'
// Intentionally broken app
const express = require('express');
const app = express();

// Memory leak
let data = [];
setInterval(() => {
  data.push(new Array(1000000));
}, 100);

app.get('/', (req, res) => {
  res.send('Hello!');
});

app.listen(3000);
EOF

cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY app.js .
RUN npm install express
CMD ["node", "app.js"]
EOF

# 2. Build and run
docker build -t broken-app .
docker run -d --name broken --memory="100m" broken-app

# 3. Debug
# Watch it get OOM killed
docker stats broken
sleep 30

# 4. Check why it failed
docker ps -a | grep broken
docker inspect broken | jq '.[0].State'
docker logs broken

# 5. Fix and test
cat > app-fixed.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Hello!');
});

app.listen(3000, () => {
  console.log('Server started');
});
EOF

cat > Dockerfile.fixed << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY app-fixed.js ./app.js
RUN npm install express
CMD ["node", "app.js"]
EOF

docker build -f Dockerfile.fixed -t fixed-app .
docker run -d --name fixed --memory="100m" fixed-app

# 6. Verify it works
sleep 5
docker stats fixed --no-stream
docker logs fixed

# 7. Cleanup
cd ..
rm -rf debug-lab
docker rm -f broken fixed
docker rmi broken-app fixed-app
```

---

## 📝 Key Takeaways

✅ **Always check logs first**  
✅ **Use docker inspect for detailed info**  
✅ **Exit codes tell you what went wrong**  
✅ **Test networking with docker exec**  
✅ **Monitor resources with docker stats**  
✅ **Override entrypoint for debugging**  
✅ **Use debugging sidecars for complex issues**  
✅ **Check volume permissions and mounts**  
✅ **Clean up regularly to avoid disk issues**  
✅ **Use specialized debugging containers**  

---

## 🚀 Next Steps

**Automate everything:**

**Next lesson:** [22 - Docker in CI/CD](22-docker-cicd.md) - Continuous integration and deployment

Debugging is a skill - practice makes perfect! 🔍
