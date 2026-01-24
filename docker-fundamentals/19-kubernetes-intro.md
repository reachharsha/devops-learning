---
render_with_liquid: false
---
# 19 - Kubernetes Introduction

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand Kubernetes architecture
- Compare Kubernetes vs Docker Swarm
- Install and configure Minikube/Kind
- Deploy applications to Kubernetes
- Understand Pods, Deployments, Services
- Use kubectl effectively
- Prepare Docker images for Kubernetes

---

## ☸️ What is Kubernetes?

**Kubernetes (K8s)** = Open-source container orchestration platform originally developed by Google.

### **Why Kubernetes?**
```
✅ Industry standard orchestration
✅ Massive ecosystem and community
✅ Runs on any cloud (AWS, GCP, Azure) or on-prem
✅ Advanced features (auto-scaling, self-healing, rolling updates)
✅ Declarative configuration
✅ Service discovery and load balancing
✅ Storage orchestration
✅ Secrets and configuration management
```

### **Kubernetes vs Docker Swarm:**
```
Docker Swarm                    Kubernetes
─────────────────              ──────────────────
✅ Simpler to learn            ✅ Industry standard
✅ Built into Docker           ✅ More features
✅ Faster setup                ✅ Better ecosystem
✅ Good for small teams        ✅ Enterprise ready
✅ Docker-specific             ✅ Cloud-native standard
❌ Limited features            ❌ Steeper learning curve
❌ Smaller ecosystem           ❌ Complex setup
```

---

## 🏗️ Kubernetes Architecture

### **Control Plane (Master):**
```
Control Plane
├── API Server (kube-apiserver)
│   └── RESTful API for cluster
├── Scheduler (kube-scheduler)
│   └── Assigns pods to nodes
├── Controller Manager (kube-controller-manager)
│   └── Maintains desired state
└── etcd
    └── Distributed key-value store (cluster state)
```

### **Worker Nodes:**
```
Worker Node
├── kubelet
│   └── Manages containers on node
├── kube-proxy
│   └── Network proxy, load balancing
└── Container Runtime (Docker, containerd, CRI-O)
    └── Runs containers
```

### **Key Concepts:**
```
Cluster
└── Nodes (Worker Machines)
    └── Pods (Smallest deployable units)
        └── Containers (Your application)

Deployment → ReplicaSet → Pods → Containers
```

---

## 🚀 Local Kubernetes Setup

### **Minikube (Recommended for Learning):**
```bash
# Install Minikube (macOS)
brew install minikube

# Install Minikube (Linux)
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# Start Minikube
minikube start

# Start with specific driver
minikube start --driver=docker

# Verify
kubectl cluster-info
kubectl get nodes

# Stop Minikube
minikube stop

# Delete cluster
minikube delete
```

### **Kind (Kubernetes in Docker):**
```bash
# Install Kind (macOS)
brew install kind

# Install Kind (Linux)
curl -Lo ./kind https://kind.sigs.k8s.io/dl/latest/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Create cluster
kind create cluster --name my-cluster

# Create multi-node cluster
cat > kind-config.yaml << 'EOF'
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF

kind create cluster --config kind-config.yaml --name multi-node

# List clusters
kind get clusters

# Delete cluster
kind delete cluster --name my-cluster
```

### **kubectl (Kubernetes CLI):**
```bash
# Install kubectl (macOS)
brew install kubectl

# Install kubectl (Linux)
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client

# Basic commands
kubectl cluster-info
kubectl get nodes
kubectl get all
kubectl config view
kubectl config get-contexts
```

---

## 📦 Pods

**Pod** = Smallest deployable unit, contains one or more containers.

### **Create Pod Imperatively:**
```bash
# Run nginx pod
kubectl run nginx --image=nginx

# List pods
kubectl get pods

# Detailed pod info
kubectl get pods -o wide

# Describe pod
kubectl describe pod nginx

# View pod logs
kubectl logs nginx

# Follow logs
kubectl logs -f nginx

# Execute command in pod
kubectl exec nginx -- ls /

# Interactive shell
kubectl exec -it nginx -- bash

# Delete pod
kubectl delete pod nginx
```

### **Create Pod Declaratively:**
```yaml
# pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
    environment: production
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    ports:
    - containerPort: 80
    resources:
      limits:
        cpu: "500m"
        memory: "512Mi"
      requests:
        cpu: "250m"
        memory: "256Mi"
```

```bash
# Create pod from YAML
kubectl apply -f pod.yaml

# View pod
kubectl get pod nginx

# Update pod (edit YAML and reapply)
kubectl apply -f pod.yaml

# Delete pod
kubectl delete -f pod.yaml
# or
kubectl delete pod nginx
```

---

## 🚀 Deployments

**Deployment** = Manages ReplicaSets and Pods, provides declarative updates.

### **Create Deployment:**
```bash
# Imperative
kubectl create deployment nginx --image=nginx --replicas=3

# View deployments
kubectl get deployments

# View replica sets
kubectl get replicasets

# View pods
kubectl get pods

# Scale deployment
kubectl scale deployment nginx --replicas=5

# Update image
kubectl set image deployment/nginx nginx=nginx:1.25

# View rollout status
kubectl rollout status deployment/nginx

# View rollout history
kubectl rollout history deployment/nginx

# Rollback to previous version
kubectl rollout undo deployment/nginx

# Rollback to specific revision
kubectl rollout undo deployment/nginx --to-revision=2
```

### **Deployment YAML:**
```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "500m"
            memory: "512Mi"
          requests:
            cpu: "250m"
            memory: "256Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

```bash
# Apply deployment
kubectl apply -f deployment.yaml

# Watch pods being created
kubectl get pods -w

# Describe deployment
kubectl describe deployment nginx

# Delete deployment
kubectl delete -f deployment.yaml
```

---

## 🌐 Services

**Service** = Stable endpoint to access pods, provides load balancing.

### **Service Types:**
```
ClusterIP (default)
└── Internal cluster IP, only accessible within cluster

NodePort
└── Exposes service on each node's IP at a static port

LoadBalancer
└── Creates external load balancer (cloud providers)

ExternalName
└── Maps service to external DNS name
```

### **ClusterIP Service:**
```yaml
# service-clusterip.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f service-clusterip.yaml

# Access from within cluster
kubectl run test --image=curlimages/curl -it --rm -- curl nginx-service
```

### **NodePort Service:**
```yaml
# service-nodeport.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080  # Optional: 30000-32767
```

```bash
kubectl apply -f service-nodeport.yaml

# Get service
kubectl get svc nginx-nodeport

# Access service (Minikube)
minikube service nginx-nodeport

# Get URL
minikube service nginx-nodeport --url

# Access via node IP
curl http://<NODE-IP>:30080
```

### **LoadBalancer Service:**
```yaml
# service-loadbalancer.yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

```bash
kubectl apply -f service-loadbalancer.yaml

# Get external IP (cloud providers)
kubectl get svc nginx-lb

# Minikube tunnel (for LoadBalancer locally)
minikube tunnel
```

---

## 🎯 Complete Application Example

### **Deploy Full Stack Application:**
```yaml
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
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
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: DB_HOST
          value: "postgres-service"
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        resources:
          limits:
            cpu: "1"
            memory: "1Gi"
          requests:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
---
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: "supersecret123"
```

```bash
# Deploy everything
kubectl apply -f app-deployment.yaml

# Watch deployment
kubectl get all

# Check pods
kubectl get pods

# Check services
kubectl get svc

# View logs
kubectl logs -f deployment/myapp

# Access application
minikube service myapp-service
```

---

## 🔧 Useful kubectl Commands

### **Namespace Management:**
```bash
# List namespaces
kubectl get namespaces

# Create namespace
kubectl create namespace dev

# Set default namespace
kubectl config set-context --current --namespace=dev

# View resources in namespace
kubectl get all -n dev

# Delete namespace
kubectl delete namespace dev
```

### **Debugging:**
```bash
# Describe resource
kubectl describe pod pod-name

# View logs
kubectl logs pod-name
kubectl logs -f pod-name
kubectl logs --previous pod-name  # Previous instance

# Execute commands
kubectl exec pod-name -- ls /
kubectl exec -it pod-name -- bash

# Port forwarding
kubectl port-forward pod/nginx 8080:80
kubectl port-forward service/nginx-service 8080:80

# Copy files
kubectl cp pod-name:/path/to/file ./local-file
kubectl cp ./local-file pod-name:/path/to/file

# View events
kubectl get events
kubectl get events --sort-by='.metadata.creationTimestamp'
```

### **Resource Management:**
```bash
# Get all resources
kubectl get all

# Get specific resource
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get nodes

# Output formats
kubectl get pods -o wide
kubectl get pods -o yaml
kubectl get pods -o json

# Watch resources
kubectl get pods -w

# Delete resources
kubectl delete pod pod-name
kubectl delete deployment deployment-name
kubectl delete -f file.yaml
kubectl delete all --all  # Delete everything in namespace
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's a Pod in Kubernetes?**
   <details>
   <summary>Answer</summary>
   Smallest deployable unit, contains one or more containers
   </details>

2. **What's the difference between Deployment and Pod?**
   <details>
   <summary>Answer</summary>
   Deployment manages replicas and updates; Pod is a single instance
   </details>

3. **What are the main Service types?**
   <details>
   <summary>Answer</summary>
   ClusterIP (internal), NodePort (node access), LoadBalancer (external)
   </details>

4. **How do you scale a deployment?**
   <details>
   <summary>Answer</summary>
   kubectl scale deployment name --replicas=5
   </details>

5. **What's the kubectl command to view logs?**
   <details>
   <summary>Answer</summary>
   kubectl logs pod-name
   </details>

---

## 📝 Hands-on Exercise

**Deploy complete application:**

```bash
# 1. Start Minikube
minikube start

# 2. Create deployment
kubectl create deployment nginx --image=nginx:alpine --replicas=3

# 3. Expose deployment
kubectl expose deployment nginx --type=NodePort --port=80

# 4. View resources
kubectl get all

# 5. Scale deployment
kubectl scale deployment nginx --replicas=5

# 6. Access service
minikube service nginx --url
curl $(minikube service nginx --url)

# 7. Update image
kubectl set image deployment/nginx nginx=nginx:1.25

# 8. Watch rollout
kubectl rollout status deployment/nginx

# 9. View rollout history
kubectl rollout history deployment/nginx

# 10. Rollback
kubectl rollout undo deployment/nginx

# 11. Cleanup
kubectl delete deployment nginx
kubectl delete service nginx
minikube stop
```

---

## 📝 Key Takeaways

✅ **Kubernetes is the industry standard orchestrator**  
✅ **Pods are the smallest deployable units**  
✅ **Deployments manage replicas and updates**  
✅ **Services provide stable networking**  
✅ **kubectl is the command-line interface**  
✅ **Declarative configuration with YAML**  
✅ **Self-healing and auto-scaling**  
✅ **Rolling updates with zero downtime**  
✅ **Minikube/Kind for local development**  
✅ **Steeper learning curve than Swarm**  

---

## 🚀 Next Steps

**Optimize performance:**

**Next lesson:** [20 - Container Performance](20-container-performance.md) - Optimization techniques

Welcome to the world of Kubernetes! ☸️
