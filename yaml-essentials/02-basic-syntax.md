---
render_with_liquid: false
---
# 02 - Basic YAML Syntax

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- YAML file structure and rules
- Indentation requirements
- Key-value pairs
- Comments and documentation
- Basic data formatting

---

## 📏 YAML Rules

### **1. File Extension**
```bash
config.yaml    # ✅ Recommended
config.yml     # ✅ Also common
```

### **2. Indentation (CRITICAL!)**

**Rules:**
- Use **2 spaces** per level (standard)
- **NO TABS** - only spaces allowed
- Consistent indentation required

```yaml
# ✅ CORRECT
parent:
  child:
    grandchild: value

# ❌ WRONG - tabs used
parent:
→→child:
→→→→grandchild: value

# ❌ WRONG - inconsistent spacing
parent:
  child:
     grandchind: value  # 3 spaces instead of 2
```

**Real-World Analogy:**
Think of indentation like an organization chart - each level shows parent-child relationships.

---

## 🔑 Key-Value Pairs

### **Basic Syntax:**
```yaml
key: value
```

**The colon `:` separates key from value**

### **Examples:**
```yaml
name: John Doe
age: 30
city: New York
active: true
```

### **Spacing Rules:**
```yaml
# ✅ CORRECT - space after colon
name: value

# ❌ WRONG - no space after colon
name:value

# ✅ Also correct - space before value
name:    value

# ✅ No value (null)
name:
```

---

## 🗂️ Nested Structures

### **Simple Nesting:**
```yaml
person:
  name: John
  age: 30
  address:
    street: 123 Main St
    city: Boston
    zip: 02101
```

**Hierarchy:**
```
person
├── name
├── age
└── address
    ├── street
    ├── city
    └── zip
```

### **Multiple Levels:**
```yaml
company:
  engineering:
    backend:
      team_lead: Alice
      members: 5
    frontend:
      team_lead: Bob
      members: 3
  sales:
    manager: Carol
    quota: 1000000
```

---

## 💬 Comments

### **Syntax:**
```yaml
# This is a comment

name: John  # Inline comment

# Multi-line comments
# just use multiple lines
# with # at the start

server:
  # Configuration for web server
  port: 8080
  host: localhost
```

### **Best Practices:**
```yaml
# ✅ GOOD - Explains why
timeout: 300  # Increased for slow networks

# ✅ GOOD - Documents section
# Database Configuration
database:
  host: localhost
  port: 5432

# ❌ BAD - States the obvious
name: John  # This is a name
```

---

## 📊 Common Data Types

YAML automatically infers types:

### **Strings:**
```yaml
# No quotes needed (usually)
name: John Doe
city: New York

# Quotes optional but allowed
name: "John Doe"
name: 'John Doe'

# Quotes REQUIRED for special cases
special: "yes"        # Without quotes, becomes boolean
version: "1.20"       # Without quotes, becomes number
complex: "key: value" # Contains special characters
```

### **Numbers:**
```yaml
age: 30           # Integer
price: 19.99      # Float
scientific: 1e10  # Scientific notation
octal: 0o14       # Octal (12 in decimal)
hex: 0xC          # Hexadecimal (12 in decimal)
```

### **Booleans:**
```yaml
# True values
enabled: true
active: yes
debug: on

# False values
disabled: false
inactive: no
logging: off
```

**⚠️ Watch out:**
```yaml
# ❌ This becomes boolean true!
answer: yes

# ✅ To keep it as string "yes"
answer: "yes"
```

### **Null Values:**
```yaml
# All of these are null
empty:
nothing: null
tilde: ~
```

---

## 🔤 String Quirks

### **When to Use Quotes:**

```yaml
# ✅ No quotes needed
simple_string: Hello World
another: This is fine

# ⚠️ Quotes NEEDED for these:
special_chars: "key: value"     # Contains :
leading_space: " padded"        # Starts with space
numeric_string: "123"           # Number as string
boolean_string: "true"          # Boolean as string
multiword_value: "yes or no"    # Could be misinterpreted
version: "3.8"                  # Preserve decimal

# Special characters that need quotes
at_symbol: "@username"
hash: "#hashtag"
ampersand: "Q&A"
asterisk: "*important*"
```

### **Single vs Double Quotes:**

```yaml
# Single quotes - literal string
message: 'This is \n literal'
# Output: This is \n literal

# Double quotes - escape sequences work
message: "This has a \n newline"
# Output: This has a
#         newline
```

---

## 📋 Real-World Examples

### **Application Config:**
```yaml
# Application Configuration
app:
  name: MyApp
  version: 1.0.0
  debug: false
  
  database:
    host: localhost
    port: 5432
    name: myapp_db
    
  cache:
    enabled: true
    ttl: 3600  # seconds
```

### **Docker Compose:**
```yaml
version: "3.8"

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    environment:
      - DEBUG=false
```

### **CI/CD Pipeline:**
```yaml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the standard indentation for YAML?**
   <details>
   <summary>Answer</summary>
   2 spaces per level
   </details>

2. **Can you use tabs for indentation?**
   <details>
   <summary>Answer</summary>
   No! Only spaces allowed
   </details>

3. **What character starts a comment?**
   <details>
   <summary>Answer</summary>
   # (hash/pound symbol)
   </details>

4. **What does `enabled: yes` become?**
   <details>
   <summary>Answer</summary>
   Boolean true (not the string "yes")
   </details>

5. **When must you use quotes around a string?**
   <details>
   <summary>Answer</summary>
   When it contains special characters, looks like a number/boolean, or starts with special chars
   </details>

---

## 📝 Hands-on Exercise

**Create a server configuration file:**

Create `server.yaml`:
```yaml
# Web Server Configuration

server:
  name: production-web-01
  ip: 192.168.1.100
  port: 8080
  ssl: true
  
  # Performance settings
  performance:
    max_connections: 1000
    timeout: 30
    keepalive: true
  
  # Logging
  logging:
    level: info      # debug, info, warn, error
    path: /var/log/app
    rotate: true

# Add your own sections below
```

**Tasks:**
1. Validate the file at yamllint.com
2. Add a `database` section with host, port, name
3. Try removing indentation - see the error
4. Add comments explaining each section

---

## ⚠️ Common Mistakes

### **1. Missing Space After Colon:**
```yaml
# ❌ WRONG
name:value

# ✅ CORRECT
name: value
```

### **2. Inconsistent Indentation:**
```yaml
# ❌ WRONG
parent:
  child1: value
   child2: value  # 3 spaces instead of 2

# ✅ CORRECT
parent:
  child1: value
  child2: value
```

### **3. Not Quoting Special Values:**
```yaml
# ❌ WRONG - becomes boolean
answer: yes

# ✅ CORRECT - stays as string
answer: "yes"
```

### **4. Using Tabs:**
```yaml
# ❌ WRONG - tabs are invisible but will cause errors
name:
→→value

# ✅ CORRECT - use spaces
name:
  value
```

---

## 📝 Key Takeaways

✅ **Use 2 spaces for indentation (NO TABS)**  
✅ **Format: `key: value` with space after colon**  
✅ **Comments start with #**  
✅ **YAML infers types automatically**  
✅ **Quote strings when they look like numbers/booleans**  
✅ **Indentation shows structure (parent-child)**  
✅ **Case sensitive**  

---

## 🚀 Next Steps

**Next lesson:** [03 - Data Types](03-data-types.md) - Deep dive into strings, numbers, and booleans

---

## 💡 Pro Tips

**Editor Setup:**
- Set tab key to insert 2 spaces
- Enable "Show Whitespace" to see spaces vs tabs
- Use YAML syntax highlighting
- Install YAML linter extension

**Debugging:**
```bash
# Check for tabs
cat -A file.yaml | grep "^I"

# Validate YAML
yamllint file.yaml
```

**Best Practices:**
- Start simple, add complexity gradually
- Comment non-obvious configurations
- Use consistent naming (snake_case or camelCase)
- Keep indentation consistent throughout

Master the basics, avoid the gotchas! 📏
