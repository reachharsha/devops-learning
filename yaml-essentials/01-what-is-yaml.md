---
render_with_liquid: false
---
# 01 - What is YAML?

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What YAML is and what it stands for
- Why YAML is popular in DevOps
- YAML vs JSON vs XML
- Where YAML is commonly used

---

## 📝 What is YAML?

**YAML** = **Y**AML **A**in't **M**arkup **L**anguage (recursive acronym)

Originally: "Yet Another Markup Language"

**YAML is a human-readable data serialization format** used for configuration files.

### **Real-World Analogy:**
Think of YAML like a recipe card:
- **Easy to read** - Anyone can understand it
- **Structured** - Organized in a specific format
- **Reusable** - Can be shared and modified
- **Standard** - Everyone follows the same format

---

## 🤔 Why YAML?

### **Advantages:**

✅ **Human-Readable** - Easy to read and write  
✅ **Minimal Syntax** - Less verbose than JSON or XML  
✅ **Comments Supported** - Use `#` for documentation  
✅ **Wide Adoption** - Used by Docker, Kubernetes, Ansible, CI/CD tools  
✅ **Powerful Features** - Anchors, aliases, multi-line strings  

### **Compared to Other Formats:**

| Feature | YAML | JSON | XML |
|---------|------|------|-----|
| Readability | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| Comments | ✅ Yes | ❌ No | ✅ Yes |
| Verbosity | Low | Medium | High |
| Strict | Flexible | Strict | Very Strict |

---

## 📊 YAML vs JSON vs XML

### **Same Data in Different Formats:**

**YAML:**
```yaml
person:
  name: John Doe
  age: 30
  skills:
    - Python
    - Docker
    - Kubernetes
```

**JSON:**
```json
{
  "person": {
    "name": "John Doe",
    "age": 30,
    "skills": ["Python", "Docker", "Kubernetes"]
  }
}
```

**XML:**
```xml
<person>
  <name>John Doe</name>
  <age>30</age>
  <skills>
    <skill>Python</skill>
    <skill>Docker</skill>
    <skill>Kubernetes</skill>
  </skills>
</person>
```

**Winner:** YAML is the most readable! 🏆

---

## 🌍 Where is YAML Used?

### **Docker Compose:**
```yaml
version: '3'
services:
  web:
    image: nginx
    ports:
      - "80:80"
```

### **Kubernetes:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: web
    image: nginx
```

### **GitHub Actions (CI/CD):**
```yaml
name: Deploy
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - run: npm install
```

### **Ansible:**
```yaml
- name: Install nginx
  apt:
    name: nginx
    state: present
```

---

## 📚 Common Use Cases

1. **Configuration Files** - Application settings
2. **CI/CD Pipelines** - GitHub Actions, GitLab CI, Jenkins
3. **Container Orchestration** - Kubernetes manifests
4. **Infrastructure as Code** - Ansible playbooks, Terraform
5. **Docker Compose** - Multi-container applications
6. **API Specifications** - OpenAPI/Swagger
7. **Data Exchange** - Between applications

---

## ⚠️ YAML Gotchas

**Watch out for:**

1. **Indentation Matters** - Use spaces, NOT tabs
2. **Case Sensitive** - `Name` ≠ `name`
3. **Type Inference** - `yes` becomes boolean, not string
4. **Special Characters** - Quotes may be needed

```yaml
# ❌ Wrong - tabs used
name:
→→value

# ✅ Correct - spaces used
name:
  value

# ❌ Wrong - will be boolean true
enabled: yes

# ✅ Correct - will be string
enabled: "yes"
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does YAML stand for?**
   <details>
   <summary>Answer</summary>
   YAML Ain't Markup Language
   </details>

2. **What's the main advantage of YAML over JSON?**
   <details>
   <summary>Answer</summary>
   More readable, supports comments, less verbose
   </details>

3. **Can you use tabs for indentation in YAML?**
   <details>
   <summary>Answer</summary>
   No! Only spaces allowed
   </details>

4. **Name 3 tools that use YAML**
   <details>
   <summary>Answer</summary>
   Docker Compose, Kubernetes, GitHub Actions (many others valid)
   </details>

---

## 📝 Hands-on Exercise

**Create your first YAML file:**

1. Create a file called `person.yaml`
2. Add this content:
```yaml
# My first YAML file
name: Your Name
role: DevOps Engineer
experience_years: 2
skills:
  - Linux
  - Docker
  - YAML
```

3. Validate it at https://www.yamllint.com/
4. Try changing values and re-validating

---

## 📝 Key Takeaways

✅ **YAML is a human-readable data format**  
✅ **Used extensively in DevOps tools**  
✅ **More readable than JSON/XML**  
✅ **Supports comments with #**  
✅ **Indentation with spaces is critical**  
✅ **Case sensitive**  

---

## 🚀 Next Steps

Ready to learn YAML syntax?

**Next lesson:** [02 - Basic Syntax](02-basic-syntax.md) - Learn the fundamentals

---

## 💡 Pro Tips

**Use a good editor:**
- VS Code with YAML extension
- Syntax highlighting helps catch errors
- Auto-completion for common patterns

**Validate often:**
- Use yamllint before deploying
- Many tools have built-in validation
- Catches syntax errors early

**Keep it simple:**
- Start with basic structures
- Add complexity gradually
- Comment your configurations

YAML makes DevOps configurations easy! 📝
