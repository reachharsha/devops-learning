# 02 - Your First Script

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Creating and running your first script
- Script structure and best practices
- Making scripts executable
- Comments and documentation
- Basic debugging techniques
- Script development workflow

---

## 📝 Creating Your First Script

### **The Development Workflow:**

```
1. Create file       → nano script.sh
2. Add shebang       → #!/bin/bash
3. Write commands    → echo "Hello"
4. Save file         → Ctrl+O, Enter, Ctrl+X
5. Make executable   → chmod +x script.sh
6. Run script        → ./script.sh
7. Debug if needed   → bash -x script.sh
```

---

## 🎬 Hello World Script

### **Version 1: Minimal**

```bash
#!/bin/bash
echo "Hello, World!"
```

**Create and run:**
```bash
# Create the script
nano hello.sh

# Add the two lines above, save and exit

# Make executable
chmod +x hello.sh

# Run it
./hello.sh
# Output: Hello, World!
```

---

### **Version 2: With Comments**

```bash
#!/bin/bash
# hello.sh - My first shell script
# Prints a greeting message

# Print greeting
echo "Hello, World!"

# Print current date
echo "Today is: $(date)"
```

**Run it:**
```bash
./hello.sh
# Output:
# Hello, World!
# Today is: Thu Jan 15 10:30:45 UTC 2024
```

---

### **Version 3: Professional Structure**

```bash
#!/bin/bash

###############################################################################
# Script Name: hello.sh
# Description: Professional greeting script
# Author: Your Name
# Date: 2024-01-15
# Version: 1.0
###############################################################################

# Exit on any error
set -e

# Variables
GREETING="Hello, World!"
USER_NAME=$(whoami)
CURRENT_DATE=$(date +"%Y-%m-%d %H:%M:%S")

# Main script
echo "========================================="
echo "$GREETING"
echo "========================================="
echo "User: $USER_NAME"
echo "Date: $CURRENT_DATE"
echo "Host: $(hostname)"
echo "========================================="

# Exit successfully
exit 0
```

**Run it:**
```bash
./hello.sh
# Output:
# =========================================
# Hello, World!
# =========================================
# User: john
# Date: 2024-01-15 10:30:45
# Host: myserver
# =========================================
```

---

## 🏗️ Script Structure Best Practices

### **Professional Script Template:**

```bash
#!/bin/bash

###############################################################################
# Script Name: script_name.sh
# Description: Brief description of what this script does
# Author: Your Name
# Email: your.email@example.com
# Date Created: YYYY-MM-DD
# Last Modified: YYYY-MM-DD
# Version: 1.0
#
# Usage: ./script_name.sh [options] [arguments]
#
# Options:
#   -h, --help     Display this help message
#   -v, --verbose  Enable verbose output
#
# Examples:
#   ./script_name.sh
#   ./script_name.sh -v
###############################################################################

#------------------------------------------------------------------------------
# VARIABLES
#------------------------------------------------------------------------------

# Script metadata
SCRIPT_NAME=$(basename "$0")
SCRIPT_DIR=$(dirname "$(readlink -f "$0")")
SCRIPT_VERSION="1.0"

# Configuration variables
DEBUG=false
VERBOSE=false

#------------------------------------------------------------------------------
# FUNCTIONS
#------------------------------------------------------------------------------

# Display usage information
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [options]

Options:
    -h, --help     Show this help message
    -v, --verbose  Enable verbose output
    -d, --debug    Enable debug mode

Examples:
    $SCRIPT_NAME
    $SCRIPT_NAME -v
EOF
    exit 0
}

# Log message with timestamp
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"
}

# Error handler
error() {
    echo "[ERROR] $*" >&2
    exit 1
}

#------------------------------------------------------------------------------
# MAIN SCRIPT
#------------------------------------------------------------------------------

main() {
    log "Script started"
    
    # Your script logic here
    echo "Hello from $SCRIPT_NAME"
    
    log "Script completed successfully"
    exit 0
}

# Run main function
main "$@"
```

---

## 📋 Comments: The Good, The Bad, The Ugly

### **Good Comments:**

```bash
#!/bin/bash

# Calculate disk usage for /var/log
DISK_USAGE=$(du -sh /var/log | cut -f1)

# Send alert if usage exceeds 1GB
if [[ "$DISK_USAGE" > "1G" ]]; then
    # Critical threshold reached - notify admin
    mail -s "Disk Alert" admin@example.com <<< "Log directory is ${DISK_USAGE}"
fi

# Rotate logs older than 30 days
find /var/log -name "*.log" -mtime +30 -delete
```

### **Bad Comments (Obvious/Redundant):**

```bash
#!/bin/bash

# Set variable x to 5
x=5

# Print x
echo $x

# Increment x
((x++))

# Loop from 1 to 10
for i in {1..10}; do
    # Print i
    echo $i
done
```

### **Commenting Best Practices:**

```bash
#!/bin/bash

# WHY, not WHAT
# ✅ Good: Explains reasoning
# Retry 3 times because API is unreliable during peak hours
for attempt in {1..3}; do
    curl -s https://api.example.com && break
    sleep 5
done

# ❌ Bad: States the obvious
# Loop 3 times
for attempt in {1..3}; do
    curl -s https://api.example.com && break
    sleep 5
done

# Document complex logic
# ✅ Good
# Extract IPv4 addresses from nginx access log
# Format: IP - - [date] "request" status size
grep -oE '\b([0-9]{1,3}\.){3}[0-9]{1,3}\b' access.log

# Mark TODOs and FIXMEs
# TODO: Add email notification on failure
# FIXME: This breaks with filenames containing spaces
# HACK: Temporary workaround for bash 3.x compatibility

# Document assumptions
# Assumes: AWS CLI is installed and configured
# Requires: User has sudo privileges
# Note: This script must run as root
```

---

## 🔍 Making Scripts Executable

### **Understanding Permissions:**

```bash
# Create new script
echo '#!/bin/bash' > test.sh
echo 'echo "Test"' >> test.sh

# Check permissions
ls -l test.sh
# -rw-r--r-- 1 user user 25 Jan 15 10:00 test.sh
#  │││ │││ │││
#  │││ │││ └── Others: read
#  │││ └────── Group: read
#  └────────── Owner: read, write

# Try to execute (fails)
./test.sh
# bash: ./test.sh: Permission denied

# Add execute permission for owner
chmod u+x test.sh

# Check again
ls -l test.sh
# -rwxr--r-- 1 user user 25 Jan 15 10:00 test.sh
#  ^^^
#  Now owner can execute!

# Now works
./test.sh
# Output: Test
```

### **Different chmod Approaches:**

```bash
# Numeric (recommended)
chmod 755 script.sh    # rwxr-xr-x (owner: all, others: read+execute)
chmod 750 script.sh    # rwxr-x--- (owner: all, group: read+execute)
chmod 700 script.sh    # rwx------ (owner only)

# Symbolic
chmod +x script.sh     # Add execute for everyone
chmod u+x script.sh    # Add execute for owner only
chmod g+x script.sh    # Add execute for group
chmod o+x script.sh    # Add execute for others
chmod a+x script.sh    # Add execute for all (same as +x)

# Remove execute
chmod -x script.sh     # Remove execute for all
chmod u-x script.sh    # Remove execute for owner

# Best practice for scripts
chmod 755 script.sh    # Most common
```

---

## 🐛 Basic Debugging Techniques

### **set Options for Safer Scripts:**

```bash
#!/bin/bash

# Exit on error (recommended for all scripts)
set -e
# If any command fails, script stops immediately

# Exit on undefined variable
set -u
# Using undefined variable causes error

# Fail on pipe errors
set -o pipefail
# Catches errors in piped commands

# Debug mode (trace execution)
set -x
# Prints each command before executing

# Combine them (common)
set -euo pipefail

# Example with set -e:
#!/bin/bash
set -e

echo "Step 1"
false              # This fails
echo "Step 2"      # Never executed (script exits on false)
```

### **Debugging Methods:**

```bash
# Method 1: Enable tracing in script
#!/bin/bash
set -x    # Turn on debugging
# ... your code ...
set +x    # Turn off debugging

# Method 2: Run with bash -x
bash -x script.sh
# Shows each command as it executes

# Method 3: Conditional debugging
#!/bin/bash
DEBUG=${DEBUG:-false}

debug_log() {
    if $DEBUG; then
        echo "[DEBUG] $*" >&2
    fi
}

debug_log "Starting process"
# ... code ...
debug_log "Process complete"

# Run with debugging:
DEBUG=true ./script.sh

# Method 4: set -v (verbose)
#!/bin/bash
set -v    # Show lines as read
# Shows each line before execution

# Method 5: Custom debug function
#!/bin/bash
debug() {
    if [[ "${DEBUG}" == "true" ]]; then
        echo "DEBUG: $*" >&2
    fi
}

debug "This is a debug message"
```

### **Common Debugging Patterns:**

```bash
#!/bin/bash

# Print variable values
echo "DEBUG: VAR=$VAR"

# Show execution point
echo "DEBUG: Reached line ${LINENO}"

# Trap errors
trap 'echo "Error at line ${LINENO}"' ERR

# Show command before execution
set -x
complex_command arg1 arg2
set +x

# Pause for inspection
echo "Press Enter to continue..."
read

# Conditional execution
if [[ "$DEBUG" == "true" ]]; then
    echo "Debug info: ..."
fi
```

---

## 🎯 Running Scripts: Different Methods

### **Method 1: Direct Execution (./):**

```bash
# Requires execute permission
chmod +x script.sh
./script.sh

# Uses shebang to determine interpreter
# Script runs in subshell (doesn't affect current shell)
```

### **Method 2: Explicit Interpreter:**

```bash
# No execute permission needed
bash script.sh
sh script.sh

# Ignores shebang
# Runs in subshell
```

### **Method 3: Source/Dot (Current Shell):**

```bash
source script.sh
. script.sh

# Runs in CURRENT shell (not subshell)
# Variables/functions remain after script ends
# Use for setting environment variables

# Example:
# set_env.sh:
export DATABASE_URL="postgresql://localhost/mydb"
export API_KEY="secret123"

# After sourcing:
source set_env.sh
echo $DATABASE_URL    # Available in current shell
```

### **Differences:**

```bash
# script.sh:
#!/bin/bash
export VAR="hello"
cd /tmp
echo "In script: $(pwd)"

# Test 1: Direct execution
./script.sh
echo "After script: $(pwd)"    # Still in original directory
echo "VAR: $VAR"                # VAR not set (subshell)

# Test 2: Source
source script.sh
echo "After source: $(pwd)"     # Now in /tmp
echo "VAR: $VAR"                # VAR is set (same shell)
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the first line of every bash script?**
   <details>
   <summary>Answer</summary>
   #!/bin/bash (the shebang line)
   </details>

2. **How do you make a script executable?**
   <details>
   <summary>Answer</summary>
   chmod +x script.sh (or chmod 755 script.sh)
   </details>

3. **What does `set -e` do?**
   <details>
   <summary>Answer</summary>
   Exits the script immediately if any command fails (returns non-zero)
   </details>

4. **What's the difference between ./script.sh and source script.sh?**
   <details>
   <summary>Answer</summary>
   ./script.sh runs in subshell (doesn't affect current shell)
   source script.sh runs in current shell (variables/changes persist)
   </details>

5. **How do you debug a script to see each command as it runs?**
   <details>
   <summary>Answer</summary>
   bash -x script.sh (or add set -x inside the script)
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Create a System Info Script**

```bash
#!/bin/bash
# system_info.sh - Display system information

echo "================================"
echo "     SYSTEM INFORMATION"
echo "================================"
echo ""
echo "Hostname: $(hostname)"
echo "Kernel: $(uname -r)"
echo "Uptime: $(uptime -p)"
echo "Current User: $USER"
echo "Home Directory: $HOME"
echo "Shell: $SHELL"
echo "Current Directory: $(pwd)"
echo "Date: $(date)"
echo ""
echo "================================"
```

### **Exercise 2: Create a Greeting Script with Functions**

```bash
#!/bin/bash
# greeting.sh - Personalized greeting

# Function to get time of day
get_greeting() {
    local hour=$(date +%H)
    
    if [ $hour -lt 12 ]; then
        echo "Good morning"
    elif [ $hour -lt 18 ]; then
        echo "Good afternoon"
    else
        echo "Good evening"
    fi
}

# Main
echo "$(get_greeting), $USER!"
echo "Welcome to $(hostname)"
echo "You are running bash version: $BASH_VERSION"
```

### **Exercise 3: Script with Error Handling**

```bash
#!/bin/bash
# safe_script.sh - Script with proper error handling

set -euo pipefail

# Function to handle errors
error_handler() {
    echo "Error occurred in script at line: ${1}" >&2
    exit 1
}

# Trap errors
trap 'error_handler ${LINENO}' ERR

# Main logic
echo "Starting script..."

# This would fail if /nonexistent doesn't exist
# cd /nonexistent    # Uncomment to test error handling

echo "Script completed successfully"
exit 0
```

---

## 📝 Key Takeaways

✅ **Always start with #!/bin/bash**  
✅ **Use chmod +x to make scripts executable**  
✅ **Add comments to explain WHY, not WHAT**  
✅ **Use set -e for safer scripts (exit on error)**  
✅ **Structure scripts with header, variables, functions, main**  
✅ **Debug with bash -x or set -x**  
✅ **Source scripts when you need to affect current shell**  
✅ **Use ./script.sh for normal execution**  
✅ **Document your scripts with usage information**  

---

## 🚀 Next Steps

You can now create, structure, and run your own scripts!

**Next lesson:** [03 - Variables & Data Types](03-variables-and-data-types.md) - Working with data in scripts

---

## 💡 Pro Tips

**Quick script template:**
```bash
#!/bin/bash
set -euo pipefail

# Script logic here

exit 0
```

**Test script syntax without running:**
```bash
bash -n script.sh    # Check for syntax errors
```

**Make script executable during creation:**
```bash
nano script.sh
# Write script
# Save and exit

# One-liner to create and make executable
chmod +x script.sh
```

**Shebang best practices:**
```bash
#!/bin/bash           # ✅ Most common
#!/usr/bin/env bash   # ✅ More portable
#!/bin/sh             # ✅ For POSIX compliance
```

**Create script from command line:**
```bash
cat > script.sh << 'EOF'
#!/bin/bash
echo "Hello, World!"
EOF
chmod +x script.sh
./script.sh
```

You're now a script creator! 🚀
