---
render_with_liquid: false
---
# 10 - Docker Compose Introduction

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Understand what Docker Compose is and why it's useful
- Write docker-compose.yml files
- Manage multi-container applications
- Use Compose commands effectively
- Configure services, networks, and volumes
- Work with environment variables
- Scale services

---

## 🎼 What is Docker Compose?

**Docker Compose** = Tool for defining and running multi-container Docker applications.

### **The Problem:**
```bash
# Without Compose - manual container management
docker network create myapp
docker volume create db-data

docker run -d --name db --network myapp -v db-data:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret postgres:15

docker run -d --name cache --network myapp redis:alpine

docker run -d --name api --network myapp -p 8000:8000 \
  -e DB_HOST=db -e REDIS_HOST=cache myapi:latest

docker run -d --name web --network myapp -p 80:80 nginx

# Too many commands! 😰
# Hard to remember all options
# Difficult to share with team
# Error-prone
```

### **The Solution:**
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - db-data:/var/lib/postgresql/data

  cache:
    image: redis:alpine

  api:
    image: myapi:latest
    ports:
      - "8000:8000"
    environment:
      DB_HOST: db
      REDIS_HOST: cache

  web:
    image: nginx
    ports:
      - "80:80"

volumes:
  db-data:
```

```bash
# One command to rule them all! 🎉
docker-compose up -d
```

---

## 📄 Compose File Structure

### **Basic Anatomy:**
```yaml
version: '3.8'  # Compose file version

services:       # Define containers
  service1:
    # Service configuration
  service2:
    # Service configuration

networks:       # Define networks (optional)
  network1:

volumes:        # Define volumes (optional)
  volume1:
```

### **Complete Example:**
```yaml
version: '3.8'

services:
  # Database service
  postgres:
    image: postgres:15
    container_name: my-postgres
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - backend
    restart: unless-stopped

  # Cache service
  redis:
    image: redis:alpine
    container_name: my-redis
    networks:
      - backend
    restart: unless-stopped

  # Application service
  app:
    build: .  # Build from Dockerfile in current directory
    container_name: my-app
    ports:
      - "8000:8000"
    environment:
      DATABASE_URL: postgres://admin:secret@postgres:5432/mydb
      REDIS_URL: redis://redis:6379
    depends_on:
      - postgres
      - redis
    networks:
      - frontend
      - backend
    restart: unless-stopped

  # Web server
  nginx:
    image: nginx:alpine
    container_name: my-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - app
    networks:
      - frontend
    restart: unless-stopped

networks:
  frontend:
  backend:

volumes:
  postgres-data:
```

---

## 🛠️ Common Service Configuration

### **Image or Build:**
```yaml
services:
  # Use existing image
  db:
    image: postgres:15

  # Build from Dockerfile
  app:
    build: .

  # Build with context and dockerfile
  api:
    build:
      context: ./api
      dockerfile: Dockerfile.prod
      args:
        NODE_ENV: production

  # Build with target stage
  frontend:
    build:
      context: ./frontend
      target: production
```

### **Ports:**
```yaml
services:
  web:
    image: nginx
    ports:
      # Short syntax
      - "8080:80"      # host:container
      - "443:443"
      
      # Long syntax
      - target: 80     # container port
        published: 8080  # host port
        protocol: tcp
        mode: host
```

### **Environment Variables:**
```yaml
services:
  app:
    image: myapp
    environment:
      # Method 1: Key-value pairs
      NODE_ENV: production
      PORT: 8000
      DEBUG: "false"
      
    # Method 2: Array format
    environment:
      - NODE_ENV=production
      - PORT=8000
      
    # Method 3: From .env file
    env_file:
      - .env
      - .env.local
```

### **Volumes:**
```yaml
services:
  app:
    image: myapp
    volumes:
      # Named volume
      - app-data:/app/data
      
      # Bind mount (relative path)
      - ./src:/app/src
      
      # Bind mount (absolute path)
      - /var/log:/app/logs
      
      # Read-only
      - ./config:/app/config:ro
      
      # Long syntax
      - type: bind
        source: ./src
        target: /app/src
      - type: volume
        source: app-data
        target: /app/data

volumes:
  app-data:
```

### **Networks:**
```yaml
services:
  app:
    image: myapp
    networks:
      # Simple
      - frontend
      - backend
      
      # With aliases
      - frontend
      - backend:
          aliases:
            - api-server
            - backend-api

networks:
  frontend:
  backend:
```

### **Depends On:**
```yaml
services:
  app:
    image: myapp
    depends_on:
      # Simple
      - db
      - cache
      
      # With conditions (Compose v2.1+)
      - db:
          condition: service_healthy
      - cache:
          condition: service_started
          
  db:
    image: postgres:15
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### **Restart Policy:**
```yaml
services:
  app:
    image: myapp
    restart: no              # Never restart (default)
    # restart: always        # Always restart
    # restart: on-failure    # Restart on non-zero exit
    # restart: unless-stopped # Always unless manually stopped
```

---

## 🚀 Docker Compose Commands

### **Start Services:**
```bash
# Start all services
docker-compose up

# Start in detached mode
docker-compose up -d

# Build and start
docker-compose up --build

# Start specific services
docker-compose up db cache

# Force recreate containers
docker-compose up --force-recreate

# Scale services
docker-compose up -d --scale api=3
```

### **Stop Services:**
```bash
# Stop all services
docker-compose stop

# Stop specific services
docker-compose stop app web

# Stop and remove containers
docker-compose down

# Stop and remove containers, networks, volumes
docker-compose down -v

# Stop and remove everything including images
docker-compose down --rmi all -v
```

### **View Services:**
```bash
# List running services
docker-compose ps

# View all services (including stopped)
docker-compose ps -a

# View logs
docker-compose logs

# Follow logs
docker-compose logs -f

# Logs for specific service
docker-compose logs -f app

# Last 50 lines
docker-compose logs --tail=50

# With timestamps
docker-compose logs -t
```

### **Execute Commands:**
```bash
# Run command in service
docker-compose exec app sh

# Run command in service (with flags)
docker-compose exec -it app bash

# Run as specific user
docker-compose exec -u root app bash

# Run one-off command
docker-compose run app npm test
docker-compose run --rm app python manage.py migrate
```

### **Build Services:**
```bash
# Build all services
docker-compose build

# Build specific service
docker-compose build app

# Build without cache
docker-compose build --no-cache

# Parallel build
docker-compose build --parallel
```

### **Other Useful Commands:**
```bash
# View config (with interpolations)
docker-compose config

# Validate compose file
docker-compose config -q

# List images
docker-compose images

# Pull images
docker-compose pull

# Push images
docker-compose push

# Pause services
docker-compose pause

# Unpause services
docker-compose unpause

# Restart services
docker-compose restart

# Remove stopped containers
docker-compose rm
```

---

## 📝 Real-World Examples

### **1. WordPress with MySQL:**
```yaml
version: '3.8'

services:
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: wppass
    volumes:
      - db-data:/var/lib/mysql
    restart: unless-stopped

  wordpress:
    image: wordpress:latest
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: wppass
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wp-data:/var/www/html
    depends_on:
      - db
    restart: unless-stopped

volumes:
  db-data:
  wp-data:
```

### **2. Node.js + MongoDB + Redis:**
```yaml
version: '3.8'

services:
  mongo:
    image: mongo:6.0
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: secret
    volumes:
      - mongo-data:/data/db
    networks:
      - backend

  redis:
    image: redis:alpine
    networks:
      - backend

  api:
    build: ./api
    ports:
      - "3000:3000"
    environment:
      MONGO_URL: mongodb://admin:secret@mongo:27017
      REDIS_URL: redis://redis:6379
      NODE_ENV: production
    depends_on:
      - mongo
      - redis
    networks:
      - backend
      - frontend
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - api
    networks:
      - frontend

networks:
  frontend:
  backend:

volumes:
  mongo-data:
```

### **3. Python Flask + PostgreSQL:**
```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: flaskdb
      POSTGRES_USER: flask
      POSTGRES_PASSWORD: secret
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U flask"]
      interval: 10s
      timeout: 5s
      retries: 5

  flask:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    environment:
      DATABASE_URL: postgresql://flask:secret@postgres:5432/flaskdb
      FLASK_ENV: production
    depends_on:
      postgres:
        condition: service_healthy
    restart: unless-stopped

volumes:
  postgres-data:
```

---

## 🔧 Environment Variables

### **.env File:**
```bash
# .env
POSTGRES_USER=admin
POSTGRES_PASSWORD=secret
POSTGRES_DB=mydb
API_PORT=8000
```

### **Use in Compose:**
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}

  api:
    build: .
    ports:
      - "${API_PORT}:8000"
    environment:
      DATABASE_URL: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@db:5432/${POSTGRES_DB}
```

### **Multiple .env Files:**
```yaml
services:
  app:
    image: myapp
    env_file:
      - .env           # Default environment
      - .env.local     # Local overrides
      - secrets.env    # Secrets
```

---

## 📊 Scaling Services

### **Scale Command:**
```bash
# Scale specific service
docker-compose up -d --scale api=3

# Multiple services
docker-compose up -d --scale api=3 --scale worker=5
```

### **Compose File for Scaling:**
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"

  api:
    image: myapi:latest
    # No port mapping! Nginx will load balance
    environment:
      NODE_ENV: production
    deploy:
      replicas: 3  # For swarm mode

# Scale manually:
# docker-compose up -d --scale api=5
```

---

## 🎯 Quick Check: Do You Understand?

1. **What file does Docker Compose read by default?**
   <details>
   <summary>Answer</summary>
   docker-compose.yml
   </details>

2. **How do you start services in background?**
   <details>
   <summary>Answer</summary>
   docker-compose up -d
   </details>

3. **What does `depends_on` do?**
   <details>
   <summary>Answer</summary>
   Starts services in dependency order (but doesn't wait for readiness)
   </details>

4. **How do you rebuild and start services?**
   <details>
   <summary>Answer</summary>
   docker-compose up --build
   </details>

5. **How do you remove all containers, networks, and volumes?**
   <details>
   <summary>Answer</summary>
   docker-compose down -v
   </details>

---

## 📝 Hands-on Exercise

**Create a complete web application:**

```bash
# 1. Create project directory
mkdir my-webapp
cd my-webapp

# 2. Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: webapp
      POSTGRES_USER: admin
      POSTGRES_PASSWORD: secret
    volumes:
      - pg-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U admin"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:alpine

  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    depends_on:
      - postgres
      - redis

volumes:
  pg-data:
EOF

# 3. Start services
docker-compose up -d

# 4. Check status
docker-compose ps

# 5. View logs
docker-compose logs

# 6. Execute command
docker-compose exec postgres psql -U admin -d webapp -c "\l"

# 7. Scale (if applicable)
docker-compose up -d --scale web=2

# 8. Stop services
docker-compose stop

# 9. Start again
docker-compose start

# 10. Remove everything
docker-compose down -v

# Cleanup
cd ..
rm -rf my-webapp
```

---

## 📝 Key Takeaways

✅ **Compose manages multi-container apps with one file**  
✅ **`docker-compose.yml` defines services, networks, volumes**  
✅ **`up` starts services, `down` stops and removes**  
✅ **Services can reference each other by name**  
✅ **`depends_on` sets startup order**  
✅ **Environment variables from `.env` file**  
✅ **`--scale` runs multiple instances of a service**  
✅ **`exec` runs commands in running services**  
✅ **Compose creates isolated environments**  
✅ **Perfect for development and testing**  

---

## 🚀 Next Steps

**Master advanced Compose features:**

**Next lesson:** [11 - Docker Compose Advanced](11-docker-compose-advanced.md) - Advanced patterns and production use

Compose simplifies everything - use it for all multi-container apps! 🎼

