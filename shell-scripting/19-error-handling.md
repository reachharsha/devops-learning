---
render_with_liquid: false
---
# 19 - Error Handling

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Exit codes and the $? variable
- set options for safer scripts (set -e, set -u, set -o pipefail)
- Error checking patterns
- Defensive programming
- Validation and input sanitization
- Logging errors
- Try-catch equivalent in bash

---

## 🔢 Exit Codes

### **Understanding Exit Codes:**

```bash
#!/bin/bash

# 0 = success
# 1-255 = error

# Check last command's exit code
ls /etc/passwd
echo $?         # 0 (success)

ls /nonexistent
echo $?         # 2 (error)

# Set exit code
exit 0          # Success
exit 1          # General error
exit 2          # Misuse of command
exit 127        # Command not found
exit 130        # Terminated by Ctrl+C
```

### **Common Exit Codes:**

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | General error |
| 2 | Misuse of shell builtin |
| 126 | Command cannot execute |
| 127 | Command not found |
| 128 | Invalid exit argument |
| 130 | Terminated by Ctrl+C |
| 255 | Exit status out of range |

### **Checking Exit Codes:**

```bash
#!/bin/bash

# Method 1: Immediate check
ls /etc/passwd
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi

# Method 2: Direct conditional (better)
if ls /etc/passwd; then
    echo "Success"
else
    echo "Failed"
fi

# Method 3: Short-circuit operators
ls /etc/passwd && echo "Success"
ls /nonexistent || echo "Failed"

# Method 4: Store exit code
ls /etc/passwd
EXIT_CODE=$?
if [ $EXIT_CODE -ne 0 ]; then
    echo "Command failed with exit code: $EXIT_CODE"
    exit $EXIT_CODE
fi
```

---

## 🛡️ Defensive Programming with set

### **set -e (Exit on Error):**

```bash
#!/bin/bash
set -e

# Script exits immediately if any command fails
echo "Step 1"
false           # This fails, script exits here
echo "Step 2" # Never executed

# Disable temporarily
set +e
false           # Won't exit
set -e

# Better pattern: check critical commands
if ! critical_command; then
    echo "Critical command failed"
    exit 1
fi
```

### **set -u (Exit on Undefined Variable):**

```bash
#!/bin/bash
set -u

# Script exits if using undefined variable
echo $DEFINED_VAR    # Error: variable not set

# Safe way with default
echo ${UNDEFINED_VAR:-"default value"}

# Check if variable is set
if [ -z "${VAR:-}" ]; then
    echo "VAR is not set"
fi
```

### **set -o pipefail (Pipe Failure Detection):**

```bash
#!/bin/bash
set -o pipefail

# Without pipefail
false | true
echo $?          # 0 (only checks last command)

# With pipefail
set -o pipefail
false | true
echo $?          # 1 (detects failure in pipeline)

# Practical example
set -o pipefail
cat nonexistent.txt | grep "pattern" | wc -l
# Exits if cat fails, even though wc would succeed
```

### **Combining set Options (Best Practice):**

```bash
#!/bin/bash

# Strict mode (recommended for production scripts)
set -euo pipefail

# Or using shorthand
set -Eeuo pipefail

# With trap for better error messages
set -Eeuo pipefail
trap 'echo "Error on line $LINENO"' ERR

echo "Script running in strict mode"
```

---

## ✅ Error Checking Patterns

### **Pattern 1: Check Before Operating:**

```bash
#!/bin/bash

FILE="/path/to/file"

# Check file exists
if [ ! -f "$FILE" ]; then
    echo "Error: File not found: $FILE"
    exit 1
fi

# Check file is readable
if [ ! -r "$FILE" ]; then
    echo "Error: File not readable: $FILE"
    exit 1
fi

# Now safe to process
cat "$FILE"
```

### **Pattern 2: Validate Input:**

```bash
#!/bin/bash

validate_email() {
    local email=$1
    
    if [[ ! $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        echo "Error: Invalid email format"
        return 1
    fi
    
    return 0
}

# Usage
if validate_email "user@example.com"; then
    echo "Email is valid"
else
    echo "Email is invalid"
    exit 1
fi
```

### **Pattern 3: Check Command Availability:**

```bash
#!/bin/bash

# Check if command exists
if ! command -v docker &> /dev/null; then
    echo "Error: docker is not installed"
    exit 1
fi

# Alternative
require_command() {
    if ! command -v "$1" &> /dev/null; then
        echo "Error: Required command '$1' not found"
        exit 1
    fi
}

require_command "git"
require_command "curl"
require_command "jq"
```

### **Pattern 4: Validate Arguments:**

```bash
#!/bin/bash

# Check argument count
if [ $# -ne 2 ]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi

SOURCE=$1
DEST=$2

# Validate source
if [ ! -d "$SOURCE" ]; then
    echo "Error: Source directory does not exist: $SOURCE"
    exit 1
fi

# Validate destination
if [ -e "$DEST" ]; then
    echo "Error: Destination already exists: $DEST"
    exit 1
fi
```

---

## 🚨 Error Handling Functions

### **Custom Error Handler:**

```bash
#!/bin/bash

error() {
    echo "[ERROR] $*" >&2
    exit 1
}

warn() {
    echo "[WARN] $*" >&2
}

info() {
    echo "[INFO] $*"
}

# Usage
info "Starting process"
[ -f "config.txt" ] || error "Config file not found"
warn "This is a warning"
```

### **Try-Catch Pattern:**

```bash
#!/bin/bash

# Simulate try-catch
try() {
    "$@"
    return $?
}

catch() {
    local exit_code=$?
    if [ $exit_code -ne 0 ]; then
        "$@" $exit_code
    fi
    return $exit_code
}

# Error handler
handle_error() {
    local exit_code=$1
    echo "Command failed with exit code: $exit_code"
    # Cleanup here
}

# Usage
try command_that_might_fail
catch handle_error
```

### **Comprehensive Error Handler:**

```bash
#!/bin/bash

set -Eeuo pipefail

error_handler() {
    local line_no=$1
    local bash_lineno=$2
    local exit_code=$3
    
    echo "Error on line $line_no"
    echo "Exit code: $exit_code"
    echo "Stack trace:"
    local frame=0
    while caller $frame; do
        ((frame++))
    done
    
    # Cleanup
    cleanup
    
    exit $exit_code
}

cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/*.tmp
}

trap 'error_handler ${LINENO} ${BASH_LINENO} $?' ERR
trap cleanup EXIT
```

---

## 🔍 Input Validation

### **Validate Numeric Input:**

```bash
#!/bin/bash

is_number() {
    [[ $1 =~ ^[0-9]+$ ]]
}

is_integer() {
    [[ $1 =~ ^-?[0-9]+$ ]]
}

is_float() {
    [[ $1 =~ ^-?[0-9]+\.?[0-9]*$ ]]
}

# Usage
read -p "Enter a number: " NUM

if ! is_number "$NUM"; then
    echo "Error: Not a valid number"
    exit 1
fi

echo "Valid number: $NUM"
```

### **Validate File Path:**

```bash
#!/bin/bash

validate_path() {
    local path=$1
    
    # Check if path is empty
    if [ -z "$path" ]; then
        echo "Error: Path cannot be empty"
        return 1
    fi
    
    # Check for path traversal attempts
    if [[ $path =~ \.\. ]]; then
        echo "Error: Path traversal not allowed"
        return 1
    fi
    
    # Check if path is absolute (if required)
    if [[ $path != /* ]]; then
        echo "Error: Path must be absolute"
        return 1
    fi
    
    return 0
}

# Usage
PATH_INPUT="/home/user/file.txt"
if validate_path "$PATH_INPUT"; then
    echo "Path is valid"
fi
```

### **Sanitize Input:**

```bash
#!/bin/bash

sanitize_filename() {
    local filename=$1
    
    # Remove path components
    filename=$(basename "$filename")
    
    # Remove special characters
    filename=${filename//[^a-zA-Z0-9._-]/}
    
    # Limit length
    filename=${filename:0:255}
    
    echo "$filename"
}

# Usage
UNSAFE="../../etc/passwd; rm -rf /"
SAFE=$(sanitize_filename "$UNSAFE")
echo "Safe filename: $SAFE"
```

---

## 📝 Logging

### **Simple Logging:**

```bash
#!/bin/bash

LOGFILE="app.log"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOGFILE"
}

log_error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOGFILE" >&2
}

# Usage
log "Application started"
log_error "Failed to connect to database"
```

### **Advanced Logging:**

```bash
#!/bin/bash

LOGFILE="app.log"
LOGLEVEL="INFO"  # DEBUG, INFO, WARN, ERROR

log_debug() { [ "$LOGLEVEL" = "DEBUG" ] && log "DEBUG" "$@"; }
log_info()  { log "INFO" "$@"; }
log_warn()  { log "WARN" "$@" >&2; }
log_error() { log "ERROR" "$@" >&2; }

log() {
    local level=$1
    shift
    local timestamp=$(date +'%Y-%m-%d %H:%M:%S')
    local message="[$timestamp] [$level] $*"
    
    echo "$message" >> "$LOGFILE"
    
    if [ "$level" != "DEBUG" ] || [ "$LOGLEVEL" = "DEBUG" ]; then
        echo "$message"
    fi
}

# Usage
log_debug "Detailed debug info"
log_info "Process started"
log_warn "Low disk space"
log_error "Connection failed"
```

---

## 🎯 Practical Examples

### **Example 1: Robust Backup Script**

```bash
#!/bin/bash

set -euo pipefail

# Configuration
SOURCE="/home/user/documents"
BACKUP_DIR="/backups"
LOGFILE="backup.log"

# Logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOGFILE"
}

error_exit() {
    log "ERROR: $*"
    exit 1
}

# Validate
[ -d "$SOURCE" ] || error_exit "Source directory not found: $SOURCE"
[ -d "$BACKUP_DIR" ] || error_exit "Backup directory not found: $BACKUP_DIR"

# Check disk space
AVAILABLE=$(df "$BACKUP_DIR" | tail -1 | awk '{print $4}')
REQUIRED=$(du -s "$SOURCE" | awk '{print $1}')

if [ $AVAILABLE -lt $REQUIRED ]; then
    error_exit "Insufficient disk space"
fi

# Perform backup
BACKUP_FILE="$BACKUP_DIR/backup_$(date +%Y%m%d_%H%M%S).tar.gz"

log "Starting backup: $SOURCE -> $BACKUP_FILE"

if tar -czf "$BACKUP_FILE" "$SOURCE" 2>&1 | tee -a "$LOGFILE"; then
    log "Backup completed successfully"
    log "Backup size: $(du -h "$BACKUP_FILE" | cut -f1)"
else
    error_exit "Backup failed"
fi
```

### **Example 2: API Client with Retry**

```bash
#!/bin/bash

set -euo pipefail

API_URL="https://api.example.com/data"
MAX_RETRIES=3
RETRY_DELAY=2

api_call() {
    local url=$1
    local attempt=1
    
    while [ $attempt -le $MAX_RETRIES ]; do
        echo "Attempt $attempt of $MAX_RETRIES..."
        
        if response=$(curl -sf "$url" 2>&1); then
            echo "Success!"
            echo "$response"
            return 0
        else
            echo "Failed (exit code: $?)"
            
            if [ $attempt -eq $MAX_RETRIES ]; then
                echo "Max retries reached, giving up"
                return 1
            fi
            
            echo "Retrying in ${RETRY_DELAY}s..."
            sleep $RETRY_DELAY
            ((attempt++))
        fi
    done
}

# Usage
if api_call "$API_URL"; then
    echo "API call successful"
else
    echo "API call failed after $MAX_RETRIES attempts"
    exit 1
fi
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does exit code 0 mean?**
   <details>
   <summary>Answer</summary>
   Success (no error)
   </details>

2. **What does set -e do?**
   <details>
   <summary>Answer</summary>
   Exit script immediately if any command fails (returns non-zero)
   </details>

3. **How do you check the last command's exit code?**
   <details>
   <summary>Answer</summary>
   $?
   </details>

4. **What does set -u do?**
   <details>
   <summary>Answer</summary>
   Exit if using undefined variable
   </details>

5. **How do you redirect errors to a file?**
   <details>
   <summary>Answer</summary>
   command 2> errors.txt
   </details>

---

## 📝 Key Takeaways

✅ **Always check exit codes ($?)**  
✅ **Use set -euo pipefail for strict mode**  
✅ **Validate all inputs before use**  
✅ **Check file existence before operations**  
✅ **Log errors to files**  
✅ **Provide meaningful error messages**  
✅ **Clean up resources on error**  
✅ **Use trap for cleanup**  

---

## 🚀 Next Steps

You now write robust, production-ready scripts!

**Next lesson:** [20 - Advanced Techniques](20-advanced-techniques.md) - getopts, debugging, optimization

---

## 💡 Pro Tips

**Strict mode template:**
```bash
#!/bin/bash
set -euo pipefail
trap 'echo "Error on line $LINENO"' ERR
```

**Safe variable usage:**
```bash
${VAR:-default}    # Use default if unset
${VAR:?error msg}  # Error if unset
```

**Always cleanup:**
```bash
trap "rm -f /tmp/*.tmp" EXIT
```

Error handling = production readiness! 🛡️
