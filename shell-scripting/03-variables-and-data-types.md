# 03 - Variables & Data Types

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Variable declaration and naming conventions
- Different types of variables (local, global, environment)
- Special bash variables ($?, $$, $!, etc.)
- Command substitution and capturing output
- Arithmetic operations
- Working with strings and numbers

---

## 📦 What are Variables?

**Real-World Analogy:**  
Think of variables as labeled boxes where you store information. You can put things in the box, look at what's inside, and even change the contents.

```bash
# Create a variable (no spaces around =!)
NAME="John"

# Use the variable
echo "Hello, $NAME"
# Output: Hello, John

echo "Hello, ${NAME}"   # Same thing (safer with curly braces)
# Output: Hello, John
```

---

## 🎨 Variable Declaration

### **Basic Syntax:**

```bash
#!/bin/bash

# String variables
NAME="Alice"
CITY="New York"
MESSAGE='Hello World'    # Single quotes: literal string

# Numbers (stored as strings but can be used in math)
AGE=25
COUNT=100

# Using variables
echo "Name: $NAME"
echo "City: $CITY"
echo "Age: $AGE"

# Curly braces (recommended for clarity)
echo "Hello, ${NAME}!"           # ✅ Good
echo "File: ${NAME}_backup.txt"  # ✅ Necessary here
# Without braces: $NAME_backup.txt would look for variable NAME_backup
```

### **Important Rules:**

```bash
# ✅ CORRECT
NAME="John"          # No spaces around =
AGE=25               # No $ when assigning
MY_VAR="value"       # Underscores allowed

# ❌ WRONG
NAME = "John"        # Error: spaces around =
$NAME="John"         # Error: $ when assigning
MY-VAR="value"       # Error: hyphens not allowed
2NAME="John"         # Error: can't start with number
```

### **Naming Conventions:**

```bash
#!/bin/bash

# 1. Lowercase for local variables
name="value"
temp_file="/tmp/data"

# 2. UPPERCASE for environment variables and constants
export DATABASE_URL="postgresql://localhost/db"
readonly MAX_RETRIES=3

# 3. Descriptive names
user_count=10              # ✅ Good
uc=10                      # ❌ Bad

# 4. Use underscores for readability
backup_directory="/backups"    # ✅ Good
backupdirectory="/backups"     # ❌ Hard to read

# 5. Avoid system variable names
PATH="/custom/path"        # ❌ Bad: overwrites system PATH
MY_PATH="/custom/path"     # ✅ Good
```

---

## 🌐 Types of Variables

### **1. Local Variables (Function Scope):**

```bash
#!/bin/bash

my_function() {
    local LOCAL_VAR="I'm local"    # Only exists in function
    GLOBAL_VAR="I'm global"         # Exists outside too
    
    echo "Inside function:"
    echo "  Local: $LOCAL_VAR"
    echo "  Global: $GLOBAL_VAR"
}

my_function

echo "Outside function:"
echo "  Local: $LOCAL_VAR"          # Empty (doesn't exist)
echo "  Global: $GLOBAL_VAR"        # Still accessible
```

**Output:**
```
Inside function:
  Local: I'm local
  Global: I'm global
Outside function:
  Local: 
  Global: I'm global
```

### **2. Environment Variables (Exported):**

```bash
#!/bin/bash

# Regular variable (only in current script)
REGULAR_VAR="Hello"

# Environment variable (available to child processes)
export ENV_VAR="World"

# Create child script
cat > child.sh << 'EOF'
#!/bin/bash
echo "Regular: $REGULAR_VAR"    # Empty
echo "Environment: $ENV_VAR"     # Has value
EOF

chmod +x child.sh
./child.sh

# Output:
# Regular: 
# Environment: World
```

### **3. Readonly Variables (Constants):**

```bash
#!/bin/bash

# Declare constant
readonly PI=3.14159
readonly MAX_USERS=100

echo "PI: $PI"

# Try to change (fails)
PI=3.14
# Error: PI: readonly variable

# Alternative syntax
declare -r DATABASE_NAME="production"
```

### **4. Integer Variables:**

```bash
#!/bin/bash

# Declare as integer
declare -i NUMBER=10

NUMBER=NUMBER+5    # Arithmetic without $
echo $NUMBER       # Output: 15

NUMBER="Hello"     # Tries to convert to number
echo $NUMBER       # Output: 0 (failed conversion)
```

---

## ⭐ Special Variables

### **Positional Parameters:**

```bash
#!/bin/bash
# script.sh

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"
echo "All arguments: $@"
echo "All arguments (single string): $*"
echo "Number of arguments: $#"
```

**Run it:**
```bash
./script.sh apple banana cherry

# Output:
# Script name: ./script.sh
# First argument: apple
# Second argument: banana
# Third argument: cherry
# All arguments: apple banana cherry
# All arguments (single string): apple banana cherry
# Number of arguments: 3
```

### **Process Variables:**

```bash
#!/bin/bash

echo "Current PID: $$"                    # Script's process ID
echo "Parent PID: $PPID"                  # Parent process ID

# Start background process
sleep 10 &
echo "Background PID: $!"                 # Last background process PID

# Get its status
wait $!
echo "Exit status: $?"                    # Exit status of last command
```

### **Exit Status ($?):**

```bash
#!/bin/bash

# Success
ls /etc/passwd
echo "Exit status: $?"    # 0 (success)

# Failure
ls /nonexistent
echo "Exit status: $?"    # Non-zero (failure, usually 1 or 2)

# Use in conditions
if ls /etc/passwd > /dev/null 2>&1; then
    echo "File exists"
fi

# Check immediately after command
grep "root" /etc/passwd
if [ $? -eq 0 ]; then
    echo "Found root user"
else
    echo "Not found"
fi
```

### **Complete Special Variables Reference:**

| Variable | Meaning | Example |
|----------|---------|---------|
| `$0` | Script name | `./myscript.sh` |
| `$1, $2, ...` | Positional arguments | `$1` = first arg |
| `$#` | Number of arguments | `3` |
| `$@` | All arguments (array) | `"arg1" "arg2" "arg3"` |
| `$*` | All arguments (string) | `"arg1 arg2 arg3"` |
| `$$` | Current process ID | `12345` |
| `$!` | Last background PID | `12346` |
| `$?` | Exit status of last command | `0` or `1-255` |
| `$-` | Current shell options | `himBH` |
| `$_` | Last argument of previous command | `filename.txt` |

**Practical Example:**

```bash
#!/bin/bash
# backup.sh - Backup script with special variables

echo "================================"
echo "Backup Script (PID: $$)"
echo "================================"

# Check arguments
if [ $# -lt 1 ]; then
    echo "Usage: $0 <directory>"
    exit 1
fi

SOURCE_DIR=$1
BACKUP_FILE="backup_$(date +%Y%m%d_%H%M%S).tar.gz"

echo "Source: $SOURCE_DIR"
echo "Backup: $BACKUP_FILE"

# Create backup
tar -czf "$BACKUP_FILE" "$SOURCE_DIR" 2>/dev/null &
BG_PID=$!

echo "Backup in progress (PID: $BG_PID)..."
wait $BG_PID

if [ $? -eq 0 ]; then
    echo "✅ Backup completed successfully!"
else
    echo "❌ Backup failed!"
    exit 1
fi
```

---

## 🔄 Command Substitution

**Capture command output into variables:**

### **Modern Syntax: $( ):**

```bash
#!/bin/bash

# Capture output
CURRENT_DATE=$(date)
echo "Date: $CURRENT_DATE"

USER_COUNT=$(who | wc -l)
echo "Logged in users: $USER_COUNT"

FILES_COUNT=$(ls -1 | wc -l)
echo "Files in directory: $FILES_COUNT"

# Nested substitution
FORMATTED_DATE=$(date -d "$(cat last_backup.txt)" +%Y-%m-%d)

# Use in strings
echo "Backup created on $(date +%Y-%m-%d)"
```

### **Old Syntax: Backticks (Avoid):**

```bash
# ❌ Old way (harder to read, can't nest easily)
CURRENT_DATE=`date`

# ✅ Modern way (recommended)
CURRENT_DATE=$(date)
```

### **Practical Examples:**

```bash
#!/bin/bash

# System information
HOSTNAME=$(hostname)
KERNEL=$(uname -r)
UPTIME=$(uptime -p)
MEMORY=$(free -h | grep Mem | awk '{print $3 "/" $2}')
DISK=$(df -h / | tail -1 | awk '{print $5}')

echo "System: $HOSTNAME"
echo "Kernel: $KERNEL"
echo "Uptime: $UPTIME"
echo "Memory: $MEMORY"
echo "Disk usage: $DISK"

# File operations
BACKUP_DIR="/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

# Count specific files
LOG_COUNT=$(find /var/log -name "*.log" -type f | wc -l)
echo "Log files: $LOG_COUNT"

# Get specific field
IP_ADDRESS=$(ip addr show eth0 | grep "inet " | awk '{print $2}' | cut -d/ -f1)
echo "IP: $IP_ADDRESS"
```

---

## 🔢 Arithmetic Operations

### **Method 1: $(( )) - Arithmetic Expansion:**

```bash
#!/bin/bash

# Basic operations
A=10
B=3

SUM=$((A + B))
DIFF=$((A - B))
PRODUCT=$((A * B))
QUOTIENT=$((A / B))        # Integer division
REMAINDER=$((A % B))
POWER=$((A ** 2))

echo "Sum: $SUM"           # 13
echo "Difference: $DIFF"   # 7
echo "Product: $PRODUCT"   # 30
echo "Quotient: $QUOTIENT" # 3 (not 3.33)
echo "Remainder: $REMAINDER" # 1
echo "Power: $POWER"       # 100

# Increment/Decrement
COUNT=5
((COUNT++))               # Increment
echo $COUNT               # 6

((COUNT--))               # Decrement
echo $COUNT               # 5

((COUNT += 10))           # Add 10
echo $COUNT               # 15

# Use in conditions
if ((COUNT > 10)); then
    echo "Count is greater than 10"
fi

# No $ needed inside (( ))
X=5
Y=10
RESULT=$((X + Y))         # ✅ Correct
RESULT=$(($X + $Y))       # ✅ Also works (redundant $)
```

### **Method 2: let Command:**

```bash
#!/bin/bash

let "A = 5 + 5"
echo $A                   # 10

let A=5+5                 # Same (no quotes needed)
echo $A                   # 10

let A++                   # Increment
let A+=10                 # Add 10

# Multiple operations
let "A = 10" "B = 20" "C = A + B"
echo $C                   # 30
```

### **Method 3: expr (Old Way):**

```bash
#!/bin/bash

# Basic math (spaces required!)
SUM=$(expr 5 + 3)
echo $SUM                 # 8

PRODUCT=$(expr 5 \* 3)    # Escape * to avoid glob expansion
echo $PRODUCT             # 15

# Comparison
expr 5 \> 3               # Returns 1 (true)
expr 5 \< 3               # Returns 0 (false)
```

### **Method 4: bc for Floating Point:**

```bash
#!/bin/bash

# $(( )) only does integer math
echo $((10 / 3))          # 3 (integer)

# bc for decimals
RESULT=$(echo "10 / 3" | bc -l)
echo $RESULT              # 3.33333333333333333333

# Set precision
RESULT=$(echo "scale=2; 10 / 3" | bc)
echo $RESULT              # 3.33

# Complex calculations
PI=$(echo "scale=10; 4*a(1)" | bc -l)  # Calculate pi
echo $PI                  # 3.1415926532

# Practical example
TAX_RATE=0.08
PRICE=100
TAX=$(echo "scale=2; $PRICE * $TAX_RATE" | bc)
TOTAL=$(echo "scale=2; $PRICE + $TAX" | bc)

echo "Price: \$$PRICE"
echo "Tax: \$$TAX"
echo "Total: \$$TOTAL"
```

### **Comparison: Which Method to Use?**

```bash
#!/bin/bash

# Integer math: Use $(( ))
COUNT=$((COUNT + 1))                # ✅ Best for integers

# Floating point: Use bc
AVERAGE=$(echo "scale=2; $TOTAL / $COUNT" | bc)  # ✅ For decimals

# Simple increment/decrement: Use (( ))
((COUNTER++))                       # ✅ Cleanest for increment

# Legacy scripts: expr
SUM=$(expr $A + $B)                 # ⚠️ Old way, avoid in new scripts
```

---

## 🎯 Variable Manipulation Examples

### **Working with Strings:**

```bash
#!/bin/bash

NAME="John Doe"

# Length
echo ${#NAME}             # 8

# Substring (start at position 0, length 4)
echo ${NAME:0:4}          # John

# Substring (start at position 5 to end)
echo ${NAME:5}            # Doe

# Replace
echo ${NAME/Doe/Smith}    # John Smith
```

*(More in [13 - String Manipulation](13-string-manipulation.md))*

### **Default Values:**

```bash
#!/bin/bash

# Use default if variable is unset
echo ${UNDEFINED_VAR:-"default value"}    # default value

# Assign default if unset
echo ${NEW_VAR:="assigned value"}         # assigned value
echo $NEW_VAR                              # assigned value

# Error if unset
echo ${REQUIRED_VAR:?"Error: REQUIRED_VAR not set"}
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you assign a value to a variable?**
   <details>
   <summary>Answer</summary>
   VAR="value" (no spaces around =, no $ when assigning)
   </details>

2. **What's the difference between $@ and $*?**
   <details>
   <summary>Answer</summary>
   $@ treats each argument as separate word ("arg1" "arg2")
   $* treats all arguments as single string ("arg1 arg2")
   </details>

3. **What does $? represent?**
   <details>
   <summary>Answer</summary>
   Exit status of the last executed command (0 = success, non-zero = error)
   </details>

4. **How do you do integer addition?**
   <details>
   <summary>Answer</summary>
   RESULT=$((A + B)) or ((RESULT = A + B)) or let RESULT=A+B
   </details>

5. **How do you capture command output?**
   <details>
   <summary>Answer</summary>
   VAR=$(command) - preferred modern syntax
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Calculator Script**

```bash
#!/bin/bash
# calculator.sh - Simple calculator

echo "Enter first number:"
read NUM1

echo "Enter second number:"
read NUM2

echo "========================="
echo "Calculator Results"
echo "========================="
echo "Sum: $((NUM1 + NUM2))"
echo "Difference: $((NUM1 - NUM2))"
echo "Product: $((NUM1 * NUM2))"
echo "Quotient: $((NUM1 / NUM2))"
echo "Remainder: $((NUM1 % NUM2))"
echo "========================="
```

### **Exercise 2: System Report Script**

```bash
#!/bin/bash
# system_report.sh - Generate system report

# Variables
HOSTNAME=$(hostname)
KERNEL=$(uname -r)
UPTIME=$(uptime -p)
LOAD=$(uptime | awk -F'load average:' '{print $2}')
MEMORY_TOTAL=$(free -h | grep Mem | awk '{print $2}')
MEMORY_USED=$(free -h | grep Mem | awk '{print $3}')
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}')
PROCESSES=$(ps aux | wc -l)
LOGGED_USERS=$(who | wc -l)
CURRENT_DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Report
cat << EOF
============================================
       SYSTEM REPORT
============================================
Generated: $CURRENT_DATE
============================================
Hostname:        $HOSTNAME
Kernel:          $KERNEL
Uptime:          $UPTIME
Load Average:    $LOAD
Memory:          $MEMORY_USED / $MEMORY_TOTAL
Disk Usage (/):  $DISK_USAGE
Processes:       $PROCESSES
Logged Users:    $LOGGED_USERS
============================================
EOF
```

### **Exercise 3: Backup Script with Variables**

```bash
#!/bin/bash
# smart_backup.sh - Backup with variables

# Configuration
SOURCE_DIR="/home/$USER/documents"
BACKUP_ROOT="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_${USER}_${DATE}.tar.gz"
BACKUP_PATH="${BACKUP_ROOT}/${BACKUP_FILE}"

# Create backup directory if needed
mkdir -p "$BACKUP_ROOT"

# Display info
echo "==================================="
echo "Starting Backup"
echo "==================================="
echo "Source: $SOURCE_DIR"
echo "Destination: $BACKUP_PATH"
echo "Timestamp: $DATE"
echo "==================================="

# Create backup
tar -czf "$BACKUP_PATH" "$SOURCE_DIR" 2>/dev/null

# Check result
if [ $? -eq 0 ]; then
    SIZE=$(du -h "$BACKUP_PATH" | cut -f1)
    echo "✅ Backup successful!"
    echo "Size: $SIZE"
else
    echo "❌ Backup failed!"
    exit 1
fi
```

---

## 📝 Key Takeaways

✅ **No spaces around = when assigning variables**  
✅ **Use $ to access variable values**  
✅ **Use ${VAR} for clarity and complex operations**  
✅ **$? contains exit status of last command (0 = success)**  
✅ **$# is number of arguments, $1, $2, etc. are argument values**  
✅ **Use $(command) for command substitution**  
✅ **Use $(( )) for integer arithmetic**  
✅ **Use bc for floating-point math**  
✅ **local variables in functions, export for environment variables**  
✅ **readonly for constants**  

---

## 🚀 Next Steps

You now know how to work with variables and data!

**Next lesson:** [04 - Quoting & Escaping](04-quoting-and-escaping.md) - Master string handling and special characters

---

## 💡 Pro Tips

**Always quote variables:**
```bash
# ❌ Bad (breaks with spaces)
rm $FILE

# ✅ Good
rm "$FILE"
```

**Check if variable is set:**
```bash
if [ -z "$VAR" ]; then
    echo "VAR is not set"
fi
```

**Use uppercase for constants:**
```bash
readonly MAX_RETRIES=3
readonly DATABASE_URL="postgresql://localhost/db"
```

**Debug variable values:**
```bash
echo "DEBUG: VAR=$VAR"
declare -p VAR    # Show variable attributes and value
```

**Array of variables (preview):**
```bash
SERVERS=("web1" "web2" "web3")
echo ${SERVERS[0]}    # web1
echo ${SERVERS[@]}    # All servers
```

You're becoming a bash pro! 🎯
