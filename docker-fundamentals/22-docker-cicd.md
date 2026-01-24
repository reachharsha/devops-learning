---
render_with_liquid: false
---
# 22 - Docker in CI/CD Pipelines

## 🎯 Learning Objectives
By the end of this lesson, you'll be able to:
- Integrate Docker into CI/CD workflows
- Build and push images automatically
- Run tests in containers
- Implement multi-stage pipelines
- Use GitHub Actions, GitLab CI, Jenkins
- Deploy containers automatically
- Implement best practices for CI/CD

---

## 🔄 CI/CD Fundamentals

### **CI/CD Pipeline Stages:**
```
1. Code Commit
   ↓
2. Build Docker Image
   ↓
3. Run Tests in Container
   ↓
4. Security Scanning
   ↓
5. Push to Registry
   ↓
6. Deploy to Environment
   ↓
7. Health Checks
   ↓
8. Notification
```

### **Benefits of Docker in CI/CD:**
```
✅ Consistent build environment
✅ Isolated test execution
✅ Reproducible builds
✅ Fast feedback loop
✅ Easy rollback
✅ Version control for infrastructure
✅ Multi-stage deployments
✅ Environment parity (dev = staging = prod)
```

---

## 🐙 GitHub Actions

### **Basic Workflow:**
```yaml
# .github/workflows/docker-build.yml
name: Docker Build and Push

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={ {version} }
            type=semver,pattern={ {major} }.{ {minor} }
            type=sha,prefix={ {branch} }-

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

### **Multi-Stage Pipeline:**
```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Run tests
        run: |
          docker build --target test -t myapp:test .
          docker run --rm myapp:test

  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Lint Dockerfile
        uses: hadolint/hadolint-action@v3.1.0
        with:
          dockerfile: Dockerfile

  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      
      - name: Run Trivy scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'

      - name: Upload results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  build-and-push:
    needs: [test, lint, security-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: |
            ${{ secrets.DOCKERHUB_USERNAME }}/myapp:latest
            ${{ secrets.DOCKERHUB_USERNAME }}/myapp:${{ github.sha }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - name: Deploy to production
        run: |
          # Deploy commands here
          echo "Deploying version ${{ github.sha }}"
```

---

## 🦊 GitLab CI/CD

### **Basic Pipeline:**
```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - security
  - push
  - deploy

variables:
  DOCKER_DRIVER: overlay2
  IMAGE_TAG: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA

before_script:
  - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $IMAGE_TAG .
    - docker push $IMAGE_TAG
  only:
    - main
    - develop

test:
  stage: test
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull $IMAGE_TAG
    - docker run --rm $IMAGE_TAG npm test
  dependencies:
    - build

security_scan:
  stage: security
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull $IMAGE_TAG
    - docker run --rm aquasec/trivy image $IMAGE_TAG
  allow_failure: true

push_latest:
  stage: push
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull $IMAGE_TAG
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main

deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - |
      curl -X POST https://api.production.com/deploy \
        -H "Authorization: Bearer $DEPLOY_TOKEN" \
        -d "image=$IMAGE_TAG"
  only:
    - main
  when: manual
```

### **Multi-Environment Deployment:**
```yaml
# .gitlab-ci.yml
.deploy_template: &deploy
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker pull $IMAGE_TAG
    - docker tag $IMAGE_TAG $CI_REGISTRY_IMAGE:$ENVIRONMENT
    - docker push $CI_REGISTRY_IMAGE:$ENVIRONMENT
    - |
      ssh $DEPLOY_USER@$DEPLOY_HOST << 'EOF'
        docker pull $CI_REGISTRY_IMAGE:$ENVIRONMENT
        docker stop myapp-$ENVIRONMENT || true
        docker rm myapp-$ENVIRONMENT || true
        docker run -d \
          --name myapp-$ENVIRONMENT \
          --restart unless-stopped \
          -p $PORT:3000 \
          $CI_REGISTRY_IMAGE:$ENVIRONMENT
      EOF

deploy_dev:
  <<: *deploy
  stage: deploy
  environment:
    name: development
  variables:
    ENVIRONMENT: dev
    PORT: 3001
  only:
    - develop

deploy_staging:
  <<: *deploy
  stage: deploy
  environment:
    name: staging
  variables:
    ENVIRONMENT: staging
    PORT: 3002
  only:
    - main

deploy_production:
  <<: *deploy
  stage: deploy
  environment:
    name: production
  variables:
    ENVIRONMENT: prod
    PORT: 3000
  only:
    - tags
  when: manual
```

---

## 🔧 Jenkins Pipeline

### **Declarative Pipeline:**
```groovy
// Jenkinsfile
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE = 'myusername/myapp'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    dockerImage.inside {
                        sh 'npm test'
                    }
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh """
                    docker run --rm \
                        -v /var/run/docker.sock:/var/run/docker.sock \
                        aquasec/trivy image ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                """
            }
        }
        
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS) {
                        dockerImage.push("${env.BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        stage('Deploy to Dev') {
            steps {
                sh """
                    docker stop myapp-dev || true
                    docker rm myapp-dev || true
                    docker run -d \
                        --name myapp-dev \
                        -p 3001:3000 \
                        ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                """
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh """
                    docker stop myapp-prod || true
                    docker rm myapp-prod || true
                    docker run -d \
                        --name myapp-prod \
                        --restart unless-stopped \
                        -p 3000:3000 \
                        ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                """
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            slackSend color: 'good', message: "Build ${env.BUILD_NUMBER} succeeded!"
        }
        failure {
            slackSend color: 'danger', message: "Build ${env.BUILD_NUMBER} failed!"
        }
    }
}
```

### **Scripted Pipeline with Docker Compose:**
```groovy
// Jenkinsfile
node {
    def app
    
    stage('Clone repository') {
        checkout scm
    }
    
    stage('Build images') {
        sh 'docker-compose build'
    }
    
    stage('Run tests') {
        sh 'docker-compose run --rm app npm test'
    }
    
    stage('Push images') {
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
            sh 'docker-compose push'
        }
    }
    
    stage('Deploy') {
        sh '''
            docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
        '''
    }
    
    stage('Cleanup') {
        sh 'docker system prune -f'
    }
}
```

---

## 🔐 Security in CI/CD

### **Scanning Images:**
```yaml
# GitHub Actions
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:latest'
    format: 'table'
    exit-code: '1'
    ignore-unfixed: true
    severity: 'CRITICAL,HIGH'

# GitLab CI
security_scan:
  stage: security
  image: docker:latest
  script:
    - docker run --rm aquasec/trivy image --severity HIGH,CRITICAL myapp:latest
  allow_failure: false
```

### **Secrets Management:**
```yaml
# GitHub Actions - Use GitHub Secrets
- name: Login to Registry
  env:
    DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
    DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
  run: |
    echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin

# GitLab CI - Use CI/CD Variables
- docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY

# Jenkins - Use Credentials
docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
    dockerImage.push()
}
```

---

## 📦 Docker Compose in CI/CD

### **Integration Tests:**
```yaml
# .github/workflows/integration-test.yml
name: Integration Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Start services
        run: docker-compose up -d
      
      - name: Wait for services
        run: |
          timeout 60 bash -c 'until docker-compose exec -T web curl -f http://localhost:3000/health; do sleep 1; done'
      
      - name: Run integration tests
        run: docker-compose exec -T app npm run test:integration
      
      - name: View logs
        if: failure()
        run: docker-compose logs
      
      - name: Cleanup
        if: always()
        run: docker-compose down -v
```

---

## 🎯 Best Practices

### **Optimized Dockerfile for CI/CD:**
```dockerfile
# syntax=docker/dockerfile:1

# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci --only=production

# Test stage (for CI)
FROM builder AS test
RUN npm ci
COPY . .
RUN npm run lint
RUN npm test

# Production stage
FROM node:18-alpine AS production
RUN addgroup -g 1001 nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --chown=nodejs:nodejs . .
USER nodejs
EXPOSE 3000
HEALTHCHECK CMD node healthcheck.js
CMD ["node", "index.js"]
```

### **CI/CD Best Practices:**
```yaml
1. ✅ Always tag images with commit SHA
2. ✅ Run tests in containers
3. ✅ Scan for vulnerabilities
4. ✅ Use multi-stage builds
5. ✅ Cache layers efficiently
6. ✅ Implement health checks
7. ✅ Use semantic versioning
8. ✅ Deploy to staging first
9. ✅ Implement rollback mechanism
10. ✅ Monitor deployments
```

---

## 🎯 Quick Check: Do You Understand?

1. **Why use Docker in CI/CD?**
   <details>
   <summary>Answer</summary>
   Consistent environments, reproducible builds, isolated tests, easy deployment
   </details>

2. **What should you scan images for?**
   <details>
   <summary>Answer</summary>
   Vulnerabilities, security issues, secrets, compliance violations
   </details>

3. **How should images be tagged?**
   <details>
   <summary>Answer</summary>
   Git commit SHA, semantic version, branch name, latest for main branch
   </details>

4. **When should tests run?**
   <details>
   <summary>Answer</summary>
   After build, before push to registry, in isolated containers
   </details>

5. **What's a good deployment strategy?**
   <details>
   <summary>Answer</summary>
   Test → Stage → Production with manual approval for prod
   </details>

---

## 📝 Hands-on Exercise

**Complete CI/CD pipeline:**

```bash
# 1. Create project
mkdir cicd-demo
cd cicd-demo

# 2. Create app
cat > app.js << 'EOF'
const express = require('express');
const app = express();
app.get('/', (req, res) => res.json({ status: 'ok' }));
app.get('/health', (req, res) => res.json({ healthy: true }));
app.listen(3000, () => console.log('Started'));
module.exports = app;
EOF

cat > package.json << 'EOF'
{
  "name": "cicd-demo",
  "version": "1.0.0",
  "scripts": {
    "start": "node app.js",
    "test": "echo 'Tests passed'"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
EOF

# 3. Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS test
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm test

FROM node:18-alpine
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# 4. Create GitHub Actions workflow
mkdir -p .github/workflows
cat > .github/workflows/ci.yml << 'EOF'
name: CI

on: [push]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build
        run: docker build --target test -t app:test .
      - name: Test
        run: docker run --rm app:test
EOF

# 5. Initialize git and push
git init
git add .
git commit -m "Initial commit"
# git remote add origin <your-repo>
# git push -u origin main

# Cleanup
cd ..
rm -rf cicd-demo
```

---

## 📝 Key Takeaways

✅ **Automate everything with CI/CD**  
✅ **Test in containers for consistency**  
✅ **Scan images for security issues**  
✅ **Tag images with commit SHA**  
✅ **Use multi-stage builds**  
✅ **Cache layers for faster builds**  
✅ **Deploy to staging before production**  
✅ **Implement health checks**  
✅ **Monitor deployments**  
✅ **Have rollback plan ready**  

---

## 🚀 Next Steps

**Build microservices:**

**Next lesson:** [23 - Microservices with Docker](23-microservices.md) - Microservices architecture

Automate all the things! 🚀
