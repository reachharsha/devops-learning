---
render_with_liquid: false
---
# 04 - Lists and Arrays

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- How to create lists in YAML
- Block style vs flow style
- Nested lists
- Lists of objects
- Common list patterns in DevOps tools

---

## 📝 What Are Lists?

**Lists** (also called arrays or sequences) store multiple values in order.

**Real-World Analogy:**
A grocery list - ordered items you need to get.

---

## 📋 Block Style Lists (Common)

### **Basic Syntax:**
```yaml
# List items start with dash and space
fruits:
  - apple
  - banana
  - orange
  - grape
```

**Key Points:**
- Dash `-` indicates list item
- Space after dash is REQUIRED
- Items must be indented consistently

### **Different Data Types:**
```yaml
# Strings
colors:
  - red
  - blue
  - green

# Numbers
numbers:
  - 1
  - 2
  - 3

# Mixed types (valid but uncommon)
mixed:
  - John
  - 30
  - true
```

---

## 🔄 Flow Style Lists (Compact)

### **Inline Format:**
```yaml
# Square brackets, comma separated
fruits: [apple, banana, orange]
numbers: [1, 2, 3, 4, 5]
colors: [red, blue, green]
```

**When to Use:**
- Short lists (3-5 items)
- Simple values
- Saves vertical space

**Block vs Flow:**
```yaml
# Block style - easier to read
ports:
  - 80
  - 443
  - 8080

# Flow style - more compact
ports: [80, 443, 8080]

# Choose based on readability
```

---

## 🎯 Empty Lists

```yaml
# Empty list - block style
empty_list: []

# Empty list - flow style
also_empty: []

# NOT a list - this is null
not_a_list:
```

---

## 📦 Lists of Objects (Very Common!)

### **Syntax:**
```yaml
users:
  - name: John
    age: 30
    role: admin
  - name: Jane
    age: 25
    role: user
  - name: Bob
    age: 35
    role: moderator
```

**Each `- name:` starts a new object in the list**

### **Docker Compose Example:**
```yaml
services:
  - name: web
    image: nginx:latest
    ports:
      - 80
      - 443
      
  - name: database
    image: postgres:14
    environment:
      - POSTGRES_PASSWORD=secret
```

### **Kubernetes Containers:**
```yaml
containers:
  - name: nginx
    image: nginx:1.21
    ports:
      - containerPort: 80
        
  - name: sidecar
    image: busybox
    command: ["/bin/sh", "-c", "while true; do sleep 30; done"]
```

---

## 🔁 Nested Lists

### **List of Lists:**
```yaml
# Matrix
matrix:
  - [1, 2, 3]
  - [4, 5, 6]
  - [7, 8, 9]

# Or block style
coordinates:
  -
    - 10
    - 20
  -
    - 30
    - 40
```

### **Complex Nesting:**
```yaml
departments:
  - name: Engineering
    teams:
      - name: Backend
        members:
          - Alice
          - Bob
      - name: Frontend
        members:
          - Carol
          - Dave
          
  - name: Sales
    teams:
      - name: Enterprise
        members:
          - Eve
          - Frank
```

---

## 🎨 Real-World Examples

### **GitHub Actions Workflow:**
```yaml
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
        
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: 16
          
      - name: Install dependencies
        run: npm install
        
      - name: Run tests
        run: npm test
```

### **Docker Compose Services:**
```yaml
version: "3.8"

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./html:/usr/share/nginx/html
      - ./nginx.conf:/etc/nginx/nginx.conf
    environment:
      - NGINX_HOST=example.com
      - NGINX_PORT=80
      
  database:
    image: postgres:14
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_USER=admin
      - POSTGRES_PASSWORD=secret
      - POSTGRES_DB=myapp
    volumes:
      - db_data:/var/lib/postgresql/data

volumes:
  db_data:
```

### **Kubernetes Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
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
        image: nginx:1.21
        ports:
        - containerPort: 80
        env:
        - name: ENV
          value: production
        - name: DEBUG
          value: "false"
```

### **Ansible Playbook:**
```yaml
- name: Configure web servers
  hosts: webservers
  become: true
  
  tasks:
    - name: Install packages
      apt:
        name:
          - nginx
          - git
          - curl
        state: present
        
    - name: Copy config files
      copy:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
      loop:
        - { src: nginx.conf, dest: /etc/nginx/nginx.conf }
        - { src: site.conf, dest: /etc/nginx/sites-available/default }
        
    - name: Start services
      service:
        name: "{{ item }}"
        state: started
        enabled: true
      loop:
        - nginx
        - ssh
```

---

## 🎯 Quick Check: Do You Understand?

1. **What character starts a list item in block style?**
   <details>
   <summary>Answer</summary>
   Dash/hyphen `-` followed by space
   </details>

2. **How do you create a list in flow style?**
   <details>
   <summary>Answer</summary>
   Square brackets: [item1, item2, item3]
   </details>

3. **Can lists contain different data types?**
   <details>
   <summary>Answer</summary>
   Yes, but it's uncommon. Usually lists have same type items.
   </details>

4. **How do you create a list of objects?**
   <details>
   <summary>Answer</summary>
   Each object starts with `-`, followed by key-value pairs indented below it
   </details>

---

## 📝 Hands-on Exercise

**Create a team roster:**

Create `team.yaml`:
```yaml
# Development Team

team_name: Platform Engineering

members:
  - name: Alice Johnson
    role: Team Lead
    skills:
      - Kubernetes
      - Python
      - AWS
    years_experience: 8
    remote: true
    
  - name: Bob Smith
    role: Senior DevOps Engineer
    skills:
      - Docker
      - Terraform
      - Azure
    years_experience: 5
    remote: false
    
  - name: Carol Davis
    role: DevOps Engineer
    skills:
      - CI/CD
      - Jenkins
      - Linux
    years_experience: 3
    remote: true

# Tech stack used
technologies:
  - Docker
  - Kubernetes
  - Terraform
  - GitHub Actions
  - Prometheus
  - Grafana

# Current projects (compact format)
projects: [Migration to K8s, CI/CD Pipeline, Monitoring Setup]
```

**Tasks:**
1. Validate the YAML
2. Add yourself as a team member
3. Try converting technologies list to flow style
4. Add a new field `certifications` (list) to members

---

## ⚠️ Common Mistakes

### **1. Missing Space After Dash:**
```yaml
# ❌ WRONG
fruits:
  -apple
  -banana

# ✅ CORRECT
fruits:
  - apple
  - banana
```

### **2. Inconsistent Indentation:**
```yaml
# ❌ WRONG
items:
  - first
   - second  # Wrong indent
  - third

# ✅ CORRECT
items:
  - first
  - second
  - third
```

### **3. Mixing Styles Incorrectly:**
```yaml
# ❌ CONFUSING
items:
  - [a, b, c]  # Flow inside block
  - d
  - e

# ✅ BETTER - be consistent
items: [a, b, c, d, e]

# ✅ OR
items:
  - a
  - b
  - c
  - d
  - e
```

### **4. Missing Dash for List Items:**
```yaml
# ❌ WRONG - This is an object, not a list
users:
  name: John
  age: 30

# ✅ CORRECT - This is a list with one object
users:
  - name: John
    age: 30
```

---

## 📝 Key Takeaways

✅ **Block style: `- item` (one per line)**  
✅ **Flow style: `[item1, item2]` (inline)**  
✅ **Dash + space starts list item**  
✅ **Lists can contain objects (very common in DevOps)**  
✅ **Use block style for readability**  
✅ **Use flow style for short, simple lists**  
✅ **Consistent indentation is critical**  

---

## 🎨 Style Guide

**When to use Block Style:**
- Long lists
- Complex items (objects)
- Lists that change frequently
- Maximum readability

**When to use Flow Style:**
- Short lists (< 5 items)
- Simple values (numbers, short strings)
- Compact configuration
- Port numbers, IDs

```yaml
# ✅ GOOD - block for complex items
services:
  - name: web
    image: nginx
    ports: [80, 443]  # Flow for simple list
    
# ✅ GOOD - flow for simple list
allowed_ips: [192.168.1.1, 192.168.1.2, 192.168.1.3]
```

---

## 🚀 Next Steps

**Next lesson:** [05 - Dictionaries and Maps](05-dictionaries-maps.md) - Working with key-value structures

---

## 💡 Pro Tips

**List vs Object:**
```yaml
# LIST - multiple items, accessed by index
fruits:
  - apple   # fruits[0]
  - banana  # fruits[1]

# OBJECT - key-value pairs, accessed by key
person:
  name: John    # person.name or person['name']
  age: 30       # person.age or person['age']
```

**Debugging:**
```bash
# Count list items
yq '.fruits | length' file.yaml

# Get first item
yq '.fruits[0]' file.yaml

# Get all items
yq '.fruits[]' file.yaml
```

**Comments in Lists:**
```yaml
services:
  # Production web server
  - name: web
    image: nginx:latest
    
  # Database - PostgreSQL 14
  - name: db
    image: postgres:14
```

Lists are everywhere in DevOps YAML! 📋
