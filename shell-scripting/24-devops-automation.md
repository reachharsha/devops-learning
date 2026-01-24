---
render_with_liquid: false
---
# 24 - DevOps Automation

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- CI/CD pipeline automation
- Docker container management
- Kubernetes deployment scripts
- Git automation and hooks
- Application deployment
- Environment management
- Infrastructure as Code basics

---

## 🚀 CI/CD Pipeline Scripts

### **Build and Test Automation:**

```bash
#!/bin/bash

set -euo pipefail

PROJECT_DIR="/var/www/app"
BUILD_DIR="$PROJECT_DIR/build"
LOG_FILE="build.log"

# Navigate to project
cd "$PROJECT_DIR"

# Clean previous build
clean() {
    echo "Cleaning previous build..."
    rm -rf "$BUILD_DIR"
    mkdir -p "$BUILD_DIR"
}

# Install dependencies
install_dependencies() {
    echo "Installing dependencies..."
    
    if [ -f "package.json" ]; then
        npm ci
    elif [ -f "requirements.txt" ]; then
        pip install -r requirements.txt
    elif [ -f "Gemfile" ]; then
        bundle install
    fi
}

# Run linter
lint() {
    echo "Running linter..."
    
    if [ -f "package.json" ]; then
        npm run lint
    elif [ -f "pyproject.toml" ]; then
        pylint src/
    fi
}

# Run tests
test() {
    echo "Running tests..."
    
    if [ -f "package.json" ]; then
        npm test
    elif [ -f "pytest.ini" ]; then
        pytest
    fi
}

# Build application
build() {
    echo "Building application..."
    
    if [ -f "package.json" ]; then
        npm run build
    elif [ -f "setup.py" ]; then
        python setup.py build
    fi
}

# Main CI pipeline
main() {
    echo "=== CI Pipeline Started ==="
    
    clean
    install_dependencies
    lint
    test
    build
    
    echo "=== CI Pipeline Completed Successfully ==="
}

# Run pipeline
main 2>&1 | tee "$LOG_FILE"
```

### **Deployment Script:**

```bash
#!/bin/bash

set -euo pipefail

APP_NAME="myapp"
ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}

DEPLOY_USER="deploy"
DEPLOY_HOST="server.example.com"
DEPLOY_PATH="/var/www/$APP_NAME"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

# Pre-deployment checks
pre_deploy_checks() {
    log "Running pre-deployment checks..."
    
    # Check SSH connection
    if ! ssh "$DEPLOY_USER@$DEPLOY_HOST" "echo 'SSH OK'"; then
        log "ERROR: Cannot connect to server"
        exit 1
    fi
    
    # Check disk space
    local available=$(ssh "$DEPLOY_USER@$DEPLOY_HOST" "df $DEPLOY_PATH | tail -1 | awk '{print \$4}'")
    if [ "$available" -lt 1000000 ]; then
        log "ERROR: Insufficient disk space"
        exit 1
    fi
    
    log "Pre-deployment checks passed"
}

# Backup current version
backup_current() {
    log "Backing up current version..."
    
    ssh "$DEPLOY_USER@$DEPLOY_HOST" << EOF
        if [ -d "$DEPLOY_PATH" ]; then
            tar -czf "$DEPLOY_PATH.backup.$(date +%Y%m%d%H%M%S).tar.gz" "$DEPLOY_PATH"
        fi
EOF
    
    log "Backup completed"
}

# Deploy application
deploy() {
    log "Deploying $APP_NAME version $VERSION to $ENVIRONMENT..."
    
    # Upload files
    rsync -avz --delete \
        --exclude='node_modules' \
        --exclude='.git' \
        --exclude='*.log' \
        ./ "$DEPLOY_USER@$DEPLOY_HOST:$DEPLOY_PATH/"
    
    # Install dependencies and restart
    ssh "$DEPLOY_USER@$DEPLOY_HOST" << EOF
        cd "$DEPLOY_PATH"
        
        # Install dependencies
        if [ -f "package.json" ]; then
            npm ci --production
        fi
        
        # Run migrations
        if [ -f "migrate.sh" ]; then
            ./migrate.sh
        fi
        
        # Restart application
        pm2 restart $APP_NAME || pm2 start ecosystem.config.js
EOF
    
    log "Deployment completed"
}

# Health check
health_check() {
    log "Running health check..."
    
    local max_attempts=30
    local attempt=1
    
    while [ $attempt -le $max_attempts ]; do
        if curl -sf "http://$DEPLOY_HOST/health" > /dev/null; then
            log "Health check passed"
            return 0
        fi
        
        log "Waiting for application to start... (attempt $attempt/$max_attempts)"
        sleep 2
        ((attempt++))
    done
    
    log "ERROR: Health check failed"
    return 1
}

# Rollback
rollback() {
    log "Rolling back deployment..."
    
    ssh "$DEPLOY_USER@$DEPLOY_HOST" << EOF
        # Find latest backup
        BACKUP=\$(ls -t "$DEPLOY_PATH.backup."*.tar.gz | head -1)
        
        if [ -z "\$BACKUP" ]; then
            echo "ERROR: No backup found"
            exit 1
        fi
        
        # Restore backup
        rm -rf "$DEPLOY_PATH"
        tar -xzf "\$BACKUP" -C /
        
        # Restart application
        pm2 restart $APP_NAME
EOF
    
    log "Rollback completed"
}

# Main deployment
main() {
    log "=== Deployment Started ==="
    log "Application: $APP_NAME"
    log "Environment: $ENVIRONMENT"
    log "Version: $VERSION"
    
    pre_deploy_checks
    backup_current
    deploy
    
    if health_check; then
        log "=== Deployment Successful ==="
    else
        log "=== Deployment Failed, Rolling Back ==="
        rollback
        exit 1
    fi
}

main
```

---

## 🐳 Docker Automation

### **Docker Management Script:**

```bash
#!/bin/bash

IMAGE_NAME="myapp"
CONTAINER_NAME="myapp-container"
VERSION=${1:-latest}

# Build Docker image
build_image() {
    echo "Building Docker image: $IMAGE_NAME:$VERSION"
    
    docker build \
        -t "$IMAGE_NAME:$VERSION" \
        -t "$IMAGE_NAME:latest" \
        --build-arg VERSION="$VERSION" \
        .
    
    echo "Image built successfully"
}

# Push to registry
push_image() {
    local registry=${1:-docker.io}
    
    echo "Pushing image to $registry..."
    
    docker tag "$IMAGE_NAME:$VERSION" "$registry/$IMAGE_NAME:$VERSION"
    docker push "$registry/$IMAGE_NAME:$VERSION"
    
    echo "Image pushed successfully"
}

# Run container
run_container() {
    echo "Starting container: $CONTAINER_NAME"
    
    # Stop existing container
    docker stop "$CONTAINER_NAME" 2>/dev/null || true
    docker rm "$CONTAINER_NAME" 2>/dev/null || true
    
    # Run new container
    docker run -d \
        --name "$CONTAINER_NAME" \
        --restart unless-stopped \
        -p 8080:8080 \
        -e NODE_ENV=production \
        -v /data:/app/data \
        "$IMAGE_NAME:$VERSION"
    
    echo "Container started"
}

# View logs
view_logs() {
    docker logs -f "$CONTAINER_NAME"
}

# Execute command in container
exec_command() {
    local command=$1
    docker exec -it "$CONTAINER_NAME" $command
}

# Container health check
health_check() {
    docker inspect "$CONTAINER_NAME" --format='{{.State.Health.Status}}'
}

# Clean up old images
cleanup() {
    echo "Cleaning up old images..."
    
    # Remove dangling images
    docker image prune -f
    
    # Remove old versions (keep last 3)
    docker images "$IMAGE_NAME" --format "{{.Tag}}" | \
        tail -n +4 | \
        xargs -I {} docker rmi "$IMAGE_NAME:{}"
}

# Docker Compose wrapper
compose_up() {
    docker-compose up -d
}

compose_down() {
    docker-compose down
}

compose_logs() {
    docker-compose logs -f
}
```

### **Multi-Container Deployment:**

```bash
#!/bin/bash

set -euo pipefail

PROJECT_NAME="myproject"
COMPOSE_FILE="docker-compose.yml"

# Start all services
start_services() {
    echo "Starting services..."
    docker-compose -f "$COMPOSE_FILE" up -d
    
    echo "Waiting for services to be ready..."
    sleep 10
}

# Stop all services
stop_services() {
    echo "Stopping services..."
    docker-compose -f "$COMPOSE_FILE" down
}

# Restart specific service
restart_service() {
    local service=$1
    echo "Restarting $service..."
    docker-compose -f "$COMPOSE_FILE" restart "$service"
}

# Scale service
scale_service() {
    local service=$1
    local count=$2
    echo "Scaling $service to $count instances..."
    docker-compose -f "$COMPOSE_FILE" up -d --scale "$service=$count"
}

# View service logs
service_logs() {
    local service=$1
    docker-compose -f "$COMPOSE_FILE" logs -f "$service"
}

# Health check all services
health_check_all() {
    docker-compose -f "$COMPOSE_FILE" ps
}

# Update and redeploy
update_deploy() {
    echo "Pulling latest images..."
    docker-compose -f "$COMPOSE_FILE" pull
    
    echo "Recreating containers..."
    docker-compose -f "$COMPOSE_FILE" up -d --force-recreate
}
```

---

## ☸️ Kubernetes Scripts

### **Kubernetes Deployment:**

```bash
#!/bin/bash

set -euo pipefail

NAMESPACE="production"
APP_NAME="myapp"
IMAGE="myregistry/myapp:latest"

# Apply configuration
deploy() {
    echo "Deploying to Kubernetes..."
    
    # Create namespace if not exists
    kubectl create namespace "$NAMESPACE" --dry-run=client -o yaml | kubectl apply -f -
    
    # Apply deployment
    kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: $APP_NAME
  namespace: $NAMESPACE
spec:
  replicas: 3
  selector:
    matchLabels:
      app: $APP_NAME
  template:
    metadata:
      labels:
        app: $APP_NAME
    spec:
      containers:
      - name: $APP_NAME
        image: $IMAGE
        ports:
        - containerPort: 8080
        env:
        - name: NODE_ENV
          value: "production"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: $APP_NAME
  namespace: $NAMESPACE
spec:
  selector:
    app: $APP_NAME
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: LoadBalancer
EOF
    
    echo "Deployment applied"
}

# Rolling update
rolling_update() {
    local new_image=$1
    
    echo "Updating image to $new_image..."
    kubectl set image deployment/$APP_NAME \
        $APP_NAME=$new_image \
        -n "$NAMESPACE"
    
    kubectl rollout status deployment/$APP_NAME -n "$NAMESPACE"
}

# Rollback
rollback() {
    echo "Rolling back deployment..."
    kubectl rollout undo deployment/$APP_NAME -n "$NAMESPACE"
    kubectl rollout status deployment/$APP_NAME -n "$NAMESPACE"
}

# Scale deployment
scale() {
    local replicas=$1
    echo "Scaling to $replicas replicas..."
    kubectl scale deployment/$APP_NAME --replicas=$replicas -n "$NAMESPACE"
}

# Get pod logs
get_logs() {
    kubectl logs -f deployment/$APP_NAME -n "$NAMESPACE"
}

# Get pod status
get_status() {
    kubectl get pods -n "$NAMESPACE" -l app=$APP_NAME
}

# Execute command in pod
exec_pod() {
    local pod=$(kubectl get pods -n "$NAMESPACE" -l app=$APP_NAME -o jsonpath='{.items[0].metadata.name}')
    kubectl exec -it "$pod" -n "$NAMESPACE" -- /bin/bash
}

# Delete deployment
delete() {
    echo "Deleting deployment..."
    kubectl delete deployment/$APP_NAME -n "$NAMESPACE"
    kubectl delete service/$APP_NAME -n "$NAMESPACE"
}
```

---

## 🔧 Git Automation

### **Git Hooks:**

```bash
#!/bin/bash
# Save as .git/hooks/pre-commit

set -e

echo "Running pre-commit checks..."

# Check for conflicts
if git diff --cached --name-only | xargs grep -l "<<<<<<< HEAD" 2>/dev/null; then
    echo "Error: Merge conflicts detected"
    exit 1
fi

# Run linter
if [ -f "package.json" ]; then
    npm run lint
fi

# Run tests
if [ -f "package.json" ]; then
    npm test
fi

echo "Pre-commit checks passed"
```

### **Git Automation Script:**

```bash
#!/bin/bash

# Auto commit and push
auto_commit() {
    local message=${1:-"Auto commit $(date)"}
    
    git add .
    git commit -m "$message"
    git push
}

# Create and push branch
create_branch() {
    local branch_name=$1
    
    git checkout -b "$branch_name"
    git push -u origin "$branch_name"
}

# Merge and delete branch
merge_branch() {
    local branch=$1
    local target=${2:-main}
    
    git checkout "$target"
    git merge "$branch"
    git push
    git branch -d "$branch"
    git push origin --delete "$branch"
}

# Tag release
tag_release() {
    local version=$1
    local message=${2:-"Release $version"}
    
    git tag -a "v$version" -m "$message"
    git push origin "v$version"
}

# Clean merged branches
clean_branches() {
    git branch --merged | grep -v "\*" | grep -v "main" | xargs -r git branch -d
}

# Generate changelog
generate_changelog() {
    local from_tag=${1:-$(git describe --tags --abbrev=0)}
    local to_tag=${2:-HEAD}
    
    git log "$from_tag..$to_tag" --pretty=format:"- %s (%an)" --reverse
}
```

---

## 🎯 Environment Management

### **Environment Switcher:**

```bash
#!/bin/bash

ENVIRONMENTS=("development" "staging" "production")
CURRENT_ENV_FILE=".current_env"

# Switch environment
switch_env() {
    local env=$1
    
    # Validate environment
    if [[ ! " ${ENVIRONMENTS[@]} " =~ " ${env} " ]]; then
        echo "Error: Invalid environment. Choose from: ${ENVIRONMENTS[*]}"
        return 1
    fi
    
    # Load environment configuration
    if [ -f ".env.$env" ]; then
        cp ".env.$env" .env
        echo "$env" > "$CURRENT_ENV_FILE"
        echo "Switched to $env environment"
    else
        echo "Error: Configuration file .env.$env not found"
        return 1
    fi
}

# Get current environment
get_current_env() {
    if [ -f "$CURRENT_ENV_FILE" ]; then
        cat "$CURRENT_ENV_FILE"
    else
        echo "unknown"
    fi
}

# Load environment variables
load_env() {
    if [ -f .env ]; then
        export $(cat .env | grep -v '^#' | xargs)
    fi
}

# Create new environment
create_env() {
    local env=$1
    local template=${2:-.env.example}
    
    if [ -f ".env.$env" ]; then
        echo "Error: Environment $env already exists"
        return 1
    fi
    
    cp "$template" ".env.$env"
    echo "Environment $env created from $template"
}
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you build a Docker image?**
   <details>
   <summary>Answer</summary>
   docker build -t image_name:tag .
   </details>

2. **How do you deploy to Kubernetes?**
   <details>
   <summary>Answer</summary>
   kubectl apply -f deployment.yaml
   </details>

3. **How do you create a Git tag?**
   <details>
   <summary>Answer</summary>
   git tag -a v1.0 -m "Version 1.0"
   </details>

4. **How do you run a health check after deployment?**
   <details>
   <summary>Answer</summary>
   curl -f http://server/health || rollback
   </details>

5. **How do you rollback a Kubernetes deployment?**
   <details>
   <summary>Answer</summary>
   kubectl rollout undo deployment/name
   </details>

---

## 📝 Key Takeaways

✅ **Automate build, test, deploy pipeline**  
✅ **Always run health checks after deployment**  
✅ **Backup before deployment, rollback on failure**  
✅ **Use Docker for consistent environments**  
✅ **Kubernetes for orchestration and scaling**  
✅ **Git hooks for automated checks**  
✅ **Environment-specific configurations**  

---

## 🚀 Next Steps

You're now a DevOps automation expert!

**Next lesson:** [25 - Security Best Practices](25-security-best-practices.md) - Secure scripting and hardening

---

## 💡 Pro Tips

**Zero-downtime deployment:**
```bash
# Deploy new version
deploy_new_version
# Health check
health_check || rollback
# Switch traffic
switch_traffic
```

**Always use version tags:**
```bash
docker build -t app:$(git rev-parse --short HEAD)
```

**Automate everything:**
```bash
# One command deployment
./deploy.sh production v1.2.3
```

DevOps = Development + Operations automation! 🚀
