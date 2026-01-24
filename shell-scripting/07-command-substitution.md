# 07 - Command Substitution

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What command substitution is and why it's essential
- Syntax: $() vs backticks
- Nested command substitution
- Process substitution <() and >()
- Practical use cases in scripts
- Performance considerations

---

## 📋 What is Command Substitution?

**Real-World Analogy:**
```
Regular Variable:
NAME="Alice"
→ Fixed value

Command Substitution:
CURRENT_USER=$(whoami)
→ Value comes from command output

Think of it as:
Variable = Storage Container
Command Substitution = Factory that fills the container
→ Dynamic value generation!
```

**Technical Definition:**
```bash
Command Substitution = Run command, capture output, use it

Instead of:
$ whoami
alice

$ echo "User is alice"

Do this:
$ echo "User is $(whoami)"
User is alice

The command runs, output becomes part of your script!
```

---

## 💻 Syntax: $() vs Backticks

### **Modern Syntax: $()**

```bash
# Recommended way (POSIX-compliant, readable)
USER=$(whoami)
echo "Current user: $USER"

DATE=$(date +%Y-%m-%d)
echo "Today is: $DATE"

FILES_COUNT=$(ls | wc -l)
echo "Number of files: $FILES_COUNT"

# Inline usage
echo "Hello, $(whoami)!"
echo "Current directory: $(pwd)"
```

### **Old Syntax: `` (Backticks)**

```bash
# Old way (still works, but deprecated)
USER=`whoami`
echo "Current user: $USER"

DATE=`date +%Y-%m-%d`
echo "Today is: $DATE"

# ⚠️ Harder to read, harder to nest
```

### **Why Prefer $() Over Backticks?**

```bash
# 1. Easier to read
$(command)              # ✅ Clear
`command`               # ❌ Confusing with quotes

# 2. Easier to nest
$(command1 $(command2))                 # ✅ Clear nesting
`command1 \`command2\``                 # ❌ Must escape!

# 3. Better syntax highlighting
echo "User: $(whoami), Dir: $(pwd)"    # ✅ Colors help
echo "User: `whoami`, Dir: `pwd`"      # ❌ Less clear

# 4. Consistent with other $() constructs
$((arithmetic))
$(command)
${parameter}
# All use $ prefix!

# Always use $() in new scripts!
```

---

## 🔄 Basic Command Substitution Examples

### **Simple Commands:**

```bash
# Get current user
CURRENT_USER=$(whoami)
echo "Logged in as: $CURRENT_USER"

# Get hostname
HOSTNAME=$(hostname)
echo "Computer name: $HOSTNAME"

# Get current date
TODAY=$(date +%Y-%m-%d)
echo "Date: $TODAY"

# Get current time
TIME=$(date +%H:%M:%S)
echo "Time: $TIME"

# Count files
FILE_COUNT=$(ls -1 | wc -l)
echo "Files in directory: $FILE_COUNT"

# Get disk usage
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}')
echo "Root disk usage: $DISK_USAGE"

# Get system uptime
UPTIME=$(uptime -p)
echo "System uptime: $UPTIME"
```

### **Using in Strings:**

```bash
#!/bin/bash

echo "=== System Report ==="
echo "Generated on: $(date)"
echo "By user: $(whoami)"
echo "On host: $(hostname)"
echo ""
echo "Current directory: $(pwd)"
echo "Files in directory: $(ls -1 | wc -l)"
echo "Disk usage: $(df -h . | tail -1 | awk '{print $5}')"
echo ""
echo "=== End Report ==="
```

### **File and Path Operations:**

```bash
# Get absolute path
SCRIPT_DIR=$(cd "$(dirname "$0")" && pwd)
echo "Script location: $SCRIPT_DIR"

# Get filename without path
FILENAME=$(basename "/path/to/file.txt")
echo "File: $FILENAME"             # file.txt

# Get directory from path
DIRECTORY=$(dirname "/path/to/file.txt")
echo "Directory: $DIRECTORY"       # /path/to

# Find files
LOG_FILE=$(find /var/log -name "*.log" -type f | head -1)
echo "Found log: $LOG_FILE"
```

---

## 🪆 Nested Command Substitution

### **Nesting with $():**

```bash
# Inner command runs first, then outer
echo "Parent directory: $(dirname $(pwd))"

# Equivalent to:
CURRENT=$(pwd)
PARENT=$(dirname "$CURRENT")
echo "Parent directory: $PARENT"

# Complex nesting
OLDEST_FILE=$(ls -t $(find . -type f) | tail -1)
echo "Oldest file: $OLDEST_FILE"

# Multiple levels
USER_HOME=$(dirname $(dirname $(which bash)))
echo "Path: $USER_HOME"
```

### **Clear vs Confusing:**

```bash
# ❌ Too complex - hard to read
RESULT=$(grep "error" $(find $(pwd) -name "*.log"))

# ✅ Better - break it down
CURRENT_DIR=$(pwd)
LOG_FILES=$(find "$CURRENT_DIR" -name "*.log")
RESULT=$(grep "error" $LOG_FILES)

# ✅ Or use variables
DIR=$(pwd)
LOGS=$(find "$DIR" -name "*.log")
ERRORS=$(grep "error" $LOGS)
echo "$ERRORS"
```

---

## 🔀 Process Substitution

**Different from Command Substitution!**
```
Command Substitution: $()
→ Captures output as text

Process Substitution: <() and >()
→ Creates temporary file-like object
→ Useful for commands expecting file arguments
```

### **Process Substitution: <()**

```bash
# <() creates a temporary file descriptor

# Compare output of two commands
diff <(ls dir1) <(ls dir2)

# Sort and compare
diff <(sort file1.txt) <(sort file2.txt)

# Multiple inputs
paste <(cat file1.txt) <(cat file2.txt)

# Practical: Compare running processes
diff <(ps aux | grep apache) <(ps aux | grep nginx)
```

### **Why Process Substitution?**

```bash
# Without process substitution (needs temporary files)
ls dir1 > /tmp/list1.txt
ls dir2 > /tmp/list2.txt
diff /tmp/list1.txt /tmp/list2.txt
rm /tmp/list1.txt /tmp/list2.txt

# With process substitution (no temporary files!)
diff <(ls dir1) <(ls dir2)

# Much cleaner and safer!
```

### **Process Substitution: >()**

```bash
# >() captures output and sends to command

# Tee-like behavior - write to multiple files
echo "Data" | tee >(cat > file1.txt) >(cat > file2.txt)

# Log to file and show on screen
command | tee >(logger -t myapp)

# Process output differently
generate_data | tee >(process_errors > errors.log) >(process_stats > stats.log)
```

---

## 🎯 Practical Use Cases

### **Example 1: Dated Backups**

```bash
#!/bin/bash

# Create backup with timestamp
SOURCE="/home/user/documents"
BACKUP_DIR="/backup"
DATE=$(date +%Y%m%d_%H%M%S)

BACKUP_FILE="${BACKUP_DIR}/backup_${DATE}.tar.gz"

echo "Creating backup: $BACKUP_FILE"
tar -czf "$BACKUP_FILE" "$SOURCE"

echo "Backup created: $(ls -lh "$BACKUP_FILE" | awk '{print $5}')"
```

### **Example 2: System Monitoring**

```bash
#!/bin/bash

# Collect system stats
CPU_USAGE=$(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)
MEM_USAGE=$(free | grep Mem | awk '{printf "%.2f", $3/$2 * 100}')
DISK_USAGE=$(df -h / | tail -1 | awk '{print $5}' | cut -d'%' -f1)

echo "=== System Status ==="
echo "CPU Usage:  ${CPU_USAGE}%"
echo "Memory:     ${MEM_USAGE}%"
echo "Disk:       ${DISK_USAGE}%"

# Alert if high
if [[ $(echo "$CPU_USAGE > 80" | bc) -eq 1 ]]; then
    echo "⚠️  High CPU usage!"
fi
```

### **Example 3: Log Rotation**

```bash
#!/bin/bash

LOG_FILE="/var/log/myapp.log"
MAX_SIZE=10485760  # 10MB in bytes

CURRENT_SIZE=$(stat -f%z "$LOG_FILE" 2>/dev/null || stat -c%s "$LOG_FILE")

if [[ $CURRENT_SIZE -gt $MAX_SIZE ]]; then
    TIMESTAMP=$(date +%Y%m%d_%H%M%S)
    mv "$LOG_FILE" "${LOG_FILE}.${TIMESTAMP}"
    touch "$LOG_FILE"
    echo "Log rotated at $(date)"
fi
```

### **Example 4: Git Status in Prompt**

```bash
#!/bin/bash

# Get current git branch
get_git_branch() {
    git branch 2>/dev/null | grep '*' | sed 's/* //'
}

# Get git status
get_git_status() {
    if [[ $(git status --porcelain 2>/dev/null | wc -l) -gt 0 ]]; then
        echo "*"  # Uncommitted changes
    fi
}

# Custom prompt
PS1="\u@\h:\w $(get_git_branch)$(get_git_status)\$ "
```

### **Example 5: Find and Process Files**

```bash
#!/bin/bash

# Find large files and list them
echo "Large files (>100MB):"

find /home -type f -size +100M 2>/dev/null | while read -r file; do
    SIZE=$(du -h "$file" | cut -f1)
    echo "  $SIZE - $file"
done

# Total count
COUNT=$(find /home -type f -size +100M 2>/dev/null | wc -l)
echo "Total large files: $COUNT"
```

---

## ⚡ Performance Considerations

### **Avoid Unnecessary Subshells:**

```bash
# ❌ Slow - creates subshell for each iteration
for i in {1..1000}; do
    DATE=$(date +%s)
    echo "$DATE"
done

# ✅ Fast - call once before loop
DATE=$(date +%s)
for i in {1..1000}; do
    echo "$DATE"
done

# ✅ Or call once if value doesn't change
```

### **Cache Results:**

```bash
# ❌ Slow - calls hostname multiple times
echo "Server: $(hostname)"
echo "Host: $(hostname)"
echo "Machine: $(hostname)"

# ✅ Fast - call once, reuse
HOSTNAME=$(hostname)
echo "Server: $HOSTNAME"
echo "Host: $HOSTNAME"
echo "Machine: $HOSTNAME"
```

### **Avoid Pipes When Not Needed:**

```bash
# ❌ Slower - unnecessary pipe
COUNT=$(cat file.txt | wc -l)

# ✅ Faster - direct input
COUNT=$(wc -l < file.txt)

# ❌ Slower
USERS=$(cat /etc/passwd | grep "/bin/bash")

# ✅ Faster
USERS=$(grep "/bin/bash" /etc/passwd)
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does $() do?**
   <details>
   <summary>Answer</summary>
   Runs command and captures its output for use in script
   </details>

2. **Why prefer $() over backticks?**
   <details>
   <summary>Answer</summary>
   Easier to read, easier to nest, better syntax highlighting
   </details>

3. **What's the difference between $() and <()? **
   <details>
   <summary>Answer</summary>
   $() captures text output, <() creates file-like descriptor
   </details>

4. **When should you cache command results?**
   <details>
   <summary>Answer</summary>
   When using the same output multiple times in a script
   </details>

5. **How do you nest commands?**
   <details>
   <summary>Answer</summary>
   $(outer $(inner)) - inner runs first, result goes to outer
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Basic Substitution**

```bash
#!/bin/bash

echo "Current user: $(whoami)"
echo "Current directory: $(pwd)"
echo "Current date: $(date +%Y-%m-%d)"
echo "Current time: $(date +%H:%M:%S)"
echo "Number of files: $(ls -1 | wc -l)"
echo "Shell: $SHELL"
echo "Home directory: $HOME"
```

### **Exercise 2: System Info Script**

```bash
#!/bin/bash

echo "=== System Information ==="
echo "Hostname:     $(hostname)"
echo "Kernel:       $(uname -r)"
echo "Uptime:       $(uptime -p)"
echo "Load Average: $(uptime | awk -F'load average:' '{print $2}')"
echo "Users:        $(who | wc -l)"
echo "Processes:    $(ps aux | wc -l)"
echo ""
echo "Memory:"
free -h | grep -E 'Mem|Swap'
echo ""
echo "Disk Usage:"
df -h / | tail -1
```

### **Exercise 3: Process Substitution**

```bash
#!/bin/bash

# Compare directories
echo "Comparing dir1 and dir2:"
diff <(ls dir1 | sort) <(ls dir2 | sort)

# Find unique files
comm -23 <(ls dir1 | sort) <(ls dir2 | sort)
```

### **Exercise 4: File Size Reporter**

```bash
#!/bin/bash

DIRECTORY=${1:-.}

echo "=== File Size Report for $DIRECTORY ==="
echo "Total files: $(find "$DIRECTORY" -type f | wc -l)"
echo "Total size:  $(du -sh "$DIRECTORY" | cut -f1)"
echo ""
echo "Largest files:"
find "$DIRECTORY" -type f -exec du -h {} + | sort -rh | head -10
```

---

## 📝 Key Takeaways

✅ **$()** captures command output  
✅ **Prefer $()** over backticks  
✅ **Nested** commands run inside-out  
✅ **Cache results** for performance  
✅ **<()** creates file-like descriptor  
✅ **>()** sends to command input  
✅ **Quote variables** in substitution  
✅ **Avoid unnecessary** pipes  
✅ **Test commands** before substituting  
✅ **Use variables** to clarify complex nesting  

---

## 🚀 Next Steps

You can now capture and use command output dynamically!

**Next lesson:** [08-user-input-output.md](08-user-input-output.md) - Reading user input

---

## 💡 Pro Tips

**Debugging Command Substitution:**
```bash
# See what command produces
RESULT=$(ls -la)
echo "Result: $RESULT"

# Or run separately first
ls -la          # See output
RESULT=$(ls -la)
```

**Error Handling:**
```bash
# Check if command succeeded
RESULT=$(command 2>/dev/null)
if [[ $? -eq 0 ]]; then
    echo "Success: $RESULT"
else
    echo "Command failed"
fi

# Or use conditional
RESULT=$(command) || RESULT="default value"
```

**Common Patterns:**
```bash
# Get script directory
SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)

# Timestamp for files
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Generate random
RANDOM_NUM=$(( RANDOM % 100 ))

# Get IP address
IP=$(hostname -I | awk '{print $1}')
```

Command substitution makes your scripts dynamic! 🚀
