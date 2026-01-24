---
render_with_liquid: false
---
# 09 - Kubernetes YAML

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Kubernetes manifest structure
- Common resource types (Pod, Deployment, Service)
- Labels and selectors
- ConfigMaps and Secrets
- Real-world K8s YAML patterns

---

## ☸️ What is Kubernetes YAML?

**Kubernetes** uses YAML to define:
- Workloads (Pods, Deployments, StatefulSets)
- Services (Networking)
- Configuration (ConfigMaps, Secrets)
- Storage (Volumes, PersistentVolumeClaims)

**Real-World Analogy:**
Like blueprints for a building - specifies what to build, how many, and how they connect.

---

## 📋 Basic Manifest Structure

```yaml
apiVersion: v1          # API version
kind: Pod               # Resource type
metadata:               # Resource metadata
  name: my-pod
  labels:
    app: myapp
spec:                   # Resource specification
  containers:
  - name: container1
    image: nginx
```

**Every K8s manifest has:**
1. `apiVersion` - Which API to use
2. `kind` - Type of resource
3. `metadata` - Name, labels, annotations
4. `spec` - Desired state

---

## 🎨 Pod - Basic Unit

### **Simple Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### **Multi-Container Pod:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  # Main application
  - name: app
    image: myapp:latest
    ports:
    - containerPort: 8080
    env:
    - name: DATABASE_URL
      value: "postgresql://db:5432/myapp"
    resources:
      limits:
        memory: "512Mi"
        cpu: "500m"
      requests:
        memory: "256Mi"
        cpu: "250m"
        
  # Sidecar - logging agent
  - name: log-agent
    image: fluent/fluentd:latest
    volumeMounts:
    - name: logs
      mountPath: /var/log
      
  volumes:
  - name: logs
    emptyDir: {}
```

---

## 🚀 Deployment - Production Workload

### **Basic Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # Number of Pods
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### **Production Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
  namespace: production
  labels:
    app: api
    tier: backend
spec:
  replicas: 5
  
  # Rolling update strategy
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
      
  selector:
    matchLabels:
      app: api
      tier: backend
      
  template:
    metadata:
      labels:
        app: api
        tier: backend
        version: "2.1.0"
    spec:
      containers:
      - name: api
        image: myregistry/api:2.1.0
        
        ports:
        - name: http
          containerPort: 8080
          protocol: TCP
          
        env:
        - name: NODE_ENV
          value: "production"
        - name: PORT
          value: "8080"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
              
        resources:
          limits:
            memory: "1Gi"
            cpu: "1000m"
          requests:
            memory: "512Mi"
            cpu: "500m"
            
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
          
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          
        volumeMounts:
        - name: config
          mountPath: /app/config
        - name: logs
          mountPath: /var/log
          
      volumes:
      - name: config
        configMap:
          name: api-config
      - name: logs
        emptyDir: {}
```

---

## 🌐 Service - Networking

### **ClusterIP (Internal):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: ClusterIP  # Default, internal only
  selector:
    app: api
  ports:
  - port: 80        # Service port
    targetPort: 8080  # Container port
    protocol: TCP
```

### **LoadBalancer (External):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - name: http
    port: 80
    targetPort: 80
  - name: https
    port: 443
    targetPort: 443
```

### **NodePort (External via Node):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 8080
    targetPort: 8080
    nodePort: 30080  # 30000-32767 range
```

---

## 🗄️ ConfigMap - Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
data:
  # Key-value pairs
  log_level: "info"
  api_url: "https://api.example.com"
  max_connections: "100"
  
  # File content
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    database.pool.size=10
    
  nginx.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://backend:8080;
      }
    }
```

**Using ConfigMap:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp
    
    # Environment from ConfigMap
    envFrom:
    - configMapRef:
        name: app-config
        
    # Individual env var
    env:
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyFrom:
          name: app-config
          key: log_level
          
    # Mount as files
    volumeMounts:
    - name: config
      mountPath: /etc/config
      
  volumes:
  - name: config
    configMap:
      name: app-config
```

---

## 🔐 Secret - Sensitive Data

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  # Base64 encoded values
  username: YWRtaW4=        # admin
  password: c2VjcmV0MTIz    # secret123
  url: cG9zdGdyZXNxbDovL2RiOjU0MzIvbXlhcHA=  # postgresql://db:5432/myapp
  
# Or use stringData (not base64)
stringData:
  api-key: "sk-abc123xyz"
```

**Using Secret:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  containers:
  - name: app
    image: myapp
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-credentials
          key: password
```

---

## 📦 Complete Application Example

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  namespace: production
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      volumes:
      - name: config
        configMap:
          name: nginx-config
---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  namespace: production
spec:
  type: LoadBalancer
  selector:
    app: web
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: production
data:
  nginx.conf: |
    events {
      worker_connections 1024;
    }
    http {
      server {
        listen 80;
        location / {
          proxy_pass http://api-service:8080;
        }
      }
    }
```

---

## 🎯 Quick Check: Do You Understand?

1. **What are the 4 required fields in a K8s manifest?**
   <details>
   <summary>Answer</summary>
   apiVersion, kind, metadata, spec
   </details>

2. **What's the difference between Pod and Deployment?**
   <details>
   <summary>Answer</summary>
   Pod is single instance, Deployment manages multiple Pods with scaling/updates
   </details>

3. **How do you expose a Deployment externally?**
   <details>
   <summary>Answer</summary>
   Create a Service with type: LoadBalancer or NodePort
   </details>

4. **ConfigMap vs Secret?**
   <details>
   <summary>Answer</summary>
   ConfigMap: non-sensitive config, Secret: sensitive data (base64 encoded)
   </details>

---

## 📝 Hands-on Exercise

**Create a complete microservice:**

Create `microservice.yaml`:
```yaml
# Namespace
apiVersion: v1
kind: Namespace
metadata:
  name: myapp
---
# ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: myapp
data:
  LOG_LEVEL: "info"
  API_URL: "https://api.example.com"
---
# Secret
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: myapp
stringData:
  DB_PASSWORD: "secret123"
  API_KEY: "sk-abc123"
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  namespace: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:latest
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: app-config
        - secretRef:
            name: app-secrets
        resources:
          limits:
            memory: "512Mi"
            cpu: "500m"
          requests:
            memory: "256Mi"
            cpu: "250m"
---
# Service
apiVersion: v1
kind: Service
metadata:
  name: app-service
  namespace: myapp
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

**Tasks:**
1. Apply: `kubectl apply -f microservice.yaml`
2. Check: `kubectl get all -n myapp`
3. Add health checks to Deployment
4. Add a PersistentVolumeClaim for storage

---

## ⚠️ Common Mistakes

### **1. Selector Mismatch:**
```yaml
# ❌ WRONG - labels don't match
spec:
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: web  # Different!
```

### **2. Missing Namespace:**
```yaml
# ⚠️ Goes to 'default' namespace
metadata:
  name: app

# ✅ BETTER - explicit namespace
metadata:
  name: app
  namespace: production
```

### **3. Resource Limits:**
```yaml
# ❌ BAD - no limits (can consume all resources)
containers:
- name: app
  image: myapp

# ✅ GOOD - set limits
containers:
- name: app
  image: myapp
  resources:
    limits:
      memory: "512Mi"
      cpu: "500m"
```

---

## 📝 Key Takeaways

✅ **Every manifest needs: apiVersion, kind, metadata, spec**  
✅ **Deployments manage Pods (preferred over bare Pods)**  
✅ **Services expose Pods to network**  
✅ **ConfigMaps for configuration, Secrets for sensitive data**  
✅ **Labels connect resources (selector -> labels)**  
✅ **Always set resource limits**  
✅ **Use health probes for reliability**  

---

## 🚀 Next Steps

**Next lesson:** [10 - CI/CD Pipelines](10-cicd-pipelines.md) - GitHub Actions, GitLab CI YAML

---

## 💡 Pro Tips

**Useful Commands:**
```bash
# Apply manifest
kubectl apply -f app.yaml

# Get resources
kubectl get pods,svc,deploy

# Describe resource
kubectl describe pod my-pod

# View logs
kubectl logs my-pod

# Edit live resource
kubectl edit deployment my-app

# Dry run (validate)
kubectl apply -f app.yaml --dry-run=client
```

**YAML Multi-Document:**
```yaml
# app.yaml - multiple resources in one file
apiVersion: v1
kind: ConfigMap
metadata:
  name: config
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
```

Kubernetes + YAML = Powerful automation! ☸️
