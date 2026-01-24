---
render_with_liquid: false
---
# 08 - Docker Compose YAML

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Docker Compose file structure
- Common service configurations
- Volumes, networks, and environment variables
- Best practices for Compose files
- Real production examples

---

## 🐳 What is Docker Compose?

**Docker Compose** uses YAML to define multi-container applications.

**File:** `docker-compose.yml` or `docker-compose.yaml`

**Real-World Analogy:**
Like a recipe that lists all ingredients (containers) and instructions (configuration) to make a complete meal (application).

---

## 📋 Basic Structure

```yaml
version: "3.8"  # Compose file version

services:       # Define containers
  service1:
    # config
  service2:
    # config

networks:       # Optional: custom networks
  network1:

volumes:        # Optional: persistent storage
  volume1:
```

---

## 🎨 Simple Example - Web + Database

```yaml
version: "3.8"

services:
  # Web server
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
    depends_on:
      - api
      
  # Application API
  api:
    build: ./app
    ports:
      - "8080:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/myapp
      - REDIS_URL=redis://cache:6379
    depends_on:
      - db
      - cache
      
  # Database
  db:
    image: postgres:14-alpine
    environment:
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=myapp
    volumes:
      - db_data:/var/lib/postgresql/data
      
  # Cache
  cache:
    image: redis:7-alpine

volumes:
  db_data:
```

---

## 🔧 Service Configuration Options

### **Image vs Build:**
```yaml
services:
  # Using pre-built image
  nginx:
    image: nginx:1.21
    
  # Building from Dockerfile
  app:
    build:
      context: ./app
      dockerfile: Dockerfile
      args:
        - NODE_ENV=production
        
  # Build with custom name
  worker:
    build: ./worker
    image: myapp/worker:latest  # Tag after building
```

### **Ports:**
```yaml
services:
  web:
    ports:
      # HOST:CONTAINER
      - "80:80"           # HTTP
      - "443:443"         # HTTPS
      - "8080:8080"       # API
      - "127.0.0.1:3000:3000"  # Bind to localhost only
      - "3306"            # Random host port -> 3306
```

### **Environment Variables:**
```yaml
services:
  app:
    # List format
    environment:
      - NODE_ENV=production
      - DEBUG=false
      - API_KEY=abc123
      
    # Or dictionary format
    environment:
      NODE_ENV: production
      DEBUG: "false"
      API_KEY: abc123
      
    # From file
    env_file:
      - .env
      - .env.production
```

### **Volumes:**
```yaml
services:
  app:
    volumes:
      # Bind mount: HOST:CONTAINER
      - ./app:/app
      - ./config:/etc/app/config:ro  # Read-only
      
      # Named volume
      - app_data:/var/lib/app
      
      # Anonymous volume
      - /var/cache/app

volumes:
  app_data:  # Define named volume
```

### **Networks:**
```yaml
services:
  web:
    networks:
      - frontend
      
  app:
    networks:
      - frontend
      - backend
      
  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

---

## 🚀 Production Example - Full Stack App

```yaml
version: "3.8"

# Extension fields for reusability
x-common: &common
  restart: unless-stopped
  logging:
    driver: "json-file"
    options:
      max-size: "10m"
      max-file: "3"

services:
  # Nginx reverse proxy
  nginx:
    <<: *common
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
      - static_files:/var/www/static:ro
    networks:
      - frontend
    depends_on:
      - web
      - api
      
  # Frontend - React/Vue/Angular
  web:
    <<: *common
    build:
      context: ./frontend
      dockerfile: Dockerfile
      args:
        - REACT_APP_API_URL=https://api.example.com
    container_name: frontend
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - static_files:/app/build
    networks:
      - frontend
    command: npm start
    
  # Backend API - Node.js/Python/Go
  api:
    <<: *common
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: api
    ports:
      - "8080:8080"
    environment:
      - NODE_ENV=production
      - PORT=8080
      - DATABASE_URL=postgresql://postgres:secret@db:5432/myapp
      - REDIS_URL=redis://cache:6379
      - JWT_SECRET=${JWT_SECRET}
      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    volumes:
      - ./backend:/app
      - /app/node_modules
      - upload_files:/app/uploads
    networks:
      - frontend
      - backend
    depends_on:
      - db
      - cache
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
      
  # Worker - Background jobs
  worker:
    <<: *common
    build:
      context: ./backend
      dockerfile: Dockerfile
    container_name: worker
    command: npm run worker
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:secret@db:5432/myapp
      - REDIS_URL=redis://cache:6379
    volumes:
      - ./backend:/app
      - /app/node_modules
    networks:
      - backend
    depends_on:
      - db
      - cache
      
  # PostgreSQL database
  db:
    <<: *common
    image: postgres:14-alpine
    container_name: database
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=myapp
      - PGDATA=/var/lib/postgresql/data/pgdata
    volumes:
      - db_data:/var/lib/postgresql/data
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - backend
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
      
  # Redis cache
  cache:
    <<: *common
    image: redis:7-alpine
    container_name: redis
    command: redis-server --appendonly yes
    volumes:
      - cache_data:/data
    networks:
      - backend
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  db_data:
  cache_data:
  static_files:
  upload_files:
```

---

## 📊 Common Patterns

### **Development vs Production:**

**docker-compose.yml (base):**
```yaml
version: "3.8"

services:
  app:
    build: .
    environment:
      - NODE_ENV=development
```

**docker-compose.override.yml (auto-loaded in dev):**
```yaml
version: "3.8"

services:
  app:
    volumes:
      - ./app:/app  # Hot reload
    ports:
      - "3000:3000"
    command: npm run dev
```

**docker-compose.prod.yml (production):**
```yaml
version: "3.8"

services:
  app:
    image: myapp/app:latest
    restart: unless-stopped
    command: npm start
```

**Usage:**
```bash
# Development (uses override automatically)
docker-compose up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the default Compose file name?**
   <details>
   <summary>Answer</summary>
   docker-compose.yml or docker-compose.yaml
   </details>

2. **How do you expose port 80?**
   <details>
   <summary>Answer</summary>
   ports: - "80:80" (HOST:CONTAINER)
   </details>

3. **What's the difference between `image` and `build`?**
   <details>
   <summary>Answer</summary>
   image: use pre-built image, build: build from Dockerfile
   </details>

4. **How do you make a service wait for another?**
   <details>
   <summary>Answer</summary>
   depends_on: - service_name
   </details>

---

## 📝 Hands-on Exercise

**Create a WordPress stack:**

Create `docker-compose.yml`:
```yaml
version: "3.8"

services:
  # WordPress
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    ports:
      - "8000:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: secret
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp_data:/var/www/html
    depends_on:
      - db
    restart: unless-stopped
    
  # MySQL database
  db:
    image: mysql:8.0
    container_name: mysql
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: secret
      MYSQL_ROOT_PASSWORD: rootsecret
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped
    
  # phpMyAdmin (optional)
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: rootsecret
    depends_on:
      - db
    restart: unless-stopped

volumes:
  wp_data:
  db_data:
```

**Tasks:**
1. Run `docker-compose up -d`
2. Access WordPress at http://localhost:8000
3. Add a Redis cache service
4. Add health checks to all services

---

## ⚠️ Common Mistakes

### **1. Wrong Port Mapping:**
```yaml
# ❌ WRONG - backwards
ports:
  - "80:8080"  # Means: container 8080 -> host 80

# ✅ CORRECT - usually want same
ports:
  - "8080:8080"  # Container 8080 -> host 8080
```

### **2. Missing Volume Definition:**
```yaml
# ❌ WRONG - named volume not defined
services:
  db:
    volumes:
      - db_data:/data

# ✅ CORRECT - define volume
services:
  db:
    volumes:
      - db_data:/data
      
volumes:
  db_data:
```

### **3. Environment Variable Types:**
```yaml
# ⚠️ CAREFUL - these become strings
environment:
  PORT: 8080      # String "8080"
  DEBUG: true     # String "true"

# ✅ BETTER - quote for clarity
environment:
  PORT: "8080"
  DEBUG: "true"
```

---

## 📝 Key Takeaways

✅ **Compose defines multi-container apps in YAML**  
✅ **Services = containers**  
✅ **Volumes = persistent data**  
✅ **Networks = container communication**  
✅ **Ports: HOST:CONTAINER format**  
✅ **Use depends_on for startup order**  
✅ **anchors (&/*) reduce repetition**  

---

## 🚀 Next Steps

**Next lesson:** [09 - Kubernetes YAML](09-kubernetes.md) - K8s manifests and deployments

---

## 💡 Pro Tips

**Useful Commands:**
```bash
# Start services
docker-compose up -d

# View logs
docker-compose logs -f service_name

# Stop services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Validate compose file
docker-compose config

# Scale service
docker-compose up -d --scale worker=3
```

**Environment Files (.env):**
```bash
# .env file
POSTGRES_PASSWORD=secret
API_KEY=abc123

# Automatically loaded
docker-compose up
```

Docker Compose makes multi-container apps easy! 🐳
