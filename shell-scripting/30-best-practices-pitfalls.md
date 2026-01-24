# 30 - Best Practices and Common Pitfalls

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Shell scripting best practices
- Common mistakes and how to avoid them
- Code organization and style
- Security pitfalls
- Performance anti-patterns
- Debugging strategies
- Documentation standards

---

## ✅ Best Practices

### **Script Structure:**

```bash
#!/bin/bash

###############################################################################
# Script Name: example.sh
# Description: Brief description of what this script does
# Author: Your Name
# Date: 2024-01-15
# Version: 1.0.0
# Usage: ./example.sh [OPTIONS] [ARGUMENTS]
# Requirements: bash 4.0+, curl, jq
###############################################################################

#------------------------------------------------------------------------------
# Strict Mode
#------------------------------------------------------------------------------

set -euo pipefail
IFS=$'\n\t'

#------------------------------------------------------------------------------
# Constants and Configuration
#------------------------------------------------------------------------------

readonly SCRIPT_NAME="$(basename "${BASH_SOURCE[0]}")"
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly VERSION="1.0.0"

# Configuration
CONFIG_FILE="${CONFIG_FILE:-/etc/app/config.conf}"
LOG_FILE="${LOG_FILE:-/var/log/app.log}"

#------------------------------------------------------------------------------
# Variables
#------------------------------------------------------------------------------

VERBOSE=false
DRY_RUN=false

#------------------------------------------------------------------------------
# Functions
#------------------------------------------------------------------------------

# Logging
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    echo "[ERROR] $*" >&2
    exit 1
}

# Usage information
usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] [ARGUMENTS]

Description:
    Detailed description of what this script does

Options:
    -v, --verbose    Enable verbose output
    -d, --dry-run    Dry run mode (don't make changes)
    -h, --help       Show this help message
    -V, --version    Show version information

Examples:
    $SCRIPT_NAME -v input.txt
    $SCRIPT_NAME --dry-run --verbose

EOF
    exit 0
}

# Version information
version() {
    echo "$SCRIPT_NAME version $VERSION"
    exit 0
}

#------------------------------------------------------------------------------
# Argument Parsing
#------------------------------------------------------------------------------

parse_args() {
    while [[ $# -gt 0 ]]; do
        case $1 in
            -v|--verbose)
                VERBOSE=true
                shift
                ;;
            -d|--dry-run)
                DRY_RUN=true
                shift
                ;;
            -h|--help)
                usage
                ;;
            -V|--version)
                version
                ;;
            -*)
                error "Unknown option: $1"
                ;;
            *)
                # Positional argument
                break
                ;;
        esac
    done
}

#------------------------------------------------------------------------------
# Main Logic
#------------------------------------------------------------------------------

main() {
    log "Script started"
    
    # Your main logic here
    
    log "Script completed successfully"
}

#------------------------------------------------------------------------------
# Signal Handlers
#------------------------------------------------------------------------------

cleanup() {
    log "Cleaning up..."
    # Remove temporary files, etc.
}

trap cleanup EXIT
trap 'error "Script interrupted"' INT TERM

#------------------------------------------------------------------------------
# Entry Point
#------------------------------------------------------------------------------

parse_args "$@"
main "$@"
```

---

## ❌ Common Pitfalls and Solutions

### **1. Unquoted Variables:**

```bash
#!/bin/bash

# ❌ WRONG
FILE=my file.txt
cat $FILE           # Tries to cat "my" and "file.txt"

# ✅ CORRECT
FILE="my file.txt"
cat "$FILE"         # Cats "my file.txt"

# ❌ WRONG
rm $FILES           # Word splitting!

# ✅ CORRECT
rm "$FILES"         # Treats as single argument
```

### **2. Not Checking Exit Codes:**

```bash
#!/bin/bash

# ❌ WRONG
cd /some/directory
rm -rf *            # Danger if cd failed!

# ✅ CORRECT
cd /some/directory || exit 1
rm -rf *

# ✅ BETTER
if ! cd /some/directory; then
    echo "Failed to change directory"
    exit 1
fi
rm -rf *

# ✅ BEST
set -e
cd /some/directory  # Script exits if this fails
rm -rf *
```

### **3. Useless Use of cat:**

```bash
#!/bin/bash

# ❌ WRONG (UUOC - Useless Use of Cat)
cat file.txt | grep pattern

# ✅ CORRECT
grep pattern file.txt

# ❌ WRONG
cat file.txt | wc -l

# ✅ CORRECT
wc -l < file.txt
# or
wc -l file.txt
```

### **4. Not Handling Spaces in Filenames:**

```bash
#!/bin/bash

# ❌ WRONG
for file in $(ls *.txt); do
    echo $file
done

# ✅ CORRECT
for file in *.txt; do
    echo "$file"
done

# ✅ ALSO CORRECT
find . -name "*.txt" -print0 | while IFS= read -r -d '' file; do
    echo "$file"
done
```

### **5. Using [ ] Instead of [[ ]]:**

```bash
#!/bin/bash

# ❌ PROBLEMATIC with [ ]
VAR="hello world"
if [ $VAR = "hello world" ]; then  # Fails due to word splitting
    echo "match"
fi

# ✅ CORRECT with [[ ]]
if [[ $VAR = "hello world" ]]; then
    echo "match"
fi

# ❌ WRONG
if [ $VAR == "value" ]; then  # == is bashism in [ ]

# ✅ CORRECT
if [[ $VAR == "value" ]]; then
    echo "match"
fi
```

### **6. Not Using Local Variables:**

```bash
#!/bin/bash

# ❌ WRONG
function my_function() {
    COUNT=0  # Global variable!
    for i in {1..10}; do
        ((COUNT++))
    done
    echo $COUNT
}

# ✅ CORRECT
function my_function() {
    local count=0  # Local to function
    for i in {1..10}; do
        ((count++))
    done
    echo $count
}
```

### **7. Unsafe Temporary Files:**

```bash
#!/bin/bash

# ❌ WRONG - Predictable filename
TMPFILE="/tmp/myfile.$$"
echo "data" > "$TMPFILE"

# ✅ CORRECT - Use mktemp
TMPFILE=$(mktemp) || exit 1
trap "rm -f '$TMPFILE'" EXIT
echo "data" > "$TMPFILE"

# ✅ ALSO CORRECT - Atomic creation
TMPFILE=$(mktemp /tmp/myfile.XXXXXX)
```

### **8. Incorrect Loop Over Lines:**

```bash
#!/bin/bash

# ❌ WRONG - Loses whitespace, splits on spaces
for line in $(cat file.txt); do
    echo "$line"
done

# ✅ CORRECT
while IFS= read -r line; do
    echo "$line"
done < file.txt

# ✅ ALSO CORRECT - Preserve trailing newline
while IFS= read -r line || [ -n "$line" ]; do
    echo "$line"
done < file.txt
```

### **9. Command Substitution Issues:**

```bash
#!/bin/bash

# ❌ WRONG - Loses exit code
RESULT=$(command_that_might_fail)
if [ $? -eq 0 ]; then  # Always 0!
    echo "success"
fi

# ✅ CORRECT
if RESULT=$(command_that_might_fail); then
    echo "success: $RESULT"
else
    echo "failed"
fi

# ❌ WRONG - Nested quotes
VAR="$(echo "$(cat file.txt)")"

# ✅ CORRECT
VAR="$(cat file.txt)"
```

### **10. Arithmetic Errors:**

```bash
#!/bin/bash

# ❌ WRONG
COUNT=$COUNT+1      # String concatenation!

# ✅ CORRECT
((COUNT++))
# or
COUNT=$((COUNT + 1))
# or
let COUNT+=1

# ❌ WRONG - Division by zero
RESULT=$((10 / 0))  # Error!

# ✅ CORRECT
if [ "$DIVISOR" -ne 0 ]; then
    RESULT=$((10 / DIVISOR))
else
    echo "Division by zero"
fi
```

---

## 🔒 Security Pitfalls

### **Command Injection:**

```bash
#!/bin/bash

# ❌ DANGEROUS
USER_INPUT=$1
eval "echo $USER_INPUT"  # Never use eval with user input!

# ❌ DANGEROUS
filename=$1
rm $(ls | grep "$filename")  # Command injection possible

# ✅ SAFE
filename=$1
if [[ $filename =~ ^[a-zA-Z0-9._-]+$ ]]; then
    rm "$filename"
else
    echo "Invalid filename"
fi
```

### **Path Traversal:**

```bash
#!/bin/bash

# ❌ DANGEROUS
USER_FILE=$1
cat "/var/data/$USER_FILE"  # Could access ../../../../etc/passwd

# ✅ SAFE
USER_FILE=$1
# Remove .. and /
SAFE_FILE=$(basename "$USER_FILE")
if [[ $SAFE_FILE =~ ^[a-zA-Z0-9._-]+$ ]]; then
    cat "/var/data/$SAFE_FILE"
else
    echo "Invalid filename"
fi
```

### **SQL Injection:**

```bash
#!/bin/bash

# ❌ DANGEROUS
QUERY="SELECT * FROM users WHERE name='$USER_INPUT'"
mysql -e "$QUERY"

# ✅ SAFER - Escape quotes
USER_INPUT=$(printf '%s' "$USER_INPUT" | sed "s/'/''/g")
QUERY="SELECT * FROM users WHERE name='$USER_INPUT'"

# ✅ BEST - Use parameterized queries (if available)
```

---

## 📏 Code Style Guidelines

### **Naming Conventions:**

```bash
#!/bin/bash

# Constants - UPPER_CASE
readonly MAX_RETRIES=3
readonly CONFIG_FILE="/etc/app.conf"

# Global variables - UPPER_CASE (but avoid globals)
GLOBAL_COUNTER=0

# Local variables - lower_case
function my_function() {
    local user_name="John"
    local file_count=0
}

# Functions - lower_case with underscores
function process_data() {
    echo "Processing..."
}

# Private functions - _leading_underscore
function _internal_helper() {
    echo "Internal use only"
}
```

### **Indentation and Formatting:**

```bash
#!/bin/bash

# Use 4 spaces (or 2, but be consistent)
function example() {
    if [ "$1" = "value" ]; then
        for i in {1..10}; do
            echo "$i"
        done
    fi
}

# Line length - max 80-100 characters
long_command \
    --option1 value1 \
    --option2 value2 \
    --option3 value3

# Operators spacing
COUNT=$((COUNT + 1))      # ✅ Spaces around operators
COUNT=$((COUNT+1))        # ❌ No spaces

# Function definition
function name() {         # ✅ Consistent
    # body
}

name() {                  # ✅ Also acceptable
    # body
}
```

---

## 📚 Documentation Standards

### **Comments:**

```bash
#!/bin/bash

# Good: Explain WHY, not WHAT
# We use a retry loop because the API is unreliable
for i in {1..3}; do
    if api_call; then
        break
    fi
    sleep 2
done

# Bad: Obvious comment
# Increment counter
((COUNT++))

# Good: Complex logic explanation
# Calculate days until expiry by converting timestamps to seconds,
# subtracting, and dividing by seconds per day (86400)
DAYS=$(( (EXPIRY_DATE - CURRENT_DATE) / 86400 ))

# Function documentation
##
# Backs up a directory to remote server
#
# Arguments:
#   $1 - Source directory path
#   $2 - Remote server (user@host)
#   $3 - Remote path
#
# Returns:
#   0 on success, 1 on failure
#
# Example:
#   backup_directory "/data" "user@server" "/backups"
##
backup_directory() {
    local source=$1
    local remote=$2
    local path=$3
    
    # Implementation
}
```

---

## 🎯 Shellcheck Recommendations

### **Enable Shellcheck:**

```bash
#!/bin/bash

# Install shellcheck
# macOS: brew install shellcheck
# Linux: apt-get install shellcheck

# Run shellcheck
shellcheck script.sh

# Fix common warnings
# SC2086: Quote variables
echo "$VAR"  # Not: echo $VAR

# SC2046: Quote command substitution
files="$(ls)"  # Not: files=$(ls)

# SC2181: Check exit code directly
if command; then  # Not: command; if [ $? -eq 0 ]; then

# SC2164: Use cd ... || exit
cd /path || exit

# SC2155: Separate declaration and assignment
local var
var=$(command)
# Not: local var=$(command)

# Disable specific warnings when needed
# shellcheck disable=SC2086
echo $VAR  # Intentionally unquoted
```

---

## 🎯 Quick Check: Do You Understand?

1. **Should you quote variables?**
   <details>
   <summary>Answer</summary>
   Yes, always: "$VAR" not $VAR
   </details>

2. **What's wrong with: cd /path; rm -rf *?**
   <details>
   <summary>Answer</summary>
   If cd fails, rm runs in current directory!
   </details>

3. **How to properly read lines from a file?**
   <details>
   <summary>Answer</summary>
   while IFS= read -r line; do ... done < file
   </details>

4. **Should functions use local variables?**
   <details>
   <summary>Answer</summary>
   Yes, always use local to avoid polluting global scope
   </details>

5. **Is eval safe with user input?**
   <details>
   <summary>Answer</summary>
   No! Never use eval with untrusted input
   </details>

---

## 📝 Key Takeaways

✅ **Always quote variables**  
✅ **Check exit codes**  
✅ **Use set -euo pipefail**  
✅ **Use local variables in functions**  
✅ **Never eval user input**  
✅ **Use [[ ]] instead of [ ]**  
✅ **Handle spaces in filenames**  
✅ **Run shellcheck on all scripts**  
✅ **Document complex logic**  
✅ **Follow consistent style**  

---

## 🚀 Next Steps

You now write professional shell scripts!

**Final lesson:** [31 - Cheat Sheet](31-cheat-sheet.md) - Quick reference for everything

---

## 💡 Pro Tips

**Use a linter:**
```bash
shellcheck -x script.sh  # Check with source
```

**Template for new scripts:**
```bash
cp template.sh new-script.sh
vim new-script.sh
```

**Pre-commit hook:**
```bash
#!/bin/sh
shellcheck *.sh || exit 1
```

Quality over speed! 🎯
