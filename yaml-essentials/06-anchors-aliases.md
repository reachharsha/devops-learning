---
render_with_liquid: false
---
# 06 - Anchors and Aliases (DRY Principle)

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What anchors and aliases are
- How to reuse configuration
- Merge keys for combining configs
- Real-world use cases
- When (and when not) to use them

---

## 🔗 What Are Anchors and Aliases?

**Anchors** (`&`) - Mark a value for reuse  
**Aliases** (`*`) - Reference the anchored value  
**Merge** (`<<`) - Merge dictionary contents

**DRY Principle:** Don't Repeat Yourself

**Real-World Analogy:**
Think of anchors like bookmarks:
- **Anchor** = Place a bookmark
- **Alias** = Jump to that bookmark
- **Merge** = Copy pages from that bookmark

---

## ⚓ Basic Anchors and Aliases

### **Simple Reference:**
```yaml
# Define with anchor (&)
default_user: &default_user
  role: user
  active: true
  permissions:
    - read

# Reuse with alias (*)
john: *default_user
jane: *default_user
bob: *default_user
```

**Result (expanded):**
```yaml
default_user:
  role: user
  active: true
  permissions:
    - read

john:
  role: user
  active: true
  permissions:
    - read

jane:
  role: user
  active: true
  permissions:
    - read

bob:
  role: user
  active: true
  permissions:
    - read
```

---

## 🔄 Merge Keys (`<<`)

### **Override Specific Values:**
```yaml
# Base configuration
defaults: &defaults
  timeout: 30
  retries: 3
  debug: false

# Inherit and override
development:
  <<: *defaults  # Merge defaults
  debug: true    # Override debug

production:
  <<: *defaults  # Merge defaults
  timeout: 60    # Override timeout
  retries: 5     # Override retries

testing:
  <<: *defaults  # Use all defaults
```

**Result:**
```yaml
development:
  timeout: 30    # from defaults
  retries: 3     # from defaults
  debug: true    # overridden

production:
  timeout: 60    # overridden
  retries: 5     # overridden
  debug: false   # from defaults

testing:
  timeout: 30    # from defaults
  retries: 3     # from defaults
  debug: false   # from defaults
```

---

## 🎨 Real-World Examples

### **Docker Compose - Common Service Config:**
```yaml
version: "3.8"

# Common service configuration
x-common-service: &common-service
  restart: unless-stopped
  networks:
    - app-network
  logging:
    driver: json-file
    options:
      max-size: "10m"
      max-file: "3"

services:
  web:
    <<: *common-service
    image: nginx:alpine
    ports:
      - "80:80"
      
  api:
    <<: *common-service
    image: myapp/api:latest
    ports:
      - "8080:8080"
    environment:
      - DB_HOST=database
      
  worker:
    <<: *common-service
    image: myapp/worker:latest
    # No ports needed

networks:
  app-network:
```

### **Kubernetes - Common Labels:**
```yaml
# Common metadata
x-labels: &common-labels
  app: myapp
  environment: production
  managed-by: kubernetes

---
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    <<: *common-labels
    component: web
spec:
  selector:
    <<: *common-labels
    component: web
  ports:
  - port: 80

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    <<: *common-labels
    component: web
spec:
  selector:
    matchLabels:
      <<: *common-labels
      component: web
  template:
    metadata:
      labels:
        <<: *common-labels
        component: web
    spec:
      containers:
      - name: web
        image: nginx:1.21
```

### **CI/CD - Reusable Job Templates:**
```yaml
# GitHub Actions workflow

# Reusable step templates
.cache-dependencies: &cache-deps
  name: Cache dependencies
  uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${ { runner.os } }-node-${ { hashFiles('**/package-lock.json') } }

.setup-node: &setup-node
  name: Setup Node.js
  uses: actions/setup-node@v3
  with:
    node-version: 16
    cache: npm

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - *setup-node
      - *cache-deps
      - run: npm test
      
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - *setup-node
      - *cache-deps
      - run: npm run build
      
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - *setup-node
      - *cache-deps
      - run: npm run deploy
```

### **Application Config - Environment Variations:**
```yaml
# Base database configuration
database_defaults: &db-defaults
  driver: postgresql
  pool_size: 10
  timeout: 5000
  ssl: false
  retry_attempts: 3

environments:
  development:
    database:
      <<: *db-defaults
      host: localhost
      port: 5432
      name: myapp_dev
      ssl: false
      
  staging:
    database:
      <<: *db-defaults
      host: staging-db.example.com
      port: 5432
      name: myapp_staging
      pool_size: 20    # More connections
      ssl: true
      
  production:
    database:
      <<: *db-defaults
      host: prod-db.example.com
      port: 5432
      name: myapp_prod
      pool_size: 50    # Even more connections
      timeout: 10000   # Longer timeout
      ssl: true
```

---

## 🔗 Multiple Merge Sources

### **Combine Multiple Anchors:**
```yaml
# Define multiple base configs
logging_config: &logging
  log_level: info
  log_format: json

monitoring_config: &monitoring
  metrics_enabled: true
  tracing_enabled: true

# Merge both
service_defaults: &service-defaults
  <<: [*logging, *monitoring]  # Merge array
  restart_policy: always

# Use merged config
web_service:
  <<: *service-defaults
  port: 80
  
api_service:
  <<: *service-defaults
  port: 8080
```

---

## ⚠️ Important Limitations

### **1. Aliases Reference, Not Copy:**
```yaml
# Anchor
colors: &my-colors
  - red
  - blue

# Aliases reference same list
primary: *my-colors
secondary: *my-colors

# Modifying one affects both (in some tools)
```

### **2. Anchors Must Be Defined Before Use:**
```yaml
# ❌ WRONG - alias before anchor
user: *default
defaults: &default
  role: user

# ✅ CORRECT - anchor before alias
defaults: &default
  role: user
user: *default
```

### **3. Merge Only Works with Dictionaries:**
```yaml
# ❌ WRONG - can't merge lists
defaults: &defaults
  - item1
  - item2
something:
  <<: *defaults  # Error!

# ✅ CORRECT - merge dictionaries
defaults: &defaults
  key: value
something:
  <<: *defaults  # Works!
```

---

## 🎯 Quick Check: Do You Understand?

1. **What symbol creates an anchor?**
   <details>
   <summary>Answer</summary>
   & (ampersand)
   </details>

2. **What symbol references an alias?**
   <details>
   <summary>Answer</summary>
   * (asterisk)
   </details>

3. **What does `<<` do?**
   <details>
   <summary>Answer</summary>
   Merge key - merges dictionary contents
   </details>

4. **Can you use an alias before defining the anchor?**
   <details>
   <summary>Answer</summary>
   No! Anchor must be defined first
   </details>

---

## 📝 Hands-on Exercise

**Create reusable server configurations:**

Create `servers.yaml`:
```yaml
# Base server configuration
base_server: &base
  os: Ubuntu 22.04
  monitoring: enabled
  backup: enabled
  firewall: enabled
  ssh_port: 22

# Resource templates
small_resources: &small
  cpu: 2
  ram: 4GB
  disk: 50GB

medium_resources: &medium
  cpu: 4
  ram: 8GB
  disk: 100GB

large_resources: &large
  cpu: 8
  ram: 16GB
  disk: 200GB

# Actual servers
servers:
  web-01:
    <<: [*base, *medium]  # Merge base + medium resources
    name: web-01.example.com
    role: webserver
    applications:
      - nginx
      - certbot
      
  db-01:
    <<: [*base, *large]  # Merge base + large resources
    name: db-01.example.com
    role: database
    applications:
      - postgresql
      - pgbackup
    backup_frequency: hourly  # Override
    
  cache-01:
    <<: [*base, *small]  # Merge base + small resources
    name: cache-01.example.com
    role: cache
    applications:
      - redis
```

**Tasks:**
1. Validate the YAML
2. Add a new resource tier (xlarge)
3. Add a new server with custom overrides
4. Create a security_defaults anchor

---

## ⚠️ When NOT to Use Anchors

**Avoid anchors when:**

1. **Configuration is used once** - No reuse = no benefit
2. **Overriding most values** - Defeats the purpose
3. **Makes config harder to read** - Keep it simple
4. **Team unfamiliar with YAML** - Learning curve

```yaml
# ❌ BAD - Anchor used only once
defaults: &defaults
  timeout: 30
server:
  <<: *defaults
  # Only one use!

# ✅ BETTER - Just write it directly
server:
  timeout: 30
```

---

## 📝 Key Takeaways

✅ **Anchors (&) mark values for reuse**  
✅ **Aliases (*) reference anchored values**  
✅ **Merge (<<) combines dictionaries**  
✅ **Great for DRY configuration**  
✅ **Anchor must be defined before use**  
✅ **Works with dicts and lists**  
✅ **Use when you have genuine repetition**  

---

## 🔍 Anchor vs Include Files

**Anchors:**
- ✅ Same file reuse
- ✅ Simple and built-in
- ❌ Limited to one file

**Include Files:**
- ✅ Cross-file reuse
- ✅ Better organization
- ⚠️ Tool-dependent (not standard YAML)

```yaml
# Some tools support includes
# Docker Compose
services:
  web:
    extends:
      file: common-services.yml
      service: webapp
```

---

## 🚀 Next Steps

**Next lesson:** [07 - Multi-line Strings](07-multiline-strings.md) - Handle long text elegantly

---

## 💡 Pro Tips

**Naming Anchors:**
```yaml
# ✅ GOOD - descriptive names
common_service_config: &common-service
db_production_settings: &db-prod

# ❌ BAD - cryptic names
cfg: &c
thing: &t
```

**Extension Fields (Docker Compose):**
```yaml
# x- prefix prevents warnings
x-common: &common
  restart: unless-stopped

services:
  web:
    <<: *common
    image: nginx
```

**Debugging:**
```bash
# Expand all anchors/aliases
yq eval '.' file.yaml

# Some tools have --no-anchors flag
```

Reuse wisely, keep it DRY! 🔗
