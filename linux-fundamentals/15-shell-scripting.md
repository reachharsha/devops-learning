# 15 - Shell Scripting

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What shell scripts are and why they matter
- Creating and running scripts
- Variables, conditionals, and loops
- Functions and command-line arguments
- Practical automation examples

---

## 🤖 What is Shell Scripting?

**Real-World Analogy:**
```
Shell Script = Recipe

Instead of:
1. Mix flour (manually)
2. Add eggs (manually)
3. Bake (manually)

You write the recipe once
Run it anytime
Consistent results every time
```

**Why Shell Scripts for DevOps:**
```
✅ Automate repetitive tasks
✅ System maintenance
✅ Deployments
✅ Backups
✅ Monitoring
✅ Server setup
✅ Batch processing

One script = hours saved
```

---

## 📝 Creating Your First Script

### **Basic Structure**

```bash
#!/bin/bash
# This is a comment

echo "Hello, World!"
```

**Breaking it down:**
```
#!/bin/bash     Shebang - tells system to use bash
#               Comment (ignored by bash)
echo            Command to print text
```

### **Creating and Running**

```bash
# 1. Create script
nano hello.sh

# 2. Add content
#!/bin/bash
echo "Hello from my script!"

# 3. Save and exit

# 4. Make executable
chmod +x hello.sh

# 5. Run it
./hello.sh

# Output: Hello from my script!
```

---

## 📦 Variables

### **Creating Variables**

```bash
#!/bin/bash

# Assign variable (NO SPACES around =)
NAME="John"
AGE=25
TODAY=$(date +%Y-%m-%d)

# Use variable (with $)
echo "My name is $NAME"
echo "I am $AGE years old"
echo "Today is $TODAY"

# Output:
# My name is John
# I am 25 years old
# Today is 2024-01-15
```

**Variable rules:**
```bash
# ✅ Good
VAR="value"
VAR=123
VAR=$(command)

# ❌ Bad
VAR = "value"     # No spaces!
VAR= "value"      # No spaces!
VAR ="value"      # No spaces!
```

### **Special Variables**

```bash
#!/bin/bash

$0      # Script name
$1      # First argument
$2      # Second argument
$#      # Number of arguments
$@      # All arguments
$$      # Process ID
$?      # Exit code of last command

# Example:
echo "Script name: $0"
echo "First arg: $1"
echo "All args: $@"
echo "Number of args: $#"
```

---

## 🔢 User Input

```bash
#!/bin/bash

# Simple input
echo "What's your name?"
read NAME
echo "Hello, $NAME!"

# Prompt on same line
read -p "Enter your age: " AGE
echo "You are $AGE years old"

# Silent input (password)
read -sp "Enter password: " PASSWORD
echo  # New line
echo "Password received"

# Multiple inputs
read -p "Enter first and last name: " FIRST LAST
echo "Hello, $FIRST $LAST"
```

---

## 🔀 Conditionals

### **If Statements**

```bash
#!/bin/bash

# Basic if
if [ condition ]; then
    # commands
fi

# If-else
if [ condition ]; then
    # commands
else
    # commands
fi

# If-elif-else
if [ condition1 ]; then
    # commands
elif [ condition2 ]; then
    # commands
else
    # commands
fi
```

### **Comparison Operators**

**Numbers:**
```bash
-eq     Equal
-ne     Not equal
-gt     Greater than
-ge     Greater than or equal
-lt     Less than
-le     Less than or equal

# Example:
if [ $AGE -ge 18 ]; then
    echo "Adult"
else
    echo "Minor"
fi
```

**Strings:**
```bash
=       Equal
!=      Not equal
-z      Empty string
-n      Not empty

# Example:
if [ "$NAME" = "admin" ]; then
    echo "Welcome, admin!"
fi

if [ -z "$NAME" ]; then
    echo "Name is empty"
fi
```

**Files:**
```bash
-f      File exists and is regular file
-d      Directory exists
-e      File/directory exists
-r      Readable
-w      Writable
-x      Executable

# Example:
if [ -f "/etc/hosts" ]; then
    echo "File exists"
fi

if [ -d "/var/log" ]; then
    echo "Directory exists"
fi
```

### **Logical Operators**

```bash
&&      AND
||      OR
!       NOT

# Example:
if [ $AGE -ge 18 ] && [ $AGE -le 65 ]; then
    echo "Working age"
fi

if [ "$USER" = "root" ] || [ "$SUDO_USER" = "root" ]; then
    echo "Running as root"
fi
```

---

## 🔄 Loops

### **For Loop**

```bash
#!/bin/bash

# Loop through list
for ITEM in item1 item2 item3; do
    echo "Processing: $ITEM"
done

# Loop through files
for FILE in *.txt; do
    echo "Found: $FILE"
done

# Loop through range
for i in {1..5}; do
    echo "Number: $i"
done

# C-style loop
for ((i=1; i<=10; i++)); do
    echo "Count: $i"
done

# Loop through command output
for USER in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $USER"
done
```

### **While Loop**

```bash
#!/bin/bash

# Basic while
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    COUNT=$((COUNT + 1))
done

# Read file line by line
while read LINE; do
    echo "Line: $LINE"
done < file.txt

# Infinite loop
while true; do
    echo "Running..."
    sleep 5
done
```

---

## 🎯 Functions

```bash
#!/bin/bash

# Define function
greet() {
    echo "Hello, $1!"
}

# Call function
greet "John"
# Output: Hello, John!

# Function with return value
add() {
    local RESULT=$(($1 + $2))
    echo $RESULT
}

SUM=$(add 5 3)
echo "Sum: $SUM"

# Function with multiple parameters
create_user() {
    local USERNAME=$1
    local EMAIL=$2
    echo "Creating user: $USERNAME ($EMAIL)"
}

create_user "john" "john@example.com"
```

---

## 🛠️ Practical Examples

### **System Backup Script**

```bash
#!/bin/bash

# backup.sh - Backup important directories

# Variables
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d-%H%M%S)
SOURCE_DIRS="/home /etc /var/www"

# Create backup directory
mkdir -p $BACKUP_DIR

# Create backup
echo "Starting backup: $DATE"

for DIR in $SOURCE_DIRS; do
    if [ -d "$DIR" ]; then
        BACKUP_FILE="$BACKUP_DIR/$(basename $DIR)-$DATE.tar.gz"
        echo "Backing up: $DIR → $BACKUP_FILE"
        tar -czf $BACKUP_FILE $DIR 2>/dev/null
    fi
done

# Remove old backups (older than 30 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete

echo "Backup completed!"
```

### **Server Health Check**

```bash
#!/bin/bash

# health-check.sh - Check server health

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Check disk space
echo "=== Disk Space ==="
DF_OUTPUT=$(df -h / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ $DF_OUTPUT -gt 80 ]; then
    echo -e "${RED}WARNING: Disk usage is ${DF_OUTPUT}%${NC}"
else
    echo -e "${GREEN}OK: Disk usage is ${DF_OUTPUT}%${NC}"
fi

# Check memory
echo -e "\n=== Memory ==="
FREE_MEM=$(free | grep Mem | awk '{print ($3/$2) * 100}' | cut -d. -f1)
if [ $FREE_MEM -gt 90 ]; then
    echo -e "${RED}WARNING: Memory usage is ${FREE_MEM}%${NC}"
else
    echo -e "${GREEN}OK: Memory usage is ${FREE_MEM}%${NC}"
fi

# Check CPU load
echo -e "\n=== CPU Load ==="
LOAD=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | sed 's/,//')
echo "Current load: $LOAD"

# Check services
echo -e "\n=== Services ==="
for SERVICE in nginx mysql ssh; do
    if systemctl is-active --quiet $SERVICE; then
        echo -e "${GREEN}✓ $SERVICE is running${NC}"
    else
        echo -e "${RED}✗ $SERVICE is NOT running${NC}"
    fi
done
```

### **Deployment Script**

```bash
#!/bin/bash

# deploy.sh - Deploy application

set -e  # Exit on error

APP_DIR="/var/www/myapp"
REPO="https://github.com/user/myapp.git"
BRANCH="main"

echo "=== Deployment Started ==="

# Pull latest code
echo "Pulling latest code..."
cd $APP_DIR
git pull origin $BRANCH

# Install dependencies
echo "Installing dependencies..."
npm install --production

# Build application
echo "Building application..."
npm run build

# Restart service
echo "Restarting service..."
sudo systemctl restart myapp

# Health check
sleep 5
if curl -f http://localhost:3000/health > /dev/null 2>&1; then
    echo "✅ Deployment successful!"
else
    echo "❌ Health check failed!"
    exit 1
fi
```

### **User Management Script**

```bash
#!/bin/bash

# user-management.sh - Manage users

create_user() {
    local USERNAME=$1
    
    if id "$USERNAME" &>/dev/null; then
        echo "User $USERNAME already exists"
        return 1
    fi
    
    echo "Creating user: $USERNAME"
    sudo useradd -m -s /bin/bash $USERNAME
    echo "User created successfully"
}

delete_user() {
    local USERNAME=$1
    
    if ! id "$USERNAME" &>/dev/null; then
        echo "User $USERNAME does not exist"
        return 1
    fi
    
    echo "Deleting user: $USERNAME"
    sudo userdel -r $USERNAME
    echo "User deleted successfully"
}

# Main menu
echo "User Management"
echo "1. Create user"
echo "2. Delete user"
echo "3. Exit"

read -p "Choose option: " OPTION

case $OPTION in
    1)
        read -p "Enter username: " USERNAME
        create_user $USERNAME
        ;;
    2)
        read -p "Enter username: " USERNAME
        delete_user $USERNAME
        ;;
    3)
        echo "Goodbye!"
        exit 0
        ;;
    *)
        echo "Invalid option"
        exit 1
        ;;
esac
```

---

## 🎨 Advanced Techniques

### **Error Handling**

```bash
#!/bin/bash

# Exit on error
set -e

# Exit on undefined variable
set -u

# Show commands as they execute
set -x

# Combine
set -eux

# Custom error handling
trap 'echo "Error on line $LINENO"' ERR

# Function error handling
do_something() {
    command || {
        echo "Command failed!"
        return 1
    }
}
```

### **Command-Line Arguments**

```bash
#!/bin/bash

# parse-args.sh - Parse command line arguments

while getopts "h:u:p:" opt; do
    case $opt in
        h)
            HOST=$OPTARG
            ;;
        u)
            USER=$OPTARG
            ;;
        p)
            PORT=$OPTARG
            ;;
        \?)
            echo "Invalid option: -$OPTARG"
            exit 1
            ;;
    esac
done

echo "Host: $HOST"
echo "User: $USER"
echo "Port: $PORT"

# Usage: ./parse-args.sh -h localhost -u admin -p 22
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is the shebang line?**
   <details>
   <summary>Answer</summary>
   #!/bin/bash (first line, tells system to use bash)
   </details>

2. **How do you make a script executable?**
   <details>
   <summary>Answer</summary>
   chmod +x script.sh
   </details>

3. **How do you access the first command-line argument?**
   <details>
   <summary>Answer</summary>
   $1
   </details>

4. **How do you check if a file exists?**
   <details>
   <summary>Answer</summary>
   if [ -f "filename" ]; then ... fi
   </details>

5. **How do you create a loop from 1 to 10?**
   <details>
   <summary>Answer</summary>
   for i in {1..10}; do ... done
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# Create a script that:
# 1. Asks for your name
# 2. Checks if you're root
# 3. Shows system info
# 4. Lists files in home directory

nano system-info.sh

#!/bin/bash

# Get user input
read -p "Enter your name: " NAME
echo "Hello, $NAME!"

# Check if root
if [ "$EUID" -eq 0 ]; then
    echo "You are running as root"
else
    echo "You are running as regular user: $USER"
fi

# System info
echo -e "\n=== System Information ==="
echo "Hostname: $(hostname)"
echo "OS: $(uname -s)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"

# Disk usage
echo -e "\n=== Disk Usage ==="
df -h /

# Files in home
echo -e "\n=== Files in Home Directory ==="
ls -lh ~ | head -10

echo -e "\nScript completed!"

# Make executable and run
chmod +x system-info.sh
./system-info.sh
```

---

## 📝 Key Takeaways

✅ **#!/bin/bash** - shebang (first line)  
✅ **chmod +x script.sh** - make executable  
✅ **./script.sh** - run script  
✅ **VAR="value"** - assign variable  
✅ **$VAR** - use variable  
✅ **$1, $2, $@** - command-line arguments  
✅ **if [ condition ]; then ... fi** - conditional  
✅ **for ... do ... done** - loop  
✅ **function_name() { ... }** - function  
✅ **set -e** - exit on error  

---

## 🚀 Next Steps

You can now automate tasks with shell scripts!

**Next lesson:** [16-security-fundamentals.md](16-security-fundamentals.md) - Linux security basics

---

## 💡 Pro Tips

**Script template:**
```bash
#!/bin/bash

#############################################
# Script Name: script.sh
# Description: What this script does
# Author: Your Name
# Date: 2024-01-15
#############################################

# Exit on error
set -e

# Variables
VAR1="value1"
VAR2="value2"

# Functions
main() {
    echo "Script started"
    # Your code here
    echo "Script completed"
}

# Run main function
main "$@"
```

**Debugging:**
```bash
# Add at top of script
set -x          # Show commands
set -v          # Show lines
bash -x script.sh  # Debug mode
```

**Best practices:**
```bash
# Always quote variables
echo "$VAR"             # Good ✅
echo $VAR               # Bad ❌

# Check if variable is set
if [ -z "$VAR" ]; then
    echo "VAR is not set"
    exit 1
fi

# Use functions for reusable code
# Add error handling
# Log important actions
# Comment your code
```

Master shell scripting = DevOps automation! 🚀
