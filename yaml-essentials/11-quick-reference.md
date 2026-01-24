---
render_with_liquid: false
---
# 11 - Quick Reference & Best Practices

## 🎯 Learning Objectives
This is your YAML cheat sheet covering:
- Syntax quick reference
- Common patterns
- Best practices
- Troubleshooting tips
- Tool recommendations

---

## 📋 Syntax Cheat Sheet

### **Basic Types:**
```yaml
# Strings
string1: Hello World
string2: "Quoted string"
string3: 'Single quotes'

# Numbers
integer: 42
float: 3.14
scientific: 1.5e10
hex: 0xFF
octal: 0o77

# Booleans
bool_true: true
bool_false: false
legacy_yes: yes
legacy_no: no

# Null
null1: null
null2: ~
null3:  # empty value

# Dates
date: 2024-03-15
datetime: 2024-03-15T14:30:00Z
```

### **Collections:**
```yaml
# Lists - Block style
fruits:
  - apple
  - banana
  - orange

# Lists - Flow style
numbers: [1, 2, 3, 4, 5]

# Dictionaries - Block style
person:
  name: John
  age: 30
  city: Boston

# Dictionaries - Flow style
config: {debug: true, port: 8080}
```

### **Multi-line Strings:**
```yaml
# Literal (preserves newlines)
script: |
  line 1
  line 2
  line 3

# Folded (joins into one line)
description: >
  This is a long
  paragraph that will
  be folded.

# Strip trailing newlines
no_trail: |-
  content

# Keep trailing newlines
keep_trail: |+
  content
  

```

### **Anchors & Aliases:**
```yaml
# Define anchor
defaults: &default_settings
  timeout: 30
  retries: 3

# Reference alias
dev: *default_settings

# Merge and override
prod:
  <<: *default_settings
  timeout: 60
```

---

## 🎨 Common Patterns

### **Environment-Specific Config:**
```yaml
_defaults: &defaults
  timeout: 30
  retries: 3
  ssl: true

environments:
  development:
    <<: *defaults
    debug: true
    ssl: false
    
  staging:
    <<: *defaults
    debug: false
    
  production:
    <<: *defaults
    debug: false
    timeout: 60
```

### **Service Template:**
```yaml
x-service: &service
  restart: unless-stopped
  logging:
    driver: json-file
    options:
      max-size: "10m"
  networks:
    - app-network

services:
  web:
    <<: *service
    image: nginx
    
  api:
    <<: *service
    build: ./api
```

### **Configuration with Secrets:**
```yaml
# config.yaml
database:
  host: localhost
  port: 5432
  # Don't put passwords in YAML!
  # Use environment variables
  password_env: DB_PASSWORD

# .env file (not in git)
DB_PASSWORD=secret123
```

---

## ✅ Best Practices

### **1. Indentation:**
```yaml
# ✅ GOOD - 2 spaces, consistent
parent:
  child:
    grandchild: value

# ❌ BAD - tabs or inconsistent
parent:
→→child:
   grandchild: value  # 3 spaces
```

### **2. Quoting:**
```yaml
# ✅ GOOD - quote when needed
version: "1.20"
id: "007"
answer: "yes"
url: "http://example.com"

# ❌ BAD - becomes wrong type
version: 1.20    # float, not string
answer: yes      # boolean, not string
```

### **3. Comments:**
```yaml
# ✅ GOOD - explain WHY
timeout: 300  # Increased for slow networks

# Configuration for production database
database:
  host: prod-db.example.com

# ❌ BAD - states the obvious
name: John  # This is a name
port: 80    # Port number
```

### **4. File Organization:**
```yaml
# ✅ GOOD - group related settings
# Server Configuration
server:
  host: localhost
  port: 8080
  workers: 4

# Database Configuration
database:
  host: localhost
  port: 5432

# ❌ BAD - random order
port: 8080
db_host: localhost
workers: 4
host: localhost
```

### **5. Naming Conventions:**
```yaml
# ✅ GOOD - consistent naming
# Use snake_case
database_url: "..."
max_connections: 100
log_level: "info"

# OR use camelCase (pick one!)
databaseUrl: "..."
maxConnections: 100
logLevel: "info"

# ❌ BAD - mixed conventions
database_url: "..."
maxConnections: 100
log-level: "info"
```

---

## ⚠️ Common Mistakes

### **1. Indentation Errors:**
```yaml
# ❌ WRONG
server:
  host: localhost
 port: 8080  # Not aligned!

# ✅ CORRECT
server:
  host: localhost
  port: 8080
```

### **2. Type Coercion:**
```yaml
# ❌ WRONG - becomes boolean
enabled: yes

# ✅ CORRECT - stays string
enabled: "yes"

# ❌ WRONG - loses precision
version: 1.20  # becomes 1.2

# ✅ CORRECT - preserves value
version: "1.20"
```

### **3. Missing Quotes:**
```yaml
# ❌ WRONG - special chars need quotes
url: http://example.com  # : causes issues
email: user@domain.com   # @ might cause issues

# ✅ CORRECT
url: "http://example.com"
email: "user@domain.com"
```

### **4. Anchor Scope:**
```yaml
# ❌ WRONG - anchor not defined yet
user: *defaults
defaults: &defaults
  role: user

# ✅ CORRECT - define before use
defaults: &defaults
  role: user
user: *defaults
```

---

## 🔧 Validation & Tools

### **Online Validators:**
- https://www.yamllint.com/
- https://codebeautify.org/yaml-validator
- https://jsonformatter.org/yaml-validator

### **CLI Tools:**
```bash
# yamllint - Python linter
pip install yamllint
yamllint config.yaml

# yq - YAML processor (like jq)
brew install yq  # macOS
yq '.server.host' config.yaml

# Kubernetes validation
kubectl apply -f manifest.yaml --dry-run=client

# Docker Compose validation
docker-compose config
```

### **Editor Plugins:**
- **VS Code**: YAML extension by Red Hat
- **IntelliJ**: Built-in YAML support
- **Vim**: vim-yaml plugin
- **Sublime**: YAML Macros

### **yamllint Configuration:**
```yaml
# .yamllint
extends: default

rules:
  line-length:
    max: 120
  indentation:
    spaces: 2
  comments:
    min-spaces-from-content: 2
```

---

## 🐛 Troubleshooting Guide

### **Error: "mapping values are not allowed here"**
```yaml
# ❌ Problem: missing quotes
url: http://example.com  # Colon in URL

# ✅ Fix: quote the value
url: "http://example.com"
```

### **Error: "could not find expected ':'"**
```yaml
# ❌ Problem: indentation wrong
server:
host: localhost  # Not indented

# ✅ Fix: proper indentation
server:
  host: localhost
```

### **Error: "found character that cannot start any token"**
```yaml
# ❌ Problem: tabs used
name:
→value

# ✅ Fix: use spaces
name:
  value
```

### **Unexpected Type:**
```yaml
# ❌ Problem: type inference
port: 8080      # integer
version: 1.20   # float 1.2
enabled: yes    # boolean true

# ✅ Fix: quote when needed
port: 8080        # OK as integer
version: "1.20"   # string "1.20"
enabled: "yes"    # string "yes"
```

---

## 📊 YAML vs Other Formats

| Feature | YAML | JSON | XML |
|---------|------|------|-----|
| Readability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Comments | ✅ Yes | ❌ No | ✅ Yes |
| Multi-line | ✅ Easy | ❌ Hard | ✅ Yes |
| Strict | Flexible | Strict | Very Strict |
| File Size | Small | Medium | Large |
| Anchors/Refs | ✅ Yes | ❌ No | ✅ Yes |
| Data Types | Rich | Limited | Custom |

---

## 🎓 Advanced Tips

### **Complex Structures:**
```yaml
# Nested lists and maps
companies:
  - name: TechCorp
    departments:
      - name: Engineering
        teams:
          backend:
            - Alice
            - Bob
          frontend:
            - Carol
      - name: Sales
        teams:
          enterprise:
            - Dave
```

### **YAML in YAML (escaped):**
```yaml
# Store YAML as string
kubernetes_manifest: |
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
    - name: nginx
      image: nginx
```

### **Environment Variable Substitution:**
```yaml
# Note: Not standard YAML, tool-dependent
# Docker Compose supports this
database:
  host: ${DB_HOST:-localhost}
  port: ${DB_PORT:-5432}
  password: ${DB_PASSWORD}
```

---

## 📝 Quick Decision Tree

```
Need to store configuration?
├─ Simple settings → Use YAML
├─ Need strict validation → Consider JSON Schema
├─ API response → Use JSON
├─ Legacy system → Might need XML
└─ Human editing frequently → Definitely YAML!

Choosing YAML style:
├─ Short list (< 5 items) → Flow style [a, b, c]
├─ Long list → Block style (- item)
├─ Short dict → Flow style {key: value}
└─ Complex structure → Block style

Multi-line strings:
├─ Script/code → Literal |
├─ Long paragraph → Folded >
├─ One-line command → Plain string
└─ Need exact formatting → Literal |
```

---

## 🔍 Security Best Practices

```yaml
# ✅ GOOD - no secrets in YAML
database:
  host: localhost
  port: 5432
  password_env: DB_PASSWORD  # Read from environment

# ❌ BAD - secrets in YAML (never do this!)
database:
  host: localhost
  password: "secret123"  # DON'T!

# ✅ GOOD - use secrets management
# Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  password: "actual-secret"  # Encrypted by K8s
```

---

## 📚 Learning Resources

**Official:**
- YAML Spec: https://yaml.org/spec/
- YAML Primer: https://yaml.org/spec/1.2/spec.html

**Tools:**
- yamllint: https://github.com/adrienverge/yamllint
- yq: https://github.com/mikefarah/yq
- Online Validator: https://www.yamllint.com/

**Documentation:**
- Docker Compose: https://docs.docker.com/compose/
- Kubernetes: https://kubernetes.io/docs/
- GitHub Actions: https://docs.github.com/actions
- GitLab CI: https://docs.gitlab.com/ee/ci/

---

## 🎯 Final Checklist

Before committing your YAML:

- [ ] **Validated** with yamllint or online tool
- [ ] **Consistent indentation** (2 spaces)
- [ ] **No tabs** (only spaces)
- [ ] **Quotes used** for version numbers, IDs
- [ ] **Comments explain** non-obvious choices
- [ ] **No secrets** in the file
- [ ] **Tested** in target environment
- [ ] **Formatted** consistently
- [ ] **Anchors used** for repetition (if needed)
- [ ] **File organized** logically

---

## 💡 Pro Tips Summary

1. **Always use 2 spaces** - Most common, most compatible
2. **Quote versions and IDs** - Preserves leading zeros, decimals
3. **Comment the WHY** - Not the what
4. **Validate early and often** - Use yamllint
5. **Use anchors for DRY** - But don't overdo it
6. **Keep it simple** - Complexity increases errors
7. **Test in target tool** - Different parsers, different behavior
8. **Version control** - Track changes
9. **Use linter** - Catches errors before deploy
10. **Read error messages** - They're usually helpful!

---

## 🚀 You're Ready!

You now know:
- ✅ Basic YAML syntax
- ✅ All data types
- ✅ Lists and dictionaries
- ✅ Multi-line strings
- ✅ Anchors and aliases
- ✅ Docker Compose
- ✅ Kubernetes manifests
- ✅ CI/CD pipelines
- ✅ Best practices
- ✅ Troubleshooting

**Go build amazing things with YAML!** 🎉

---

## 📖 Where to Go Next

**Practice:**
1. Convert JSON configs to YAML
2. Create Docker Compose stacks
3. Write Kubernetes manifests
4. Build CI/CD pipelines
5. Contribute to open source projects

**Advanced Topics:**
- JSON Schema for YAML validation
- Custom YAML tags
- YAML 1.2 vs 1.1 differences
- Performance optimization
- Security scanning

**Keep Learning:**
- Join DevOps communities
- Read tool documentation
- Practice, practice, practice!
- Share your knowledge

---

## 🎓 Congratulations!

You've completed the YAML Essentials course!

**Share what you learned:**
- Help teammates understand YAML
- Write better configuration files
- Debug YAML issues faster
- Build more reliable systems

**Remember:**
> *"YAML is easy to read, easy to write, but hard to master. Keep practicing!"*

Happy YAMLing! 🚀

---

**Course Complete!** Return to [00-START-HERE](00-START-HERE.md) for the full roadmap.
