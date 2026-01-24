---
render_with_liquid: false
---
# 10 - CI/CD Pipeline YAML

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- GitHub Actions workflow syntax
- GitLab CI pipeline configuration
- Common CI/CD patterns
- Building, testing, and deploying with YAML
- Real-world pipeline examples

---

## 🔄 What is CI/CD?

**CI** = Continuous Integration - Automatically test code changes  
**CD** = Continuous Deployment - Automatically deploy to production

**YAML defines:**
- When to run (triggers)
- What to run (jobs/steps)
- Where to run (runners)
- How to run (commands)

---

## 🐙 GitHub Actions

### **Basic Workflow Structure:**
```yaml
name: Workflow Name

on:  # Triggers
  push:
  pull_request:

jobs:  # Jobs to run
  job1:
    runs-on: ubuntu-latest
    steps:
      - name: Step 1
        run: command
```

### **Simple CI Workflow:**
```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'npm'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Run linter
        run: npm run lint
```

---

## 🚀 Complete CI/CD Pipeline (GitHub Actions)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  release:
    types: [published]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${ { github.repository } }

jobs:
  # Job 1: Test
  test:
    name: Test Application
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [16, 18, 20]
        
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js ${ { matrix.node-version } }
        uses: actions/setup-node@v3
        with:
          node-version: ${ { matrix.node-version } }
          cache: npm
          
      - name: Install dependencies
        run: npm ci
        
      - name: Run tests
        run: npm test
        
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/coverage.xml
          
  # Job 2: Lint
  lint:
    name: Code Quality
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm
          
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
      
  # Job 3: Security Scan
  security:
    name: Security Scan
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Run Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${ { secrets.SNYK_TOKEN } }
          
  # Job 4: Build Docker Image
  build:
    name: Build Image
    runs-on: ubuntu-latest
    needs: [test, lint, security]
    
    permissions:
      contents: read
      packages: write
      
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
        
      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${ { env.REGISTRY } }
          username: ${ { github.actor } }
          password: ${ { secrets.GITHUB_TOKEN } }
          
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ${ { env.REGISTRY } }/${ { env.IMAGE_NAME } }
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={ {version} }
            type=semver,pattern={ {major} }.{ {minor} }
            type=sha
            
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${ { steps.meta.outputs.tags } }
          labels: ${ { steps.meta.outputs.labels } }
          cache-from: type=gha
          cache-to: type=gha,mode=max
          
  # Job 5: Deploy to Staging
  deploy-staging:
    name: Deploy to Staging
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/develop'
    
    environment:
      name: staging
      url: https://staging.example.com
      
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${ { secrets.KUBE_CONFIG_STAGING } }
          
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/app \
            app=${ { env.REGISTRY } }/${ { env.IMAGE_NAME } }:${ { github.sha } }
          kubectl rollout status deployment/app
          
      - name: Run smoke tests
        run: |
          curl -f https://staging.example.com/health || exit 1
          
  # Job 6: Deploy to Production
  deploy-production:
    name: Deploy to Production
    runs-on: ubuntu-latest
    needs: build
    if: github.event_name == 'release'
    
    environment:
      name: production
      url: https://example.com
      
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure kubectl
        uses: azure/k8s-set-context@v3
        with:
          method: kubeconfig
          kubeconfig: ${ { secrets.KUBE_CONFIG_PROD } }
          
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/app \
            app=${ { env.REGISTRY } }/${ { env.IMAGE_NAME } }:${ { github.event.release.tag_name } }
          kubectl rollout status deployment/app
          
      - name: Notify Slack
        uses: 8398a7/action-slack@v3
        with:
          status: ${ { job.status } }
          text: 'Production deployment completed!'
          webhook_url: ${ { secrets.SLACK_WEBHOOK } }
```

---

## 🦊 GitLab CI/CD

### **Basic Pipeline:**
```yaml
# .gitlab-ci.yml

stages:
  - test
  - build
  - deploy

# Global variables
variables:
  NODE_VERSION: "18"
  
# Test job
test:
  stage: test
  image: node:18
  script:
    - npm ci
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'
  
# Build job
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    
# Deploy job
deploy:
  stage: deploy
  script:
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

### **Complete GitLab Pipeline:**
```yaml
# .gitlab-ci.yml

image: node:18

stages:
  - install
  - test
  - build
  - deploy

# Cache node_modules
cache:
  paths:
    - node_modules/

# Install dependencies
install:
  stage: install
  script:
    - npm ci
  artifacts:
    paths:
      - node_modules/
    expire_in: 1 day

# Lint code
lint:
  stage: test
  needs: [install]
  script:
    - npm run lint
    - npm run format:check

# Unit tests
test:unit:
  stage: test
  needs: [install]
  script:
    - npm run test:unit
  coverage: '/All files[^|]*\|[^|]*\s+([\d\.]+)/'
  artifacts:
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage/cobertura-coverage.xml

# Integration tests
test:integration:
  stage: test
  needs: [install]
  services:
    - postgres:14
    - redis:7
  variables:
    POSTGRES_DB: test_db
    POSTGRES_USER: test
    POSTGRES_PASSWORD: test
  script:
    - npm run test:integration

# Security scan
security:
  stage: test
  image: aquasec/trivy:latest
  script:
    - trivy fs --exit-code 1 --severity HIGH,CRITICAL .

# Build Docker image
build:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker tag $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - docker push $CI_REGISTRY_IMAGE:latest
  only:
    - main
    - develop

# Deploy to staging
deploy:staging:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context staging
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kubectl rollout status deployment/app
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

# Deploy to production
deploy:production:
  stage: deploy
  image: bitnami/kubectl:latest
  script:
    - kubectl config use-context production
    - kubectl set image deployment/app app=$CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
    - kubectl rollout status deployment/app
  environment:
    name: production
    url: https://example.com
  when: manual
  only:
    - main
```

---

## 🎨 Common Patterns

### **Matrix Builds:**
```yaml
# GitHub Actions
strategy:
  matrix:
    os: [ubuntu-latest, windows-latest, macos-latest]
    node: [16, 18, 20]
runs-on: ${ { matrix.os } }
```

### **Conditional Jobs:**
```yaml
# GitHub Actions - only on main branch
deploy:
  if: github.ref == 'refs/heads/main'
  
# GitLab CI - only on tags
deploy:
  only:
    - tags
  except:
    - branches
```

### **Reusable Workflows (GitHub):**
```yaml
# .github/workflows/reusable.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${ { inputs.environment } }"

# Use it
jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable.yml
    with:
      environment: staging
```

---

## 🎯 Quick Check: Do You Understand?

1. **What triggers a GitHub Actions workflow?**
   <details>
   <summary>Answer</summary>
   on: push, pull_request, schedule, workflow_dispatch, etc.
   </details>

2. **How do you run jobs in parallel?**
   <details>
   <summary>Answer</summary>
   Don't use 'needs' - jobs without dependencies run in parallel
   </details>

3. **How do you require job completion before another?**
   <details>
   <summary>Answer</summary>
   Use needs: [job1, job2]
   </details>

4. **GitLab stages vs jobs?**
   <details>
   <summary>Answer</summary>
   Stages run sequentially, jobs within a stage run in parallel
   </details>

---

## 📝 Hands-on Exercise

**Create a Node.js CI/CD pipeline:**

Create `.github/workflows/nodejs.yml`:
```yaml
name: Node.js CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: npm
          
      - name: Install
        run: npm ci
        
      - name: Lint
        run: npm run lint
        
      - name: Test
        run: npm test
        
  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Build
        run: npm run build
        
      - name: Upload artifact
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
```

**Tasks:**
1. Add a deploy job after build
2. Add matrix testing for Node 16, 18, 20
3. Add Docker build step
4. Add Slack notification on failure

---

## ⚠️ Common Mistakes

### **1. Missing needs:**
```yaml
# ❌ WRONG - deploy runs before test completes
jobs:
  test:
    ...
  deploy:
    ...

# ✅ CORRECT - deploy waits for test
jobs:
  test:
    ...
  deploy:
    needs: test
```

### **2. Secrets in Logs:**
```yaml
# ❌ WRONG - exposes secret
run: echo "Token: ${ { secrets.API_TOKEN } }"

# ✅ CORRECT - use environment variable
env:
  API_TOKEN: ${ { secrets.API_TOKEN } }
run: ./deploy.sh  # Script uses $API_TOKEN
```

---

## 📝 Key Takeaways

✅ **YAML defines entire CI/CD pipeline**  
✅ **Jobs run in parallel by default**  
✅ **Use needs for dependencies**  
✅ **Matrix for multiple configurations**  
✅ **Secrets for sensitive data**  
✅ **Artifacts to share between jobs**  
✅ **Environments for deployment tracking**  

---

## 🚀 Next Steps

**Next lesson:** [11 - Quick Reference](11-quick-reference.md) - Cheat sheet and best practices

---

## 💡 Pro Tips

**Local Testing:**
```bash
# GitHub Actions - use act
act -j test

# GitLab CI - use gitlab-runner
gitlab-runner exec docker test
```

**Debugging:**
```yaml
# GitHub Actions
- name: Debug
  run: |
    echo "Event: ${ { github.event_name } }"
    echo "Ref: ${ { github.ref } }"
    echo "SHA: ${ { github.sha } }"

# GitLab CI
- echo "Pipeline: $CI_PIPELINE_ID"
- echo "Branch: $CI_COMMIT_BRANCH"
- echo "Tag: $CI_COMMIT_TAG"
```

Automate everything with YAML! 🔄
