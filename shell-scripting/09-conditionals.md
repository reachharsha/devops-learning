---
render_with_liquid: false
---
# 09 - Conditionals

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- if/elif/else statements
- Test operators and conditions
- File tests (checking if files exist, permissions, etc.)
- String comparisons
- Numeric comparisons
- Logical operators (AND, OR, NOT)
- case statements
- The difference between [ ], [[ ]], and (( ))

---

## 🔀 The if Statement

### **Basic Syntax:**

```bash
#!/bin/bash

# Simple if
if [ condition ]; then
    # commands
fi

# if-else
if [ condition ]; then
    # commands if true
else
    # commands if false
fi

# if-elif-else
if [ condition1 ]; then
    # commands if condition1 is true
elif [ condition2 ]; then
    # commands if condition2 is true
else
    # commands if all conditions are false
fi
```

### **Real Examples:**

```bash
#!/bin/bash

# Check if file exists
if [ -f "/etc/passwd" ]; then
    echo "File exists"
fi

# Check with else
FILE="data.txt"
if [ -f "$FILE" ]; then
    echo "File $FILE exists"
else
    echo "File $FILE not found"
fi

# Multiple conditions
AGE=25
if [ $AGE -lt 18 ]; then
    echo "Minor"
elif [ $AGE -lt 65 ]; then
    echo "Adult"
else
    echo "Senior"
fi
```

---

## 📋 File Test Operators

### **Common File Tests:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `-e file` | File exists | `[ -e file.txt ]` |
| `-f file` | Regular file exists | `[ -f file.txt ]` |
| `-d dir` | Directory exists | `[ -d /home ]` |
| `-r file` | File is readable | `[ -r file.txt ]` |
| `-w file` | File is writable | `[ -w file.txt ]` |
| `-x file` | File is executable | `[ -x script.sh ]` |
| `-s file` | File exists and not empty | `[ -s file.txt ]` |
| `-L file` | File is symbolic link | `[ -L link ]` |
| `file1 -nt file2` | file1 newer than file2 | `[ f1 -nt f2 ]` |
| `file1 -ot file2` | file1 older than file2 | `[ f1 -ot f2 ]` |

### **Practical Examples:**

```bash
#!/bin/bash

# Check if file exists before reading
CONFIG_FILE="/etc/app.conf"
if [ -f "$CONFIG_FILE" ]; then
    echo "Loading configuration..."
    source "$CONFIG_FILE"
else
    echo "Error: Configuration file not found!"
    exit 1
fi

# Check if directory exists before creating
BACKUP_DIR="/backups"
if [ ! -d "$BACKUP_DIR" ]; then
    echo "Creating backup directory..."
    mkdir -p "$BACKUP_DIR"
fi

# Check if file is executable
SCRIPT="deploy.sh"
if [ -x "$SCRIPT" ]; then
    ./"$SCRIPT"
else
    echo "Error: $SCRIPT is not executable"
    echo "Run: chmod +x $SCRIPT"
fi

# Check if file is empty
LOG_FILE="app.log"
if [ -s "$LOG_FILE" ]; then
    echo "Log file has content ($(wc -l < "$LOG_FILE") lines)"
else
    echo "Log file is empty or doesn't exist"
fi

# Check file age
CACHE_FILE="cache.tmp"
REFERENCE_FILE="source.txt"

if [ -f "$CACHE_FILE" ] && [ "$CACHE_FILE" -nt "$REFERENCE_FILE" ]; then
    echo "Using cached data"
    cat "$CACHE_FILE"
else
    echo "Regenerating cache..."
    # Generate new cache
fi
```

---

## 🔤 String Comparisons

### **String Operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` or `==` | Equal | `[ "$a" = "$b" ]` |
| `!=` | Not equal | `[ "$a" != "$b" ]` |
| `<` | Less than (alphabetically) | `[[ "$a" < "$b" ]]` |
| `>` | Greater than | `[[ "$a" > "$b" ]]` |
| `-z string` | String is empty | `[ -z "$var" ]` |
| `-n string` | String is not empty | `[ -n "$var" ]` |

### **Examples:**

```bash
#!/bin/bash

# Check if strings are equal
NAME="John"
if [ "$NAME" = "John" ]; then
    echo "Hello, John!"
fi

# Check if string is empty
read -p "Enter your name: " INPUT
if [ -z "$INPUT" ]; then
    echo "Error: Name cannot be empty"
    exit 1
fi

# Check if string is NOT empty
PASSWORD="secret"
if [ -n "$PASSWORD" ]; then
    echo "Password is set"
fi

# String inequality
ENV="production"
if [ "$ENV" != "development" ]; then
    echo "Running in production mode"
fi

# Alphabetical comparison (use [[ ]])
FILE1="apple.txt"
FILE2="banana.txt"
if [[ "$FILE1" < "$FILE2" ]]; then
    echo "$FILE1 comes before $FILE2 alphabetically"
fi

# Pattern matching (use [[ ]])
FILENAME="report.pdf"
if [[ "$FILENAME" == *.pdf ]]; then
    echo "This is a PDF file"
fi

# Check multiple values
ACTION="start"
if [ "$ACTION" = "start" ] || [ "$ACTION" = "restart" ]; then
    echo "Starting service..."
fi
```

---

## 🔢 Numeric Comparisons

### **Numeric Operators:**

| Operator | Meaning | Example |
|----------|---------|---------|
| `-eq` | Equal | `[ $a -eq $b ]` |
| `-ne` | Not equal | `[ $a -ne $b ]` |
| `-lt` | Less than | `[ $a -lt $b ]` |
| `-le` | Less than or equal | `[ $a -le $b ]` |
| `-gt` | Greater than | `[ $a -gt $b ]` |
| `-ge` | Greater than or equal | `[ $a -ge $b ]` |

### **Examples:**

```bash
#!/bin/bash

# Age validation
AGE=25
if [ $AGE -ge 18 ]; then
    echo "You are an adult"
fi

# Range check
SCORE=85
if [ $SCORE -ge 90 ]; then
    echo "Grade: A"
elif [ $SCORE -ge 80 ]; then
    echo "Grade: B"
elif [ $SCORE -ge 70 ]; then
    echo "Grade: C"
else
    echo "Grade: F"
fi

# Check boundaries
PORT=8080
if [ $PORT -lt 1024 ]; then
    echo "Warning: Privileged port (requires root)"
elif [ $PORT -gt 65535 ]; then
    echo "Error: Invalid port number"
else
    echo "Port $PORT is valid"
fi

# Disk space check
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ $DISK_USAGE -gt 90 ]; then
    echo "CRITICAL: Disk usage is ${DISK_USAGE}%"
elif [ $DISK_USAGE -gt 80 ]; then
    echo "WARNING: Disk usage is ${DISK_USAGE}%"
else
    echo "Disk usage OK: ${DISK_USAGE}%"
fi
```

---

## 🔗 Logical Operators

### **AND, OR, NOT:**

```bash
#!/bin/bash

# AND with -a or &&
AGE=25
COUNTRY="USA"

# Method 1: -a inside [ ]
if [ $AGE -ge 18 -a "$COUNTRY" = "USA" ]; then
    echo "Can vote in USA"
fi

# Method 2: && between [ ] (preferred)
if [ $AGE -ge 18 ] && [ "$COUNTRY" = "USA" ]; then
    echo "Can vote in USA"
fi

# Method 3: && inside [[ ]] (best)
if [[ $AGE -ge 18 && "$COUNTRY" = "USA" ]]; then
    echo "Can vote in USA"
fi

# OR with -o or ||
DAY="Saturday"
if [ "$DAY" = "Saturday" ] || [ "$DAY" = "Sunday" ]; then
    echo "It's the weekend!"
fi

# NOT with !
if [ ! -f "config.txt" ]; then
    echo "Config file not found"
fi

# Complex conditions
USERNAME="admin"
PASSWORD="secret"
if [[ "$USERNAME" = "admin" && "$PASSWORD" = "secret" ]]; then
    echo "Access granted"
else
    echo "Access denied"
fi

# Multiple conditions
FILE="data.txt"
if [[ -f "$FILE" && -r "$FILE" && -s "$FILE" ]]; then
    echo "File exists, is readable, and not empty"
    cat "$FILE"
fi
```

---

## 🎨 [ ] vs [[ ]] vs (( ))

### **Single Brackets [ ]:**

```bash
#!/bin/bash

# POSIX compliant, works in all shells
# Must quote variables
# Use -a and -o for AND/OR

NAME="John"
if [ "$NAME" = "John" ]; then
    echo "Hello, John"
fi

# Limitations:
# [ $NAME = John ]        # ❌ Error if NAME has spaces
# [ "$NAME" = John ]      # ✅ Must quote
```

### **Double Brackets [[ ]]:**

```bash
#!/bin/bash

# Bash-specific, more features
# No word splitting (safer)
# Supports pattern matching
# Better operators (&& || < >)

NAME="John Doe"
if [[ $NAME = "John Doe" ]]; then    # No quotes needed
    echo "Hello, John"
fi

# Pattern matching
FILE="report.pdf"
if [[ $FILE == *.pdf ]]; then
    echo "PDF file"
fi

# Regex matching
EMAIL="user@example.com"
if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email format"
fi

# Cleaner logic
if [[ $AGE -ge 18 && $COUNTRY = "USA" ]]; then
    echo "Can vote"
fi
```

### **Double Parentheses (( )):**

```bash
#!/bin/bash

# Arithmetic evaluation
# C-style syntax
# No $ needed for variables

COUNT=10
if (( COUNT > 5 )); then
    echo "Count is greater than 5"
fi

# Arithmetic operations
if (( COUNT % 2 == 0 )); then
    echo "Count is even"
fi

# C-style operators
if (( COUNT >= 10 && COUNT <= 20 )); then
    echo "Count is between 10 and 20"
fi

# Increment/decrement
if (( ++COUNT > 10 )); then
    echo "After increment: $COUNT"
fi
```

### **When to Use What:**

```bash
#!/bin/bash

# File tests: use [ ] or [[ ]]
if [ -f "file.txt" ]; then          # ✅ Good
if [[ -f "file.txt" ]]; then        # ✅ Also good

# String comparisons: prefer [[ ]]
if [[ "$name" = "John" ]]; then     # ✅ Best (no word splitting)
if [ "$name" = "John" ]; then       # ✅ OK (must quote)

# Pattern matching: use [[ ]]
if [[ $file == *.txt ]]; then       # ✅ Only [[ ]] supports this

# Numeric comparisons: use (( )) or [ ]
if (( age > 18 )); then             # ✅ Clean syntax
if [ $age -gt 18 ]; then            # ✅ Also works

# Complex logic: use [[ ]] or (( ))
if [[ $a -gt 5 && $b -lt 10 ]]; then    # ✅ Readable
if (( a > 5 && b < 10 )); then          # ✅ C-style
```

---

## 🎯 case Statements

### **Basic Syntax:**

```bash
#!/bin/bash

case $VARIABLE in
    pattern1)
        # commands
        ;;
    pattern2)
        # commands
        ;;
    *)
        # default case
        ;;
esac
```

### **Practical Examples:**

```bash
#!/bin/bash

# Simple menu
read -p "Enter command (start|stop|restart): " ACTION

case $ACTION in
    start)
        echo "Starting service..."
        ;;
    stop)
        echo "Stopping service..."
        ;;
    restart)
        echo "Restarting service..."
        ;;
    *)
        echo "Invalid command"
        exit 1
        ;;
esac

# Multiple patterns
DAY=$(date +%A)
case $DAY in
    Monday|Tuesday|Wednesday|Thursday|Friday)
        echo "It's a weekday"
        ;;
    Saturday|Sunday)
        echo "It's the weekend!"
        ;;
esac

# Pattern matching
FILE="document.pdf"
case $FILE in
    *.txt)
        echo "Text file"
        ;;
    *.pdf)
        echo "PDF document"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        ;;
    *)
        echo "Unknown file type"
        ;;
esac

# File type handler
handle_file() {
    local file=$1
    
    case $file in
        *.tar.gz|*.tgz)
            tar -xzf "$file"
            ;;
        *.zip)
            unzip "$file"
            ;;
        *.tar.bz2)
            tar -xjf "$file"
            ;;
        *)
            echo "Unsupported archive format"
            return 1
            ;;
    esac
}
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between = and -eq?**
   <details>
   <summary>Answer</summary>
   = is for string comparison, -eq is for numeric comparison
   </details>

2. **How do you check if a file exists?**
   <details>
   <summary>Answer</summary>
   [ -f "filename" ] or [ -e "filename" ] (-f for regular file, -e for any file)
   </details>

3. **What's better: [ ] or [[ ]]?**
   <details>
   <summary>Answer</summary>
   [[ ]] is better for bash scripts (safer, more features, pattern matching)
   [ ] is better for POSIX compliance
   </details>

4. **How do you negate a condition?**
   <details>
   <summary>Answer</summary>
   Use ! (NOT operator): if [ ! -f "file" ]; then ... or if [[ ! $var = "value" ]]
   </details>

5. **When should you use case instead of if?**
   <details>
   <summary>Answer</summary>
   When checking a single variable against multiple specific values (cleaner and more readable)
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: File Validator**

```bash
#!/bin/bash
# file_validator.sh

if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

FILE=$1

echo "Checking file: $FILE"
echo "---"

if [ ! -e "$FILE" ]; then
    echo "❌ File does not exist"
    exit 1
fi

if [ -d "$FILE" ]; then
    echo "📁 This is a directory"
    echo "   Contents: $(ls -1 "$FILE" | wc -l) items"
elif [ -f "$FILE" ]; then
    echo "📄 This is a regular file"
    echo "   Size: $(du -h "$FILE" | cut -f1)"
    
    if [ -r "$FILE" ]; then
        echo "   ✅ Readable"
    else
        echo "   ❌ Not readable"
    fi
    
    if [ -w "$FILE" ]; then
        echo "   ✅ Writable"
    else
        echo "   ❌ Not writable"
    fi
    
    if [ -x "$FILE" ]; then
        echo "   ✅ Executable"
    else
        echo "   ❌ Not executable"
    fi
    
    if [ -s "$FILE" ]; then
        echo "   ✅ File has content"
    else
        echo "   ⚠️  File is empty"
    fi
fi
```

### **Exercise 2: User Age Classifier**

```bash
#!/bin/bash
# age_classifier.sh

read -p "Enter your age: " AGE

# Validate input is a number
if ! [[ "$AGE" =~ ^[0-9]+$ ]]; then
    echo "Error: Please enter a valid number"
    exit 1
fi

# Classify by age
if [ $AGE -lt 0 ]; then
    echo "Error: Age cannot be negative"
elif [ $AGE -lt 13 ]; then
    echo "You are a child"
elif [ $AGE -lt 20 ]; then
    echo "You are a teenager"
elif [ $AGE -lt 65 ]; then
    echo "You are an adult"
else
    echo "You are a senior"
fi

# Additional checks
if [ $AGE -ge 18 ]; then
    echo "✅ You can vote"
    echo "✅ You can drive"
fi

if [ $AGE -ge 21 ]; then
    echo "✅ You can drink alcohol (in USA)"
fi
```

### **Exercise 3: System Health Check**

```bash
#!/bin/bash
# health_check.sh

echo "System Health Check"
echo "==================="

# Check disk usage
DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')

if [ $DISK_USAGE -gt 90 ]; then
    echo "🔴 CRITICAL: Disk usage is ${DISK_USAGE}%"
elif [ $DISK_USAGE -gt 80 ]; then
    echo "🟡 WARNING: Disk usage is ${DISK_USAGE}%"
else
    echo "🟢 OK: Disk usage is ${DISK_USAGE}%"
fi

# Check memory
MEM_FREE=$(free | grep Mem | awk '{print ($4/$2) * 100}' | cut -d. -f1)

if [ $MEM_FREE -lt 10 ]; then
    echo "🔴 CRITICAL: Only ${MEM_FREE}% memory free"
elif [ $MEM_FREE -lt 20 ]; then
    echo "🟡 WARNING: Only ${MEM_FREE}% memory free"
else
    echo "🟢 OK: ${MEM_FREE}% memory free"
fi

# Check critical services
SERVICES=("sshd" "cron")

for service in "${SERVICES[@]}"; do
    if systemctl is-active --quiet "$service" 2>/dev/null; then
        echo "🟢 Service $service is running"
    else
        echo "🔴 Service $service is NOT running"
    fi
done
```

### **Exercise 4: File Type Handler**

```bash
#!/bin/bash
# file_handler.sh

if [ $# -eq 0 ]; then
    echo "Usage: $0 <filename>"
    exit 1
fi

FILE=$1

if [ ! -f "$FILE" ]; then
    echo "Error: File not found"
    exit 1
fi

case $FILE in
    *.txt|*.md)
        echo "Text file - displaying content:"
        cat "$FILE"
        ;;
    *.jpg|*.png|*.gif)
        echo "Image file"
        echo "Size: $(du -h "$FILE" | cut -f1)"
        if command -v identify &> /dev/null; then
            identify "$FILE"
        fi
        ;;
    *.tar.gz|*.tgz)
        echo "Tar+Gzip archive"
        echo "Contents:"
        tar -tzf "$FILE"
        ;;
    *.zip)
        echo "ZIP archive"
        echo "Contents:"
        unzip -l "$FILE"
        ;;
    *.sh)
        echo "Shell script"
        if [ -x "$FILE" ]; then
            echo "✅ Executable"
        else
            echo "❌ Not executable - run: chmod +x $FILE"
        fi
        ;;
    *)
        echo "Unknown file type"
        file "$FILE"
        ;;
esac
```

---

## 📝 Key Takeaways

✅ **Use [[ ]] for bash scripts (safer, more features)**  
✅ **Use (( )) for arithmetic comparisons**  
✅ **Always quote variables in [ ]: [ "$var" = "value" ]**  
✅ **File tests: -f (file), -d (dir), -e (exists), -r (readable)**  
✅ **String tests: = (equal), != (not equal), -z (empty), -n (not empty)**  
✅ **Numeric tests: -eq, -ne, -lt, -le, -gt, -ge**  
✅ **Logical operators: && (AND), || (OR), ! (NOT)**  
✅ **Use case for checking multiple specific values**  
✅ **Negate with !: if [ ! -f "file" ]**  

---

## 🚀 Next Steps

You now know how to make decisions in your scripts!

**Next lesson:** [10 - Loops](10-loops.md) - Repeating actions efficiently

---

## 💡 Pro Tips

**Always validate input:**
```bash
if [ $# -eq 0 ]; then
    echo "Usage: $0 <arg>"
    exit 1
fi
```

**Check before operating:**
```bash
if [ -f "$FILE" ]; then
    rm "$FILE"
fi
```

**Use pattern matching in [[ ]]:**
```bash
if [[ $FILENAME == *.txt ]]; then
    # Process text file
fi
```

**Readable complex conditions:**
```bash
if [[ $AGE -ge 18 && $COUNTRY = "USA" && $REGISTERED = true ]]; then
    echo "Can vote"
fi
```

**Short-circuit evaluation:**
```bash
# Only runs second command if first succeeds
[ -f "$FILE" ] && cat "$FILE"

# Only runs second command if first fails
[ -f "$FILE" ] || echo "File not found"
```

Conditions are the brain of your scripts! 🧠
