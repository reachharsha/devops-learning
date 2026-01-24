---
render_with_liquid: false
---
# 16 - Resource Management

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Limit container CPU and memory usage
- Monitor resource consumption
- Optimize container performance
- Handle resource constraints
- Implement resource quotas
- Debug resource issues
- Use cgroups effectively

---

## 💾 Memory Limits

### **Set Memory Limits:**
```bash
# Limit memory to 512MB
docker run -d --memory="512m" nginx

# Memory with swap limit
docker run -d \
  --memory="512m" \
  --memory-swap="1g" \
  nginx

# Disable swap (memory-swap = memory)
docker run -d \
  --memory="512m" \
  --memory-swap="512m" \
  nginx

# Memory reservation (soft limit)
docker run -d \
  --memory="512m" \
  --memory-reservation="256m" \
  nginx

# OOM kill disable (dangerous!)
docker run -d \
  --memory="512m" \
  --oom-kill-disable \
  nginx
```

### **Memory in Compose:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          memory: 512M      # Hard limit
        reservations:
          memory: 256M      # Soft limit (guaranteed)
    
  # Alternative syntax (v2)
  web:
    image: nginx:alpine
    mem_limit: 512m
    mem_reservation: 256m
    memswap_limit: 1g
```

### **What Happens When Limit Exceeded:**
```bash
# Container gets OOM killed
docker run -d --name memory-test --memory="100m" \
  progrium/stress --vm 1 --vm-bytes 150M

# Check exit code
docker inspect memory-test --format='{ {.State.ExitCode} }'
# 137 = killed by OOM

# View events
docker events --filter container=memory-test
```

---

## 🖥️ CPU Limits

### **CPU Shares (Relative Weight):**
```bash
# Default CPU share = 1024

# Container 1: Gets 75% CPU under contention
docker run -d --name cpu-high --cpu-shares=1536 alpine sleep 3600

# Container 2: Gets 25% CPU under contention
docker run -d --name cpu-low --cpu-shares=512 alpine sleep 3600

# No limit when CPUs are idle - both can use 100%
```

### **CPU Quota (Hard Limit):**
```bash
# Limit to 50% of one CPU
docker run -d --cpus="0.5" nginx

# Limit to 1.5 CPUs
docker run -d --cpus="1.5" nginx

# CPU period and quota (advanced)
docker run -d \
  --cpu-period=100000 \
  --cpu-quota=50000 \
  nginx
# 50000/100000 = 50% of one CPU
```

### **CPU Pinning:**
```bash
# Pin to specific CPUs (0 and 1)
docker run -d --cpuset-cpus="0,1" nginx

# Pin to CPU range
docker run -d --cpuset-cpus="0-3" nginx

# Pin to specific memory nodes (NUMA)
docker run -d --cpuset-mems="0,1" nginx
```

### **CPU in Compose:**
```yaml
version: '3.8'

services:
  app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          cpus: '1.5'        # Hard limit: 1.5 CPUs max
        reservations:
          cpus: '0.5'        # Soft limit: 0.5 CPUs guaranteed
    
  # Alternative syntax (v2)
  web:
    image: nginx:alpine
    cpu_shares: 1024
    cpu_quota: 50000
    cpu_period: 100000
    cpuset: "0,1"
```

---

## 💿 Disk I/O Limits

### **Block I/O Weight:**
```bash
# Default weight = 500 (range: 10-1000)

# High I/O priority
docker run -d --blkio-weight=1000 nginx

# Low I/O priority
docker run -d --blkio-weight=100 nginx
```

### **Device Read/Write Limits:**
```bash
# Limit read to 10 MB/s
docker run -d \
  --device-read-bps=/dev/sda:10mb \
  nginx

# Limit write to 5 MB/s
docker run -d \
  --device-write-bps=/dev/sda:5mb \
  nginx

# Limit read IOPS
docker run -d \
  --device-read-iops=/dev/sda:1000 \
  nginx

# Limit write IOPS
docker run -d \
  --device-write-iops=/dev/sda:500 \
  nginx
```

### **Storage Driver Options:**
```bash
# Set size limit for container storage
docker run -d \
  --storage-opt size=10G \
  nginx

# Only works with certain storage drivers (overlay2, devicemapper)
```

---

## 📊 Monitoring Resources

### **docker stats:**
```bash
# Live resource usage
docker stats

# Specific containers
docker stats container1 container2

# No streaming (one-time)
docker stats --no-stream

# Format output
docker stats --format "table { {.Container}}\t{ {.CPUPerc}}\t{ {.MemUsage}}"

# All containers (including stopped)
docker stats --all
```

### **Inspect Container Resources:**
```bash
# Get memory stats
docker inspect container \
  --format='{ {.HostConfig.Memory} }'

# Get CPU stats
docker inspect container \
  --format='{ {.HostConfig.CpuShares} }'

# Get all resource limits
docker inspect container | jq '.[0].HostConfig | {
  Memory,
  MemoryReservation,
  MemorySwap,
  CpuShares,
  NanoCpus,
  CpusetCpus
}'
```

### **System-wide Resource Usage:**
```bash
# Docker system info
docker system df

# Detailed usage
docker system df -v

# Events
docker events --filter type=container

# Resource usage of Docker daemon
docker system info | grep -A 20 "Runtime"
```

---

## 🔍 Advanced Monitoring

### **cAdvisor (Container Advisor):**
```yaml
# docker-compose.yml
version: '3.8'

services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    container_name: cadvisor
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
      - /dev/disk:/dev/disk:ro
    privileged: true
    devices:
      - /dev/kmsg
```

### **Prometheus + Grafana:**
```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

volumes:
  prometheus-data:
  grafana-data:
```

```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['cadvisor:8080']
```

---

## 🎯 Resource Optimization

### **Right-Sizing Containers:**
```bash
# 1. Monitor baseline usage
docker stats myapp --no-stream

# 2. Run load test
# Note peak CPU and memory

# 3. Set limits with headroom
docker run -d \
  --name myapp \
  --memory="512m" \          # Peak was 350MB, set to 512MB
  --memory-reservation="256m" \
  --cpus="0.5" \             # Peak was 35%, set to 0.5
  myapp:latest
```

### **Optimize Images:**
```dockerfile
# Before: 1.2GB
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "index.js"]

# After: 150MB
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production && npm cache clean --force
COPY . .
CMD ["node", "index.js"]
```

### **Multi-container Resource Allocation:**
```yaml
version: '3.8'

services:
  # Frontend - lightweight
  web:
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: '0.25'
          memory: 128M
        reservations:
          cpus: '0.1'
          memory: 64M

  # Application - moderate
  app:
    image: myapp:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M

  # Database - heavy
  postgres:
    image: postgres:15-alpine
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
    shm_size: 256m  # Shared memory for Postgres
```

---

## 🐛 Debugging Resource Issues

### **High Memory Usage:**
```bash
# 1. Check memory usage
docker stats myapp --no-stream

# 2. Inspect process inside container
docker exec myapp ps aux --sort=-%mem | head

# 3. Get memory map
docker exec myapp cat /proc/meminfo

# 4. Check for memory leaks
docker exec myapp top -b -n 1

# 5. Heap dump (Node.js)
docker exec myapp node --expose-gc --heap-snapshot index.js
```

### **High CPU Usage:**
```bash
# 1. Check CPU usage
docker stats myapp --no-stream

# 2. Top processes in container
docker exec myapp top -b -n 1

# 3. Profile application (Node.js)
docker exec myapp node --prof index.js

# 4. Check for infinite loops
docker exec myapp strace -p 1

# 5. Thread count
docker exec myapp ps -eLf | wc -l
```

### **Disk Space Issues:**
```bash
# 1. Check container disk usage
docker exec myapp df -h

# 2. Find large files
docker exec myapp du -sh /* | sort -rh | head -10

# 3. Check logs size
docker inspect myapp | grep LogPath
ls -lh $(docker inspect myapp | grep LogPath | cut -d'"' -f4)

# 4. Check layer sizes
docker history myapp:latest --no-trunc

# 5. Clean up inside container
docker exec myapp sh -c 'rm -rf /tmp/* /var/tmp/*'
```

---

## 🎯 Production Resource Patterns

### **Database Container:**
```yaml
services:
  postgres:
    image: postgres:15-alpine
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 4G
        reservations:
          cpus: '1.0'
          memory: 2G
    shm_size: 512m
    environment:
      POSTGRES_SHARED_BUFFERS: 1GB
      POSTGRES_EFFECTIVE_CACHE_SIZE: 3GB
      POSTGRES_WORK_MEM: 16MB
    volumes:
      - postgres-data:/var/lib/postgresql/data
```

### **Redis Container:**
```yaml
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    command: >
      redis-server
      --maxmemory 450mb
      --maxmemory-policy allkeys-lru
      --save ""
```

### **Application Container:**
```yaml
services:
  app:
    image: myapp:latest
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    environment:
      NODE_OPTIONS: "--max-old-space-size=896"  # 90% of 1GB
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 5s
      retries: 3
```

---

## 🎯 Quick Check: Do You Understand?

1. **What happens when a container exceeds memory limit?**
   <details>
   <summary>Answer</summary>
   Container gets OOM (Out Of Memory) killed, exit code 137
   </details>

2. **What's the difference between CPU shares and CPU quota?**
   <details>
   <summary>Answer</summary>
   Shares = relative weight under contention; Quota = hard limit always enforced
   </details>

3. **How do you monitor container resource usage?**
   <details>
   <summary>Answer</summary>
   docker stats, cAdvisor, Prometheus, or docker inspect
   </details>

4. **What's memory reservation vs memory limit?**
   <details>
   <summary>Answer</summary>
   Reservation = guaranteed minimum (soft); Limit = maximum allowed (hard)
   </details>

5. **Why set resource limits in production?**
   <details>
   <summary>Answer</summary>
   Prevent one container from consuming all resources and affecting others
   </details>

---

## 📝 Hands-on Exercise

**Resource management practice:**

```bash
# 1. Start container without limits
docker run -d --name unlimited nginx
docker stats unlimited --no-stream

# 2. Start with memory limit
docker run -d --name limited-mem \
  --memory="100m" \
  --memory-reservation="50m" \
  nginx
docker stats limited-mem --no-stream

# 3. Start with CPU limit
docker run -d --name limited-cpu \
  --cpus="0.5" \
  nginx
docker stats limited-cpu --no-stream

# 4. Start with both limits
docker run -d --name limited-both \
  --memory="100m" \
  --cpus="0.5" \
  nginx
docker stats limited-both --no-stream

# 5. Stress test
docker run -d --name stress-test \
  --memory="256m" \
  --cpus="0.5" \
  progrium/stress \
  --cpu 2 --vm 1 --vm-bytes 512M --timeout 30s

# Watch it get killed for exceeding memory
docker stats stress-test
docker wait stress-test
docker inspect stress-test --format='{ {.State.ExitCode} }'

# 6. Monitoring with cAdvisor
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  app:
    image: nginx:alpine
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 256M

  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker:/var/lib/docker:ro
    privileged: true
EOF

docker-compose up -d
echo "Visit http://localhost:8080"

# 7. Cleanup
docker-compose down
docker rm -f unlimited limited-mem limited-cpu limited-both stress-test
rm docker-compose.yml
```

---

## 📝 Key Takeaways

✅ **Always set resource limits in production**  
✅ **Memory limits prevent OOM issues**  
✅ **CPU limits prevent resource starvation**  
✅ **Monitor resources continuously**  
✅ **Right-size based on actual usage**  
✅ **Use reservations for critical services**  
✅ **Optimize images to reduce resource needs**  
✅ **Test limits before production**  
✅ **Set both limits and reservations**  
✅ **Monitor with proper tools (cAdvisor, Prometheus)**  

---

## 🚀 Next Steps

**Implement monitoring and logging:**

**Next lesson:** [17 - Logging & Monitoring](17-logging-monitoring.md) - Centralized logging and metrics

---

## 💡 Pro Tips

**Resource Calculation:**
```bash
# Calculate total resources needed
# If you have:
# - 3 app containers: 1GB each = 3GB
# - 1 database: 4GB
# - 1 cache: 512MB
# Total: 7.5GB + 2GB buffer = 10GB RAM needed

# CPU calculation:
# - 3 app containers: 1 CPU each = 3 CPUs
# - 1 database: 2 CPUs
# - 1 cache: 0.5 CPU
# Total: 5.5 CPUs + 1 CPU buffer = 7 CPUs needed
```

**Memory Best Practices:**
```yaml
# For Java applications
services:
  java-app:
    image: myapp:latest
    deploy:
      resources:
        limits:
          memory: 2G
    environment:
      # Set JVM heap to 75% of container memory
      JAVA_OPTS: -Xmx1536m -Xms1536m
```

**Node.js Memory:**
```yaml
services:
  node-app:
    image: node-app:latest
    deploy:
      resources:
        limits:
          memory: 1G
    environment:
      # Set Node heap to 90% of container memory
      NODE_OPTIONS: --max-old-space-size=896
```

Right-sizing saves money and improves reliability! 💰
