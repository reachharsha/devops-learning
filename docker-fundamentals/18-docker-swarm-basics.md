---
render_with_liquid: false
---
# 18 - Docker Swarm Basics

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand container orchestration concepts
- Initialize and manage Docker Swarm clusters
- Deploy services to Swarm
- Scale services up and down
- Implement rolling updates
- Configure service networking and load balancing
- Handle node failures

---

## 🐝 What is Docker Swarm?

**Docker Swarm** = Native Docker clustering and orchestration tool.

### **Key Concepts:**
```
Swarm Cluster
├── Manager Nodes (Control plane)
│   ├── Leader (Raft consensus)
│   ├── Reachable
│   └── Reachable
└── Worker Nodes (Run containers)
    ├── Worker 1
    ├── Worker 2
    └── Worker 3

Service → Tasks → Containers
```

### **Swarm vs Standalone:**
```bash
# Standalone Docker
docker run -d nginx

# Docker Swarm
docker service create --replicas 3 nginx
# ✅ Automatic load balancing
# ✅ Self-healing
# ✅ Rolling updates
# ✅ Service discovery
```

---

## 🚀 Initialize Swarm

### **Create Single-Node Swarm:**
```bash
# Initialize Swarm
docker swarm init

# Output shows:
# - Swarm initialized
# - Current node is now a manager
# - Worker join token

# View swarm info
docker info | grep Swarm
# Swarm: active

# List nodes
docker node ls
# ID      HOSTNAME    STATUS   AVAILABILITY   MANAGER STATUS
# abc123  docker-1    Ready    Active         Leader
```

### **Multi-Node Swarm:**
```bash
# On Manager Node
docker swarm init --advertise-addr 192.168.1.10

# Output:
# To add a worker to this swarm, run the following command:
#   docker swarm join --token SWMTKN-1-xxx 192.168.1.10:2377

# To add a manager to this swarm, run:
#   docker swarm join-token manager

# On Worker Nodes
docker swarm join \
  --token SWMTKN-1-xxx \
  192.168.1.10:2377

# Back on Manager - view all nodes
docker node ls
```

### **Add More Managers (High Availability):**
```bash
# On existing manager, get manager token
docker swarm join-token manager

# On new manager node
docker swarm join \
  --token SWMTKN-1-manager-token-xxx \
  192.168.1.10:2377

# Best practice: 3 or 5 managers (odd number for quorum)
```

---

## 🔧 Managing Nodes

### **Node Operations:**
```bash
# List nodes
docker node ls

# Inspect node
docker node inspect node-id
docker node inspect self

# Promote worker to manager
docker node promote worker-node

# Demote manager to worker
docker node demote manager-node

# Update node availability
docker node update --availability drain node-id    # No new tasks
docker node update --availability active node-id   # Accept tasks
docker node update --availability pause node-id    # No new, keep existing

# Add labels to nodes
docker node update --label-add environment=production node-id
docker node update --label-add disktype=ssd node-id

# Remove node (on the node itself)
docker swarm leave

# Remove node from cluster (on manager)
docker node rm node-id

# Force remove (if node is down)
docker node rm --force node-id
```

---

## 📦 Services

### **Create Service:**
```bash
# Basic service
docker service create --name web nginx

# With replicas
docker service create \
  --name web \
  --replicas 3 \
  nginx

# With published port
docker service create \
  --name web \
  --replicas 3 \
  --publish published=80,target=80 \
  nginx

# With environment variables
docker service create \
  --name api \
  --replicas 5 \
  --env NODE_ENV=production \
  --env PORT=3000 \
  myapp:latest

# With resource limits
docker service create \
  --name app \
  --replicas 3 \
  --limit-cpu 0.5 \
  --limit-memory 512M \
  --reserve-cpu 0.25 \
  --reserve-memory 256M \
  myapp:latest

# With restart policy
docker service create \
  --name app \
  --replicas 3 \
  --restart-condition on-failure \
  --restart-max-attempts 3 \
  --restart-delay 10s \
  myapp:latest
```

### **List and Inspect Services:**
```bash
# List services
docker service ls

# Inspect service
docker service inspect web
docker service inspect --pretty web

# List tasks (containers) of a service
docker service ps web

# View service logs
docker service logs web
docker service logs -f web
docker service logs --tail 100 web
```

---

## 📈 Scaling Services

### **Scale Up/Down:**
```bash
# Scale to 5 replicas
docker service scale web=5

# Scale multiple services
docker service scale web=5 api=10

# Update service replicas
docker service update --replicas 10 web

# View scaling in action
watch docker service ps web
```

### **Global Services:**
```bash
# Run one replica on EVERY node
docker service create \
  --name monitor \
  --mode global \
  prom/node-exporter

# Useful for:
# - Monitoring agents
# - Log collectors
# - Security scanners
```

---

## 🔄 Rolling Updates

### **Update Service:**
```bash
# Update image
docker service update --image nginx:1.25 web

# Update with configuration
docker service update \
  --image myapp:v2 \
  --update-parallelism 2 \        # Update 2 at a time
  --update-delay 10s \            # Wait 10s between updates
  --update-failure-action rollback \
  --update-monitor 60s \          # Monitor for 60s
  web

# Update environment variable
docker service update \
  --env-add NEW_VAR=value \
  web

# Update resource limits
docker service update \
  --limit-cpu 1 \
  --limit-memory 1G \
  web

# Update published port
docker service update \
  --publish-add 8080:80 \
  web
```

### **Rollback:**
```bash
# Manual rollback to previous version
docker service rollback web

# Automatic rollback on failure
docker service update \
  --image myapp:v2 \
  --update-failure-action rollback \
  --rollback-parallelism 2 \
  --rollback-delay 5s \
  web
```

---

## 🌐 Networking

### **Overlay Networks:**
```bash
# Create overlay network
docker network create \
  --driver overlay \
  my-overlay

# Create service on network
docker service create \
  --name web \
  --network my-overlay \
  --replicas 3 \
  nginx

# Add service to existing network
docker service update \
  --network-add my-overlay \
  web

# Services on same overlay can communicate by service name
# web → api (automatic DNS resolution)
```

### **Ingress Network (Load Balancing):**
```bash
# All published ports use ingress network (routing mesh)
docker service create \
  --name web \
  --replicas 3 \
  --publish published=80,target=80 \
  nginx

# Request to ANY node's port 80 → load balanced across all replicas
# Even nodes not running the service!

# Bypass ingress (host mode)
docker service create \
  --name web \
  --publish published=80,target=80,mode=host \
  nginx
# Only works on nodes actually running the container
```

---

## 🔐 Secrets Management

### **Create and Use Secrets:**
```bash
# Create secret from stdin
echo "mySecretPassword" | docker secret create db_password -

# Create secret from file
docker secret create ssl_cert ./cert.pem

# List secrets
docker secret ls

# Inspect secret (value is hidden)
docker secret inspect db_password

# Use secret in service
docker service create \
  --name db \
  --secret db_password \
  postgres

# Secret available at: /run/secrets/db_password

# Remove secret
docker secret rm db_password
```

### **Secrets in Application:**
```bash
# Example: PostgreSQL with secret
docker secret create db_password <(echo "supersecret123")

docker service create \
  --name postgres \
  --secret db_password \
  --env POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  postgres:15

# Inside container:
# cat /run/secrets/db_password
# supersecret123
```

---

## 📋 Configs

### **Manage Configurations:**
```bash
# Create config
docker config create nginx_config nginx.conf

# From stdin
echo "server { listen 80; }" | docker config create nginx_config -

# List configs
docker config ls

# Inspect config
docker config inspect nginx_config

# Use config in service
docker service create \
  --name web \
  --config source=nginx_config,target=/etc/nginx/nginx.conf \
  nginx

# Update service with new config
docker config create nginx_config_v2 nginx-v2.conf
docker service update \
  --config-rm nginx_config \
  --config-add source=nginx_config_v2,target=/etc/nginx/nginx.conf \
  web
```

---

## 🎯 Service Placement

### **Placement Constraints:**
```bash
# Run only on nodes with specific label
docker service create \
  --name db \
  --constraint 'node.labels.disktype==ssd' \
  postgres

# Run on manager nodes only
docker service create \
  --name monitor \
  --constraint 'node.role==manager' \
  prometheus

# Run on specific hostname
docker service create \
  --name special \
  --constraint 'node.hostname==worker-1' \
  myapp

# Multiple constraints (AND logic)
docker service create \
  --name app \
  --constraint 'node.labels.environment==production' \
  --constraint 'node.labels.region==us-east' \
  myapp
```

### **Placement Preferences:**
```bash
# Spread across zones
docker service create \
  --name app \
  --replicas 9 \
  --placement-pref 'spread=node.labels.zone' \
  myapp

# If you have zones: zone=a, zone=b, zone=c
# Will distribute 3 replicas per zone
```

---

## 📊 Stack Deployment

### **Deploy Multi-Service Stack:**
```yaml
# stack.yml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    networks:
      - frontend
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
    configs:
      - source: nginx_config
        target: /etc/nginx/nginx.conf

  api:
    image: myapi:latest
    networks:
      - frontend
      - backend
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
        reservations:
          cpus: '0.25'
          memory: 256M
    secrets:
      - api_key
    environment:
      - NODE_ENV=production

  db:
    image: postgres:15-alpine
    networks:
      - backend
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.disktype==ssd
    secrets:
      - db_password
    environment:
      - POSTGRES_PASSWORD_FILE=/run/secrets/db_password
    volumes:
      - db-data:/var/lib/postgresql/data

networks:
  frontend:
    driver: overlay
  backend:
    driver: overlay

volumes:
  db-data:

secrets:
  db_password:
    external: true
  api_key:
    external: true

configs:
  nginx_config:
    external: true
```

```bash
# Create secrets and configs first
echo "dbpass123" | docker secret create db_password -
echo "apikey456" | docker secret create api_key -
docker config create nginx_config nginx.conf

# Deploy stack
docker stack deploy -c stack.yml myapp

# List stacks
docker stack ls

# List stack services
docker stack services myapp

# List stack tasks
docker stack ps myapp

# View service logs
docker service logs myapp_web

# Update stack (redeploy with changes)
docker stack deploy -c stack.yml myapp

# Remove stack
docker stack rm myapp
```

---

## 🔍 Monitoring Swarm

### **Health Checks:**
```bash
# Service with health check
docker service create \
  --name web \
  --replicas 3 \
  --health-cmd "curl -f http://localhost/ || exit 1" \
  --health-interval 30s \
  --health-timeout 10s \
  --health-retries 3 \
  nginx

# In stack file
services:
  web:
    image: nginx
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
```

### **Monitoring Commands:**
```bash
# Watch service tasks
watch docker service ps web

# View service logs
docker service logs -f web

# Inspect task
docker inspect task-id

# Node resource usage
docker node ps node-id
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the minimum number of managers for HA?**
   <details>
   <summary>Answer</summary>
   3 managers (odd number for quorum - can tolerate 1 failure)
   </details>

2. **How do you scale a service to 10 replicas?**
   <details>
   <summary>Answer</summary>
   docker service scale servicename=10
   </details>

3. **What's the routing mesh?**
   <details>
   <summary>Answer</summary>
   Ingress network that load balances requests to published ports across all service replicas
   </details>

4. **Where are secrets mounted in containers?**
   <details>
   <summary>Answer</summary>
   /run/secrets/secret_name
   </details>

5. **How do you rollback a service?**
   <details>
   <summary>Answer</summary>
   docker service rollback servicename
   </details>

---

## 📝 Hands-on Exercise

**Create production Swarm cluster:**

```bash
# 1. Initialize Swarm
docker swarm init

# 2. Create overlay network
docker network create --driver overlay app-net

# 3. Create secrets
echo "dbpass123" | docker secret create db_password -

# 4. Create service
docker service create \
  --name web \
  --replicas 3 \
  --network app-net \
  --publish 80:80 \
  --update-delay 10s \
  --update-parallelism 1 \
  --rollback-parallelism 1 \
  --health-cmd "wget -q --spider http://localhost || exit 1" \
  --health-interval 30s \
  nginx:alpine

# 5. Watch deployment
docker service ps web

# 6. Test load balancing
for i in {1..10}; do curl http://localhost; done

# 7. Scale up
docker service scale web=5

# 8. Rolling update
docker service update --image nginx:1.25 web

# 9. Create database service
docker service create \
  --name postgres \
  --network app-net \
  --secret db_password \
  --env POSTGRES_PASSWORD_FILE=/run/secrets/db_password \
  --constraint 'node.role==manager' \
  --replicas 1 \
  postgres:15-alpine

# 10. Deploy stack
cat > stack.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    networks:
      - webnet
    deploy:
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s

networks:
  webnet:
    driver: overlay
EOF

docker stack deploy -c stack.yml mystack

# 11. View stack
docker stack services mystack
docker stack ps mystack

# 12. Cleanup
docker stack rm mystack
docker service rm web postgres
docker secret rm db_password
docker network rm app-net
docker swarm leave --force
rm stack.yml
```

---

## 📝 Key Takeaways

✅ **Swarm provides native orchestration**  
✅ **Services are scaled across nodes automatically**  
✅ **Routing mesh provides load balancing**  
✅ **Rolling updates enable zero-downtime deployments**  
✅ **Secrets and configs are managed securely**  
✅ **Overlay networks enable service communication**  
✅ **Stacks deploy multi-service applications**  
✅ **High availability with multiple managers**  
✅ **Self-healing replaces failed containers**  
✅ **Placement constraints control task distribution**  

---

## 🚀 Next Steps

**Compare with Kubernetes:**

**Next lesson:** [19 - Kubernetes Introduction](19-kubernetes-intro.md) - The industry standard orchestrator

Swarm is simple, powerful, and built into Docker! 🐝
