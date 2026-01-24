---
render_with_liquid: false
---
# 09 - Docker Networking

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand Docker networking architecture
- Work with different network drivers
- Create and manage custom networks
- Connect containers to networks
- Expose and publish container ports
- Implement container name resolution
- Troubleshoot network issues

---

## 🌐 Docker Networking Basics

### **Default Behavior:**
```bash
# Run container - automatically connected to default bridge network
docker run -d --name web nginx

# Container gets:
# - Private IP address (172.17.0.x)
# - Can access internet
# - Isolated from host network
# - Cannot be accessed from outside (unless ports published)
```

### **Network Architecture:**
```
┌─────────────────────────────────────────────┐
│              Host Machine                   │
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │   docker0 bridge (172.17.0.1)        │  │
│  └────┬─────────────────┬────────────┬──┘  │
│       │                 │            │     │
│  ┌────▼────┐      ┌────▼────┐  ┌───▼───┐  │
│  │Container│      │Container│  │Container │
│  │172.17.0.2│     │172.17.0.3│ │172.17.0.4│
│  └─────────┘      └─────────┘  └─────────┘ │
│                                             │
└─────────────────────────────────────────────┘
```

---

## 🔌 Network Drivers

### **1. Bridge (Default)**
**Use Case:** Single host, multiple containers

```bash
# Default bridge network
docker run -d --name app nginx

# Containers can:
# ✅ Access internet
# ✅ Communicate via IP
# ❌ Cannot use container names (no DNS)
```

### **2. Host**
**Use Case:** Maximum performance, no isolation

```bash
# Use host network
docker run -d --network host nginx

# Container:
# ✅ Uses host's network directly
# ✅ No port mapping needed
# ✅ Maximum performance
# ❌ No network isolation
# ❌ Port conflicts with host
```

### **3. None**
**Use Case:** No networking, maximum isolation

```bash
# No network access
docker run -d --network none alpine sleep infinity

# Container:
# ❌ No network interface
# ❌ Cannot access internet
# ✅ Maximum isolation
```

### **4. Custom Bridge**
**Use Case:** User-defined networks (RECOMMENDED)

```bash
# Create custom network
docker network create my-network

# Run containers
docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network myapp

# Containers can:
# ✅ Communicate via container names (DNS)
# ✅ Better isolation
# ✅ More control
```

### **5. Overlay**
**Use Case:** Multi-host networking (Docker Swarm)

```bash
# Create overlay network (requires Swarm)
docker network create -d overlay my-overlay

# Containers on different hosts can communicate
```

### **6. Macvlan**
**Use Case:** Container needs its own MAC address

```bash
# Create macvlan network
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  my-macvlan

# Container appears as physical device on network
```

---

## 🔧 Managing Networks

### **List Networks:**
```bash
# List all networks
docker network ls

# Output:
NETWORK ID     NAME      DRIVER    SCOPE
f7ab26d71ddd   bridge    bridge    local
373d49e7d25d   host      host      local
8e8e9b37f5c0   none      null      local

# Filter by driver
docker network ls --filter driver=bridge

# Filter by name
docker network ls --filter name=my
```

### **Inspect Network:**
```bash
# Detailed information
docker network inspect bridge

# Output (JSON):
[
    {
        "Name": "bridge",
        "Driver": "bridge",
        "IPAM": {
            "Config": [{
                "Subnet": "172.17.0.0/16",
                "Gateway": "172.17.0.1"
            }]
        },
        "Containers": {
            "abc123": {
                "Name": "web",
                "IPv4Address": "172.17.0.2/16"
            }
        }
    }
]

# Get specific info
docker network inspect bridge --format='{ {.IPAM.Config} }'
```

### **Create Network:**
```bash
# Basic network
docker network create my-network

# With specific subnet
docker network create \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  my-network

# With IP range
docker network create \
  --subnet=172.20.0.0/16 \
  --ip-range=172.20.10.0/24 \
  --gateway=172.20.0.1 \
  my-network

# With options
docker network create \
  --driver=bridge \
  --subnet=172.20.0.0/16 \
  --opt "com.docker.network.bridge.name"="my-bridge" \
  my-network
```

### **Remove Network:**
```bash
# Remove specific network
docker network rm my-network

# Remove multiple networks
docker network rm net1 net2 net3

# Remove all unused networks
docker network prune

# Remove without confirmation
docker network prune -f
```

---

## 🔗 Connecting Containers

### **Connect at Runtime:**
```bash
# Create network
docker network create app-network

# Run containers on network
docker run -d --name db --network app-network postgres
docker run -d --name web --network app-network nginx

# Containers can now communicate via names:
docker exec web ping db  # Works!
```

### **Connect Existing Container:**
```bash
# Run container
docker run -d --name app nginx

# Connect to network
docker network connect my-network app

# Disconnect from network
docker network disconnect my-network app
```

### **Multiple Networks:**
```bash
# Container can be on multiple networks
docker network create frontend
docker network create backend

docker run -d --name app nginx
docker network connect frontend app
docker network connect backend app

# Check container's networks
docker inspect app --format='{ {json .NetworkSettings.Networks} }'
```

---

## 🌍 Port Publishing

### **Publish Ports:**
```bash
# Map container port to host port
docker run -d -p 8080:80 nginx
# Host:Container (localhost:8080 → container:80)

# Map to specific interface
docker run -d -p 127.0.0.1:8080:80 nginx

# Map to random host port
docker run -d -P nginx

# Multiple ports
docker run -d -p 8080:80 -p 8443:443 nginx

# UDP port
docker run -d -p 53:53/udp dns-server
```

### **Check Published Ports:**
```bash
# View published ports
docker port nginx

# Output:
80/tcp -> 0.0.0.0:8080

# In docker ps
docker ps --format "{ {.Names} }: { {.Ports} }"
```

---

## 🏷️ Container Name Resolution

### **Default Bridge - No DNS:**
```bash
# Default bridge network
docker run -d --name db postgres
docker run -d --name app myapp

# Cannot use names
docker exec app ping db  # ❌ FAILS
docker exec app ping 172.17.0.2  # ✅ Works (but need to know IP)
```

### **Custom Network - DNS Enabled:**
```bash
# Custom bridge network
docker network create my-network
docker run -d --name db --network my-network postgres
docker run -d --name app --network my-network myapp

# Can use container names!
docker exec app ping db  # ✅ Works!
docker exec app curl http://db:5432  # ✅ Works!
```

### **Network Aliases:**
```bash
# Create container with network alias
docker run -d \
  --name web1 \
  --network my-network \
  --network-alias web \
  nginx

docker run -d \
  --name web2 \
  --network my-network \
  --network-alias web \
  nginx

# Both containers reachable as "web" (round-robin DNS)
docker run --rm --network my-network alpine ping web
```

---

## 🔐 Network Isolation

### **Isolated Networks:**
```bash
# Create separate networks
docker network create frontend
docker network create backend

# Frontend containers
docker run -d --name web --network frontend nginx
docker run -d --name lb --network frontend haproxy

# Backend containers
docker run -d --name db --network backend postgres
docker run -d --name cache --network backend redis

# web cannot reach db (different networks)
docker exec web ping db  # ❌ FAILS
```

### **Bridge Networks:**
```bash
# App connects to both networks
docker run -d --name app myapp
docker network connect frontend app
docker network connect backend app

# Now app can communicate with both:
docker exec app ping web   # ✅ Works
docker exec app ping db    # ✅ Works

# But web still cannot reach db
docker exec web ping db  # ❌ FAILS
```

---

## 🎯 Real-World Example

### **Three-Tier Application:**
```bash
# 1. Create networks
docker network create frontend
docker network create backend

# 2. Database (backend only)
docker run -d \
  --name postgres \
  --network backend \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. API server (both networks)
docker run -d \
  --name api \
  --network backend \
  -e DATABASE_URL=postgres://postgres:secret@postgres:5432/mydb \
  myapi:latest

docker network connect frontend api

# 4. Web server (frontend only)
docker run -d \
  --name nginx \
  --network frontend \
  -p 80:80 \
  -e API_URL=http://api:8000 \
  nginx

# Security:
# ✅ nginx can reach api
# ✅ api can reach postgres
# ❌ nginx cannot reach postgres (isolated)
```

---

## 🛠️ Network Troubleshooting

### **Debug Network Issues:**
```bash
# 1. Check container is running
docker ps

# 2. Inspect container network
docker inspect app --format='{ {json .NetworkSettings.Networks} }'

# 3. Check IP address
docker exec app ip addr
docker exec app hostname -i

# 4. Test connectivity
docker exec app ping google.com  # Internet
docker exec app ping other-container  # Container-to-container

# 5. Check DNS resolution
docker exec app nslookup other-container
docker exec app cat /etc/resolv.conf

# 6. Check listening ports
docker exec app netstat -tlnp
docker exec app ss -tlnp

# 7. Check network from host
docker network inspect my-network
```

### **Common Issues:**

**Issue 1: Cannot reach container by name**
```bash
# ❌ Problem: Using default bridge
docker run --name app1 nginx
docker exec app1 ping app2  # Fails

# ✅ Solution: Use custom network
docker network create my-net
docker run --name app1 --network my-net nginx
docker run --name app2 --network my-net nginx
docker exec app1 ping app2  # Works!
```

**Issue 2: Port already in use**
```bash
# ❌ Error: Bind for 0.0.0.0:80 failed: port is already allocated

# Check what's using the port
sudo lsof -i :80
sudo netstat -tlnp | grep :80

# Solution: Use different port
docker run -p 8080:80 nginx
```

**Issue 3: Container cannot access internet**
```bash
# Check DNS
docker exec app cat /etc/resolv.conf

# Check routing
docker exec app ip route

# Test DNS
docker exec app ping 8.8.8.8
docker exec app ping google.com

# Restart Docker daemon if needed
sudo systemctl restart docker
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the default network driver?**
   <details>
   <summary>Answer</summary>
   Bridge
   </details>

2. **Can containers on default bridge use name resolution?**
   <details>
   <summary>Answer</summary>
   No, only IP addresses work. Use custom networks for DNS.
   </details>

3. **How do you publish container port 80 to host port 8080?**
   <details>
   <summary>Answer</summary>
   docker run -p 8080:80 image
   </details>

4. **What network driver gives containers their own MAC address?**
   <details>
   <summary>Answer</summary>
   macvlan
   </details>

5. **How do you connect an existing container to a network?**
   <details>
   <summary>Answer</summary>
   docker network connect network-name container-name
   </details>

---

## 📝 Hands-on Exercise

**Complete networking workflow:**

```bash
# 1. Create custom network
docker network create webapp

# 2. Run database
docker run -d \
  --name db \
  --network webapp \
  -e POSTGRES_PASSWORD=secret \
  postgres:15

# 3. Wait for database to start
sleep 5

# 4. Run application
docker run -d \
  --name api \
  --network webapp \
  -e DB_HOST=db \
  -e DB_PASSWORD=secret \
  -p 8000:8000 \
  nginx  # Replace with your app image

# 5. Test connectivity
docker exec api ping db
docker exec api nslookup db

# 6. Check network
docker network inspect webapp

# 7. Test from outside
curl http://localhost:8000

# 8. Add another container to same network
docker run -d \
  --name cache \
  --network webapp \
  redis

# 9. Verify all can communicate
docker exec api ping cache
docker exec db ping cache

# 10. Cleanup
docker stop api db cache
docker rm api db cache
docker network rm webapp
```

**Challenge - Multi-network setup:**
```bash
# Create isolated environments
docker network create frontend
docker network create backend

# Backend services
docker run -d --name postgres --network backend postgres:15
docker run -d --name redis --network backend redis:alpine

# API (bridge between networks)
docker run -d --name api nginx  # Your API image
docker network connect backend api
docker network connect frontend api

# Frontend
docker run -d --name web --network frontend -p 80:80 nginx

# Test isolation
docker exec web ping api      # ✅ Should work
docker exec web ping postgres # ❌ Should fail (isolated)
docker exec api ping postgres # ✅ Should work

# Cleanup
docker stop web api postgres redis
docker rm web api postgres redis
docker network rm frontend backend
```

---

## 📝 Key Takeaways

✅ **Default bridge: automatic, but no DNS**  
✅ **Custom networks: DNS-enabled, better isolation**  
✅ **Host network: no isolation, max performance**  
✅ **Use custom networks in production**  
✅ **Containers on same network can use names**  
✅ **Publish ports with `-p host:container`**  
✅ **One container can be on multiple networks**  
✅ **Network aliases enable load balancing**  
✅ **Isolate with separate networks**  
✅ **Inspect networks for troubleshooting**  

---

## 🚀 Next Steps

**Ready for multi-container applications?**

**Next lesson:** [10 - Docker Compose Introduction](10-docker-compose-intro.md) - Orchestrate multiple containers

---

## 💡 Pro Tips

**Network Naming:**
```bash
# ✅ GOOD - Descriptive names
docker network create myapp-frontend
docker network create myapp-backend
docker network create myapp-cache

# ❌ BAD - Generic names
docker network create net1
docker network create network
```

**Development Setup:**
```bash
# Create project network
docker network create myproject

# All project containers use same network
docker run --network myproject --name db postgres
docker run --network myproject --name api myapi
docker run --network myproject --name web nginx

# Easy cleanup
docker network rm myproject
```

**Network Debugging Container:**
```bash
# Run container with network tools
docker run -it --rm --network myapp nicolaka/netshoot

# Inside container:
ping other-container
nslookup other-container
curl http://other-container
tcpdump -i eth0
```

**Security:**
```bash
# Principle of least privilege
# Only connect containers that need to communicate
# Use separate networks for different tiers
# Never expose database directly to internet
```

Networking is the glue that connects containers! 🌐
