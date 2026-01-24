---
render_with_liquid: false
---
# 05 - Dictionaries and Maps

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- How dictionaries/maps work in YAML
- Nested dictionaries
- Combining dicts with lists
- Flow style dictionaries
- Real-world dictionary patterns

---

## 📚 What Are Dictionaries?

**Dictionaries** (also called maps, hashes, or objects) store **key-value pairs**.

**Other Names:**
- **Maps** - Key-value mapping
- **Hashes** - Hash table
- **Objects** - In JSON/JavaScript
- **Associative Arrays** - In PHP

**Real-World Analogy:**
Think of a dictionary like a phone book:
- **Key** = Person's name
- **Value** = Phone number

---

## 🔑 Basic Dictionary Syntax

### **Simple Key-Value Pairs:**
```yaml
person:
  name: John Doe
  age: 30
  city: New York
  active: true
```

**Structure:**
```
person (dictionary/map)
├── name: John Doe
├── age: 30
├── city: New York
└── active: true
```

### **Root Level Dictionary:**
```yaml
# The entire file is a dictionary
name: MyApp
version: 1.0.0
port: 8080
debug: false
```

---

## 🎯 Nested Dictionaries

### **Multiple Levels:**
```yaml
person:
  name: John Doe
  age: 30
  contact:
    email: john@example.com
    phone: 555-1234
  address:
    street: 123 Main St
    city: Boston
    zip: 02101
```

**Hierarchy:**
```
person
├── name: John Doe
├── age: 30
├── contact
│   ├── email: john@example.com
│   └── phone: 555-1234
└── address
    ├── street: 123 Main St
    ├── city: Boston
    └── zip: 02101
```

### **Deep Nesting:**
```yaml
company:
  departments:
    engineering:
      backend:
        team_lead: Alice
        members: 5
        stack:
          language: Python
          framework: Django
          database: PostgreSQL
      frontend:
        team_lead: Bob
        members: 3
        stack:
          language: JavaScript
          framework: React
          build_tool: Webpack
```

---

## 🔄 Flow Style Dictionaries

### **Inline Format:**
```yaml
# Curly braces, comma separated
person: {name: John, age: 30, city: Boston}

# Multiple items
config: {debug: true, port: 8080, host: localhost}
```

### **When to Use:**
```yaml
# ✅ GOOD - short, simple
metadata: {version: 1.0, author: John}

# ❌ BAD - hard to read
person: {name: John, age: 30, email: john@example.com, phone: 555-1234, address: {street: 123 Main St, city: Boston}}

# ✅ BETTER - block style for complex data
person:
  name: John
  age: 30
  email: john@example.com
  phone: 555-1234
  address:
    street: 123 Main St
    city: Boston
```

---

## 📋 Combining Dictionaries and Lists

### **List of Dictionaries (Very Common!):**
```yaml
users:
  - name: John
    role: admin
    email: john@example.com
    
  - name: Jane
    role: user
    email: jane@example.com
    
  - name: Bob
    role: moderator
    email: bob@example.com
```

### **Dictionary of Lists:**
```yaml
teams:
  backend:
    - Alice
    - Bob
    - Carol
  frontend:
    - Dave
    - Eve
  devops:
    - Frank
    - Grace
```

### **Complex Structures:**
```yaml
projects:
  web_app:
    name: E-commerce Platform
    status: active
    team:
      - name: Alice
        role: Lead
      - name: Bob
        role: Developer
    technologies:
      - Python
      - React
      - PostgreSQL
      
  mobile_app:
    name: Shopping App
    status: planning
    team:
      - name: Carol
        role: Lead
      - name: Dave
        role: Developer
    technologies:
      - Flutter
      - Firebase
```

---

## 🎨 Real-World Examples

### **Application Configuration:**
```yaml
app:
  name: MyApp
  version: 1.2.0
  
server:
  host: 0.0.0.0
  port: 8080
  workers: 4
  timeout: 30
  
database:
  type: postgresql
  host: localhost
  port: 5432
  name: myapp_db
  credentials:
    username: admin
    password_env: DB_PASSWORD
  pool:
    min_size: 5
    max_size: 20
    
cache:
  enabled: true
  type: redis
  host: localhost
  port: 6379
  ttl: 3600
  
logging:
  level: info
  format: json
  outputs:
    - type: console
    - type: file
      path: /var/log/app.log
      max_size: 100MB
```

### **Docker Compose:**
```yaml
version: "3.8"

services:
  web:
    image: nginx:alpine
    container_name: web_server
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html:ro
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    environment:
      NGINX_HOST: example.com
      NGINX_PORT: 80
    networks:
      - frontend
    restart: unless-stopped
    
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
    container_name: application
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    environment:
      DATABASE_URL: postgresql://user:pass@db:5432/appdb
      REDIS_URL: redis://cache:6379/0
      DEBUG: "false"
    depends_on:
      - db
      - cache
    networks:
      - frontend
      - backend
    restart: unless-stopped
    
  db:
    image: postgres:14-alpine
    container_name: database
    volumes:
      - db_data:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: appdb
    networks:
      - backend
    restart: unless-stopped
    
  cache:
    image: redis:7-alpine
    container_name: redis_cache
    networks:
      - backend
    restart: unless-stopped

networks:
  frontend:
  backend:

volumes:
  db_data:
```

### **Kubernetes ConfigMap:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: production
  labels:
    app: myapp
    env: production
data:
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    logging.level=INFO
  
  database.yaml: |
    host: postgres.production.svc.cluster.local
    port: 5432
    database: myapp
    
  config.json: |
    {
      "features": {
        "cache": true,
        "monitoring": true
      }
    }
```

### **GitHub Actions Workflow:**
```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main

env:
  NODE_VERSION: 16
  DOCKER_REGISTRY: ghcr.io

jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [14, 16, 18]
        
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: ${ { matrix.node-version } }
          cache: npm
          
      - name: Install
        run: npm ci
        
      - name: Test
        run: npm test
        env:
          CI: true
          
  build:
    needs: test
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout
        uses: actions/checkout@v3
        
      - name: Docker Build
        run: |
          docker build -t ${ { env.DOCKER_REGISTRY } }/myapp:latest .
          
      - name: Push
        if: github.ref == 'refs/heads/main'
        run: |
          echo "${ { secrets.GITHUB_TOKEN } }" | docker login ghcr.io -u ${ { github.actor } } --password-stdin
          docker push ${ { env.DOCKER_REGISTRY } }/myapp:latest
```

---

## 🎯 Quick Check: Do You Understand?

1. **What are three other names for YAML dictionaries?**
   <details>
   <summary>Answer</summary>
   Maps, hashes, objects (any 3 are correct)
   </details>

2. **How do you access nested values?**
   <details>
   <summary>Answer</summary>
   Use indentation to show nesting: parent -> child -> grandchild
   </details>

3. **What's the flow style syntax for dictionaries?**
   <details>
   <summary>Answer</summary>
   Curly braces: {key: value, key2: value2}
   </details>

4. **Can dictionaries contain lists?**
   <details>
   <summary>Answer</summary>
   Yes! Very common pattern in DevOps configs
   </details>

---

## 📝 Hands-on Exercise

**Create a complete application stack:**

Create `stack.yaml`:
```yaml
# Application Stack Configuration

application:
  name: E-Commerce Platform
  version: 2.1.0
  environment: production
  
infrastructure:
  cloud_provider: AWS
  region: us-east-1
  
  compute:
    instance_type: t3.medium
    min_instances: 2
    max_instances: 10
    auto_scaling: true
    
  network:
    vpc_cidr: 10.0.0.0/16
    subnets:
      public:
        - 10.0.1.0/24
        - 10.0.2.0/24
      private:
        - 10.0.10.0/24
        - 10.0.11.0/24
        
  storage:
    database:
      type: RDS
      engine: postgres
      version: "14.7"
      instance_class: db.t3.large
      storage_gb: 100
      backup_retention_days: 7
      multi_az: true
      
    cache:
      type: ElastiCache
      engine: redis
      version: "7.0"
      node_type: cache.t3.micro
      nodes: 2
      
services:
  - name: web
    type: nginx
    version: "1.24"
    replicas: 3
    ports:
      - 80
      - 443
    health_check:
      path: /health
      interval: 10
      timeout: 5
      
  - name: api
    type: custom
    image: myapp/api:2.1.0
    replicas: 5
    ports:
      - 8080
    environment:
      LOG_LEVEL: info
      DB_POOL_SIZE: 20
    resources:
      cpu: 500m
      memory: 1Gi
      
monitoring:
  enabled: true
  tools:
    metrics: prometheus
    logging: elasticsearch
    tracing: jaeger
  alerts:
    - name: HighCPU
      threshold: 80
      duration: 5m
    - name: HighMemory
      threshold: 90
      duration: 5m
```

**Tasks:**
1. Validate the YAML
2. Add a new service (e.g., worker, queue)
3. Add cost tracking section with budget
4. Add security section with firewall rules

---

## ⚠️ Common Mistakes

### **1. Mixing Keys at Wrong Level:**
```yaml
# ❌ WRONG - port at wrong level
server:
  host: localhost
port: 8080

# ✅ CORRECT - port under server
server:
  host: localhost
  port: 8080
```

### **2. Duplicate Keys:**
```yaml
# ❌ WRONG - duplicate 'name' key
person:
  name: John
  age: 30
  name: Jane  # This overwrites first name!

# ✅ CORRECT - unique keys
person:
  first_name: John
  age: 30
  last_name: Doe
```

### **3. Inconsistent Indentation:**
```yaml
# ❌ WRONG
server:
  host: localhost
   port: 8080  # 3 spaces instead of 2

# ✅ CORRECT
server:
  host: localhost
  port: 8080
```

---

## 📝 Key Takeaways

✅ **Dictionaries store key-value pairs**  
✅ **Use indentation for nesting**  
✅ **Keys must be unique at same level**  
✅ **Can mix dictionaries and lists**  
✅ **Block style for readability**  
✅ **Flow style for simple data**  
✅ **Most YAML configs are dictionaries**  

---

## 🔍 Dict vs List Decision Tree

```
Do you need to access by name/key?
├── YES → Use Dictionary
│   Example: person.name, config.port
│
└── NO → Do you need ordered collection?
    ├── YES → Use List
    │   Example: steps[0], users[1]
    │
    └── NO → Still use Dictionary (for clarity)
```

---

## 🚀 Next Steps

**Next lesson:** [06 - Anchors and Aliases](06-anchors-aliases.md) - Reuse configuration with YAML anchors

---

## 💡 Pro Tips

**Access Patterns:**
```python
# Python example
import yaml

data = yaml.safe_load("""
server:
  host: localhost
  port: 8080
""")

# Access nested values
print(data['server']['host'])  # localhost
print(data['server']['port'])  # 8080
```

**Validation:**
```bash
# Check structure with yq
yq '.server.host' config.yaml  # Get value
yq '.server | keys' config.yaml  # List keys
yq '.server | length' config.yaml  # Count keys
```

**Organizing Large Configs:**
```yaml
# Group related settings
server:
  # Server settings here
  
database:
  # Database settings here
  
cache:
  # Cache settings here

# Not scattered randomly
```

Dictionaries are the foundation of YAML! 🗂️
