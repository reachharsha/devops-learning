---
render_with_liquid: false
---
# 07 - Multi-line Strings

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Different multi-line string styles
- Literal (`|`) vs folded (`>`) blocks
- Block chomping indicators
- When to use each style
- Common use cases in DevOps

---

## 📜 Why Multi-line Strings?

**Use Cases:**
- Scripts and commands
- SQL queries
- Configuration file contents
- Documentation
- Kubernetes manifests
- Docker files

**Problem with single-line:**
```yaml
# ❌ Hard to read
script: "apt-get update && apt-get install -y nginx && systemctl start nginx && systemctl enable nginx"
```

---

## 📝 Literal Block (`|`) - Preserves Newlines

### **Syntax:**
```yaml
script: |
  Line 1
  Line 2
  Line 3
```

**Output:** `"Line 1\nLine 2\nLine 3\n"`

### **Preserves Line Breaks:**
```yaml
description: |
  This is line 1
  This is line 2
  This is line 3
```

**Result:**
```
This is line 1
This is line 2
This is line 3

```

### **Common Usage - Shell Scripts:**
```yaml
script: |
  #!/bin/bash
  apt-get update
  apt-get install -y nginx
  systemctl start nginx
  systemctl enable nginx
  echo "Installation complete"
```

---

## 📄 Folded Block (`>`) - Folds Newlines

### **Syntax:**
```yaml
description: >
  This is a long sentence that
  spans multiple lines but will
  be folded into a single line
  with spaces between them.
```

**Output:** `"This is a long sentence that spans multiple lines but will be folded into a single line with spaces between them.\n"`

### **Use Case - Long Text:**
```yaml
help_text: >
  This command will install and configure the web server.
  It performs the following actions: updates package lists,
  installs nginx, starts the service, and enables it at boot.
  For more information, see the documentation at example.com/docs.
```

**Result (single line):**
```
This command will install and configure the web server. It performs the following actions: updates package lists, installs nginx, starts the service, and enables it at boot. For more information, see the documentation at example.com/docs.

```

---

## 🔍 Literal vs Folded Comparison

```yaml
# Literal (|) - Preserves newlines
poem: |
  Roses are red
  Violets are blue
  YAML is great
  And so are you

# Folded (>) - Joins into one line
paragraph: >
  This is a very long paragraph that
  we want to wrap across multiple lines
  for readability in our YAML file
  but should be one line in output.
```

**Output:**
```yaml
# poem (literal)
Roses are red
Violets are blue
YAML is great
And so are you

# paragraph (folded)
This is a very long paragraph that we want to wrap across multiple lines for readability in our YAML file but should be one line in output.

```

---

## ✂️ Block Chomping Indicators

Controls trailing newlines at the end of the block.

### **Default (clip):**
```yaml
default: |
  Line 1
  Line 2
```
**Output:** `"Line 1\nLine 2\n"` (one newline at end)

### **Strip (`|-` or `>-`)** - Remove trailing newlines:
```yaml
strip: |-
  Line 1
  Line 2
```
**Output:** `"Line 1\nLine 2"` (no trailing newline)

### **Keep (`|+` or `>+`)** - Keep all trailing newlines:
```yaml
keep: |+
  Line 1
  Line 2


```
**Output:** `"Line 1\nLine 2\n\n\n"` (all newlines kept)

---

## 🎨 Real-World Examples

### **Kubernetes ConfigMap - Script:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: scripts
data:
  backup.sh: |
    #!/bin/bash
    set -e
    
    DATE=$(date +%Y%m%d_%H%M%S)
    BACKUP_FILE="/backups/db_$DATE.sql"
    
    echo "Starting backup at $(date)"
    pg_dump -U postgres myapp > "$BACKUP_FILE"
    gzip "$BACKUP_FILE"
    echo "Backup complete: $BACKUP_FILE.gz"
    
    # Cleanup old backups (keep last 7 days)
    find /backups -name "db_*.sql.gz" -mtime +7 -delete
    
  healthcheck.sh: |
    #!/bin/bash
    curl -f http://localhost:8080/health || exit 1
```

### **Docker Compose - Commands:**
```yaml
version: "3.8"

services:
  web:
    image: nginx
    command: |
      bash -c "
      echo 'Starting nginx...'
      nginx -g 'daemon off;'
      "
      
  worker:
    image: myapp/worker
    entrypoint: |
      /bin/sh -c "
      python manage.py migrate &&
      python manage.py collectstatic --noinput &&
      celery -A myapp worker --loglevel=info
      "
```

### **GitHub Actions - Multi-line Run:**
```yaml
name: Deploy

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          
          # Build application
          npm run build
          
          # Run tests
          npm test
          
          # Deploy
          rsync -avz dist/ user@server:/var/www/app/
          
          # Restart service
          ssh user@server 'systemctl restart myapp'
          
          echo "Deployment complete!"
```

### **SQL Queries:**
```yaml
queries:
  get_active_users: |
    SELECT u.id, u.name, u.email, u.created_at
    FROM users u
    WHERE u.active = true
      AND u.verified = true
      AND u.created_at >= NOW() - INTERVAL '30 days'
    ORDER BY u.created_at DESC
    LIMIT 100;
    
  user_statistics: |
    SELECT
      DATE(created_at) as date,
      COUNT(*) as total_users,
      COUNT(CASE WHEN verified = true THEN 1 END) as verified_users,
      COUNT(CASE WHEN active = true THEN 1 END) as active_users
    FROM users
    GROUP BY DATE(created_at)
    ORDER BY date DESC;
```

### **Configuration Files:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    user nginx;
    worker_processes auto;
    error_log /var/log/nginx/error.log;
    
    events {
        worker_connections 1024;
    }
    
    http {
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        
        log_format main '$remote_addr - $remote_user [$time_local] '
                        '"$request" $status $body_bytes_sent '
                        '"$http_referer" "$http_user_agent"';
        
        access_log /var/log/nginx/access.log main;
        
        server {
            listen 80;
            server_name example.com;
            
            location / {
                proxy_pass http://backend:8080;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
            }
        }
    }
    
  mime.types: |
    types {
        text/html html htm;
        text/css css;
        application/javascript js;
        application/json json;
        image/jpeg jpg jpeg;
        image/png png;
    }
```

---

## 📊 Choosing the Right Style

| Use Case | Style | Example |
|----------|-------|---------|
| Scripts | `\|` Literal | Shell scripts, Python |
| Commands | `\|` Literal | Docker commands |
| SQL | `\|` Literal | Database queries |
| Config files | `\|` Literal | nginx.conf, etc. |
| Long descriptions | `>` Folded | Help text, docs |
| Paragraphs | `>` Folded | README content |
| One-line commands | Plain | Simple strings |

---

## 🎯 Quick Check: Do You Understand?

1. **What does `|` do?**
   <details>
   <summary>Answer</summary>
   Literal block - preserves newlines
   </details>

2. **What does `>` do?**
   <details>
   <summary>Answer</summary>
   Folded block - joins lines into one with spaces
   </details>

3. **What does `|-` do?**
   <details>
   <summary>Answer</summary>
   Literal block with trailing newlines stripped
   </details>

4. **When would you use `>`?**
   <details>
   <summary>Answer</summary>
   Long descriptions or paragraphs that should be one line
   </details>

---

## 📝 Hands-on Exercise

**Create a deployment script configuration:**

Create `deploy-config.yaml`:
```yaml
deployment:
  name: Production Deploy
  
  description: >
    This deployment configuration handles the complete production
    deployment process including building, testing, and deploying
    the application to the production Kubernetes cluster. It includes
    health checks and rollback capabilities.
  
  pre_deploy_script: |
    #!/bin/bash
    set -euo pipefail
    
    echo "Running pre-deployment checks..."
    
    # Check if cluster is healthy
    kubectl cluster-info
    
    # Verify Docker registry access
    docker login -u $DOCKER_USER -p $DOCKER_PASS
    
    # Run tests
    npm test
    
    echo "Pre-deployment checks passed!"
  
  deploy_script: |
    #!/bin/bash
    set -euo pipefail
    
    IMAGE_TAG=$(git rev-parse --short HEAD)
    REGISTRY="ghcr.io/myorg/myapp"
    
    echo "Building image: $REGISTRY:$IMAGE_TAG"
    docker build -t $REGISTRY:$IMAGE_TAG .
    docker tag $REGISTRY:$IMAGE_TAG $REGISTRY:latest
    
    echo "Pushing images..."
    docker push $REGISTRY:$IMAGE_TAG
    docker push $REGISTRY:latest
    
    echo "Deploying to Kubernetes..."
    kubectl set image deployment/myapp myapp=$REGISTRY:$IMAGE_TAG
    kubectl rollout status deployment/myapp
    
    echo "Deployment complete!"
  
  rollback_script: |
    #!/bin/bash
    kubectl rollout undo deployment/myapp
    kubectl rollout status deployment/myapp
    echo "Rollback complete"
  
  healthcheck_command: |-
    curl -f http://myapp.example.com/health || exit 1
  
  success_message: >
    Deployment completed successfully! The new version is now
    serving traffic. Monitor the application dashboard for any issues.
```

**Tasks:**
1. Validate the YAML
2. Add a `post_deploy_script` with notification
3. Try changing `description` to use `|` - see the difference
4. Add a SQL migration script

---

## ⚠️ Common Mistakes

### **1. Wrong Indentation:**
```yaml
# ❌ WRONG - content not indented
script: |
Line 1
Line 2

# ✅ CORRECT - content indented
script: |
  Line 1
  Line 2
```

### **2. Missing Space After Indicator:**
```yaml
# ❌ WRONG - no space
script:|
  content

# ✅ CORRECT - space after |
script: |
  content
```

### **3. Using Wrong Style:**
```yaml
# ❌ BAD - folded for script (joins lines!)
script: >
  apt-get update
  apt-get install nginx

# Result: "apt-get update apt-get install nginx"

# ✅ CORRECT - literal for script
script: |
  apt-get update
  apt-get install nginx
```

---

## 📝 Key Takeaways

✅ **Literal `|` preserves newlines (for scripts)**  
✅ **Folded `>` joins lines (for paragraphs)**  
✅ **`|-` strips trailing newlines**  
✅ **`|+` keeps all trailing newlines**  
✅ **Content must be indented**  
✅ **Use literal for code/scripts**  
✅ **Use folded for long text descriptions**  

---

## 🔍 Quick Reference

```yaml
# Literal - Preserves everything
script: |
  line 1
  line 2

# Folded - Joins into one line
text: >
  long text that
  gets folded

# Strip trailing newlines
no_trail: |-
  content

# Keep trailing newlines
keep_trail: |+
  content


```

---

## 🚀 Next Steps

**Next lesson:** [08 - Docker Compose Deep Dive](08-docker-compose.md) - Real-world YAML in action

---

## 💡 Pro Tips

**Syntax Highlighting:**
```yaml
# Many editors support syntax highlighting in multi-line strings
script: |
  #!/bin/bash  # Detected as shell script
  echo "Hello"
```

**Escaping:**
```yaml
# No escaping needed in literal blocks
content: |
  Special chars: $ { } [ ] : " ' \
  All work without escaping!
```

**Empty Lines:**
```yaml
script: |
  Line 1
  
  Line 3  # Empty line preserved
```

Master multi-line strings for powerful configs! 📜
