---
render_with_liquid: false
---
# 03 - Data Types in YAML

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- All YAML data types
- Type inference rules
- How to force specific types
- Type conversion gotchas
- Best practices for each type

---

## 📊 YAML Data Types Overview

YAML supports these scalar (single value) types:
1. **Strings** - Text data
2. **Numbers** - Integers and floats
3. **Booleans** - True/false values
4. **Null** - Empty/undefined values
5. **Dates/Timestamps** - ISO 8601 format

**YAML automatically infers the type** based on the value!

---

## 🔤 Strings

### **Unquoted Strings (Most Common):**
```yaml
name: John Doe
city: New York
message: Hello World
description: This is a simple string
```

### **Quoted Strings:**

**Single Quotes (Literal):**
```yaml
message: 'This is a string'
path: 'C:\Users\John'     # No escaping needed
special: 'He said "hi"'   # Can contain double quotes
literal: 'Line breaks\ndon''t work'  # \n is literal
```

**Double Quotes (Escape Sequences Work):**
```yaml
message: "Hello\nWorld"    # Newline works
tab: "Col1\tCol2"          # Tab works
quote: "He said \"hi\""    # Escape double quotes
unicode: "\u0041"          # Unicode (A)
```

**When You MUST Use Quotes:**
```yaml
# These become booleans without quotes
string_yes: "yes"
string_no: "no"
string_true: "true"
string_false: "false"
string_on: "on"
string_off: "off"

# These become numbers without quotes
version: "1.20"
id: "007"
zip_code: "02134"

# Special characters require quotes
email: "user@domain.com"
url: "http://example.com"
colon: "key: value"
hash: "#tag"
asterisk: "*important"
ampersand: "Q&A"
```

### **Empty Strings:**
```yaml
# All represent empty string
empty1: ""
empty2: ''
# empty3:  # This is NULL, not empty string!
```

---

## 🔢 Numbers

### **Integers:**
```yaml
age: 30
count: 1000
negative: -5
zero: 0
```

### **Floating Point:**
```yaml
price: 19.99
rate: 0.05
negative: -3.14
scientific: 1.5e10    # 15000000000
```

### **Different Number Formats:**
```yaml
# Decimal (default)
decimal: 123

# Hexadecimal (base 16)
hex: 0x7B      # 123 in decimal

# Octal (base 8)
octal: 0o173   # 123 in decimal

# Binary (base 2)
binary: 0b1111011  # 123 in decimal

# Scientific notation
large: 1.23e6   # 1230000
small: 1.23e-6  # 0.00000123
```

### **Special Number Values:**
```yaml
# Infinity
positive_infinity: .inf
negative_infinity: -.inf

# Not a number
not_a_number: .nan
```

### **Number as String (Force String Type):**
```yaml
# Without quotes - becomes number
version: 3.8        # float: 3.8

# With quotes - stays string
version: "3.8"      # string: "3.8"

# Leading zeros preserved as string
zip: "02134"        # string: "02134"
id: "007"           # string: "007"
```

---

## ✅ Booleans

### **True Values:**
```yaml
enabled: true
active: TRUE
debug: True
legacy_yes: yes
legacy_y: y
legacy_on: on
```

### **False Values:**
```yaml
disabled: false
inactive: FALSE
production: False
legacy_no: no
legacy_n: n
legacy_off: off
```

**⚠️ IMPORTANT:**
```yaml
# These are BOOLEANS
answer: yes        # true
enabled: on        # true
active: off        # false

# These are STRINGS
answer: "yes"      # "yes"
enabled: "on"      # "on"
active: "off"      # "off"
```

**Best Practice:**
```yaml
# ✅ Use explicit true/false (clearest)
debug: true
production: false

# ⚠️ Avoid yes/no (confusing for beginners)
debug: yes
production: no
```

---

## ❌ Null Values

### **Ways to Represent Null:**
```yaml
# All of these are null/undefined
empty_value:
null_value: null
tilde_value: ~
explicit_null: Null
```

### **Null vs Empty String:**
```yaml
# NULL (undefined)
value1:
value2: null
value3: ~

# EMPTY STRING (defined but empty)
value4: ""
value5: ''
```

### **Real-World Usage:**
```yaml
database:
  host: localhost
  port: 5432
  password:  # null - use default or environment variable
```

---

## 📅 Dates and Timestamps

### **ISO 8601 Format:**
```yaml
# Date only
date: 2024-03-15

# Date and time
datetime: 2024-03-15T14:30:00

# With timezone
datetime_utc: 2024-03-15T14:30:00Z
datetime_offset: 2024-03-15T14:30:00+05:30

# Unix timestamp (number)
timestamp: 1710511800
```

### **Canonical Format:**
```yaml
# YAML standard date format
canonical: 2024-03-15
```

---

## 🎭 Type Coercion Examples

### **What Type Will It Be?**

```yaml
# STRINGS
name: John
description: Hello World

# INTEGERS
age: 30
count: -5

# FLOATS
price: 19.99
rate: 3.14

# BOOLEANS
enabled: true
active: yes
disabled: false

# NULLS
empty:
nothing: null

# DATES
created: 2024-03-15

# STRINGS (quoted to prevent coercion)
version: "1.20"      # String, not float
zip: "02134"         # String, not integer
answer: "yes"        # String, not boolean
id: "null"           # String, not null
```

---

## 🔧 Forcing Specific Types (Advanced)

### **Explicit Type Tags:**
```yaml
# Force string
number_as_string: !!str 123
boolean_as_string: !!str true

# Force integer
string_as_int: !!int "123"

# Force float
int_as_float: !!float 42

# Force boolean
string_as_bool: !!bool "yes"
```

**When to Use:**
- Rarely needed in practice
- Most tools understand context
- Quotes usually sufficient for strings

---

## 📋 Real-World Examples

### **Application Configuration:**
```yaml
app:
  name: MyApp                    # String
  version: "1.2.0"               # String (quoted)
  port: 8080                     # Integer
  debug: false                   # Boolean
  
  database:
    host: localhost              # String
    port: 5432                   # Integer
    timeout: 30.5                # Float (seconds)
    pool_size: 10                # Integer
    password:                    # Null
  
  features:
    cache: true                  # Boolean
    logging: on                  # Boolean (legacy)
    backup: off                  # Boolean (legacy)
  
  limits:
    max_size: 1.5e9              # Float (scientific)
    infinity: .inf               # Infinity
```

### **Kubernetes Pod:**
```yaml
apiVersion: v1                   # String
kind: Pod                        # String
metadata:
  name: nginx-pod                # String
  labels:
    app: nginx                   # String
    version: "1.21"              # String (quoted!)
spec:
  containers:
  - name: nginx                  # String
    image: nginx:1.21            # String
    ports:
    - containerPort: 80          # Integer
    resources:
      limits:
        memory: "512Mi"          # String
        cpu: "500m"              # String
      requests:
        memory: "256Mi"          # String
        cpu: "250m"              # String
```

---

## 🎯 Quick Check: Do You Understand?

1. **What type is `age: 30`?**
   <details>
   <summary>Answer</summary>
   Integer
   </details>

2. **What type is `version: 1.20`?**
   <details>
   <summary>Answer</summary>
   Float (number 1.2)
   </details>

3. **How to make version stay as "1.20"?**
   <details>
   <summary>Answer</summary>
   Quote it: version: "1.20"
   </details>

4. **What type is `enabled: yes`?**
   <details>
   <summary>Answer</summary>
   Boolean (true)
   </details>

5. **What's the difference between `value:` and `value: ""`?**
   <details>
   <summary>Answer</summary>
   First is null, second is empty string
   </details>

---

## 📝 Hands-on Exercise

**Create a product catalog:**

Create `products.yaml`:
```yaml
# Product Catalog

products:
  - id: "001"                    # String (preserve leading zero)
    name: Laptop
    price: 999.99                # Float
    in_stock: true               # Boolean
    quantity: 25                 # Integer
    rating: 4.5                  # Float
    discount:                    # Null (no discount)
    
  - id: "002"
    name: Mouse
    price: 19.99
    in_stock: yes                # Boolean (legacy)
    quantity: 100
    rating: 4.0
    discount: 0.1                # Float (10% off)
    
  - id: "010"                    # String (quoted)
    name: Keyboard
    price: 49.99
    in_stock: false
    quantity: 0
    rating: 4.2
    discount: null               # Explicitly null

# Add more products
```

**Tasks:**
1. Validate the YAML
2. Identify the type of each field
3. Add a product with version "2.0" (must stay string)
4. Add a product with price in scientific notation

---

## ⚠️ Common Type Mistakes

### **1. Version Numbers:**
```yaml
# ❌ WRONG - becomes float 1.2
version: 1.20

# ✅ CORRECT - stays "1.20"
version: "1.20"
```

### **2. Leading Zeros:**
```yaml
# ❌ WRONG - becomes integer 123
zip_code: 0123

# ✅ CORRECT - stays "0123"
zip_code: "0123"
```

### **3. Yes/No Answers:**
```yaml
# ❌ WRONG - becomes boolean true
answer: yes

# ✅ CORRECT - stays "yes"
answer: "yes"
```

### **4. Null vs Empty:**
```yaml
# NULL
password:

# EMPTY STRING
password: ""

# Be explicit if it matters
password: null  # or ""
```

---

## 📝 Key Takeaways

✅ **YAML automatically infers types**  
✅ **Quote strings that look like numbers/booleans**  
✅ **Use true/false instead of yes/no for clarity**  
✅ **Empty value (`:`) is null, not empty string**  
✅ **Preserve leading zeros with quotes**  
✅ **Version numbers should be quoted**  
✅ **Dates use ISO 8601 format**  

---

## 🔍 Type Inference Cheat Sheet

| Value | Type | Notes |
|-------|------|-------|
| `Hello` | String | |
| `123` | Integer | |
| `1.23` | Float | |
| `true` | Boolean | |
| `yes` | Boolean | Legacy |
| `on` | Boolean | Legacy |
| `:` | Null | Empty value |
| `null` | Null | Explicit |
| `~` | Null | Tilde |
| `"123"` | String | Quoted |
| `"true"` | String | Quoted |
| `2024-03-15` | Date | ISO format |

---

## 🚀 Next Steps

**Next lesson:** [04 - Lists and Arrays](04-lists-arrays.md) - Working with collections

---

## 💡 Pro Tips

**Type Safety:**
```yaml
# When in doubt, quote it
version: "1.20"
id: "007"
answer: "yes"
```

**Validation:**
```bash
# Check types with Python
python3 << EOF
import yaml
with open('file.yaml') as f:
    data = yaml.safe_load(f)
    for key, value in data.items():
        print(f"{key}: {type(value).__name__} = {value}")
EOF
```

**Best Practices:**
- Use explicit `true`/`false` for booleans
- Quote version numbers and IDs
- Be explicit with null when it matters
- Document expected types in comments

Know your types, avoid surprises! 🎯
