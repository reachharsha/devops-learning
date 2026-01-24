# 20 - Advanced Techniques

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- getopts for command-line option parsing
- Configuration file handling
- Debugging techniques
- Performance optimization
- Script templates and best practices
- Shell options and modes

---

## ⚙️ getopts - Command-Line Options

### **Basic getopts Usage:**

```bash
#!/bin/bash

# Simple options: -a -b -c
while getopts "abc" opt; do
    case $opt in
        a) echo "Option -a specified";;
        b) echo "Option -b specified";;
        c) echo "Option -c specified";;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1;;
    esac
done

# Usage: ./script.sh -a -b -c
```

### **Options with Arguments:**

```bash
#!/bin/bash

# Options with arguments: -f file -n number
while getopts "f:n:h" opt; do
    case $opt in
        f)
            FILE=$OPTARG
            echo "File: $FILE"
            ;;
        n)
            NUMBER=$OPTARG
            echo "Number: $NUMBER"
            ;;
        h)
            echo "Usage: $0 [-f file] [-n number] [-h]"
            exit 0
            ;;
        :)
            echo "Option -$OPTARG requires an argument" >&2
            exit 1
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
    esac
done

# Shift processed options
shift $((OPTIND-1))

# Remaining arguments
echo "Remaining args: $@"

# Usage: ./script.sh -f myfile.txt -n 42 arg1 arg2
```

### **Complete getopts Example:**

```bash
#!/bin/bash

# Default values
VERBOSE=false
OUTPUT=""
COUNT=1

usage() {
    cat << EOF
Usage: $0 [OPTIONS] [ARGUMENTS]

Options:
    -v          Verbose mode
    -o FILE     Output file
    -c COUNT    Number of iterations
    -h          Show this help

Examples:
    $0 -v -o output.txt -c 10 input.txt
    $0 -h
EOF
    exit 0
}

# Parse options
while getopts "vo:c:h" opt; do
    case $opt in
        v) VERBOSE=true;;
        o) OUTPUT=$OPTARG;;
        c) COUNT=$OPTARG;;
        h) usage;;
        :) echo "Option -$OPTARG requires an argument" >&2; exit 1;;
        \?) echo "Invalid option: -$OPTARG" >&2; exit 1;;
    esac
done

shift $((OPTIND-1))

# Validate
if [ -z "$OUTPUT" ]; then
    echo "Error: Output file required (-o)" >&2
    exit 1
fi

# Use options
$VERBOSE && echo "Verbose mode enabled"
echo "Output file: $OUTPUT"
echo "Count: $COUNT"
echo "Arguments: $@"
```

---

## 📝 Configuration Files

### **Reading INI-Style Config:**

```bash
#!/bin/bash

# config.ini:
# [database]
# host=localhost
# port=5432
# user=admin
#
# [app]
# debug=true
# timeout=30

read_config() {
    local config_file=$1
    local section=""
    
    while IFS='=' read -r key value; do
        # Remove whitespace
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        # Skip empty lines and comments
        [[ -z $key || $key =~ ^# ]] && continue
        
        # Section header
        if [[ $key =~ ^\[(.*)\]$ ]]; then
            section="${BASH_REMATCH[1]}"
            continue
        fi
        
        # Create variable: SECTION_KEY=value
        local var_name="${section}_${key}"
        declare -g "$var_name=$value"
        
    done < "$config_file"
}

# Load config
read_config "config.ini"

# Use config
echo "Database host: $database_host"
echo "Database port: $database_port"
echo "App debug: $app_debug"
```

### **Key-Value Config:**

```bash
#!/bin/bash

# config.conf:
# PORT=8080
# HOST=localhost
# DEBUG=true

load_config() {
    local config_file=$1
    
    if [ ! -f "$config_file" ]; then
        echo "Config file not found: $config_file" >&2
        return 1
    fi
    
    # Source the config
    # shellcheck disable=SC1090
    source "$config_file"
}

# Default values
PORT=3000
HOST="0.0.0.0"
DEBUG=false

# Load config (overrides defaults)
load_config "config.conf" || exit 1

echo "Server: $HOST:$PORT"
echo "Debug: $DEBUG"
```

### **Associative Array Config:**

```bash
#!/bin/bash

declare -A CONFIG

load_config() {
    local config_file=$1
    
    while IFS='=' read -r key value; do
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        [[ -z $key || $key =~ ^# ]] && continue
        
        CONFIG[$key]=$value
    done < "$config_file"
}

# Usage
load_config "app.conf"

echo "Port: ${CONFIG[port]}"
echo "Host: ${CONFIG[host]}"
```

---

## 🐛 Debugging Techniques

### **Enable Debug Mode:**

```bash
#!/bin/bash

# Method 1: set -x (trace execution)
set -x          # Enable
echo "Hello"
set +x          # Disable

# Method 2: Run with bash -x
# bash -x script.sh

# Method 3: Conditional debug
DEBUG=${DEBUG:-false}

debug() {
    if $DEBUG; then
        echo "[DEBUG] $*" >&2
    fi
}

debug "This is a debug message"

# Usage: DEBUG=true ./script.sh
```

### **Debug Functions:**

```bash
#!/bin/bash

# Print variable values
debug_var() {
    local var_name=$1
    local var_value="${!var_name}"
    echo "[DEBUG] $var_name = $var_value" >&2
}

NAME="John"
debug_var NAME

# Stack trace
print_stack() {
    local frame=0
    echo "Stack trace:"
    while caller $frame; do
        ((frame++))
    done
}

func1() { func2; }
func2() { func3; }
func3() { print_stack; }
func1
```

### **Line-by-Line Debugging:**

```bash
#!/bin/bash

# Enable PS4 for better trace output
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'
set -x

# Now trace shows file, line, and function
echo "Hello"
```

### **Dry Run Mode:**

```bash
#!/bin/bash

DRY_RUN=false

execute() {
    if $DRY_RUN; then
        echo "[DRY-RUN] Would execute: $*"
    else
        "$@"
    fi
}

# Usage
execute rm -rf /tmp/data
execute mkdir -p /backups

# Run: DRY_RUN=true ./script.sh
```

---

## ⚡ Performance Optimization

### **Avoid Unnecessary Subshells:**

```bash
#!/bin/bash

# ❌ Slow (creates subshell)
COUNT=$(expr $COUNT + 1)

# ✅ Fast (bash arithmetic)
((COUNT++))

# ❌ Slow
RESULT=$(cat file.txt | grep pattern)

# ✅ Fast
RESULT=$(grep pattern file.txt)
```

### **Use Built-ins Instead of External Commands:**

```bash
#!/bin/bash

# ❌ Slow (spawns process)
BASENAME=$(basename "$FILE")
DIRNAME=$(dirname "$FILE")

# ✅ Fast (parameter expansion)
BASENAME=${FILE##*/}
DIRNAME=${FILE%/*}

# ❌ Slow
LENGTH=$(echo "$STRING" | wc -c)

# ✅ Fast
LENGTH=${#STRING}
```

### **Batch Operations:**

```bash
#!/bin/bash

# ❌ Slow (many process spawns)
for file in *.txt; do
    cat "$file" >> combined.txt
done

# ✅ Fast (single command)
cat *.txt > combined.txt

# ❌ Slow
for i in {1..1000}; do
    echo $i >> numbers.txt
done

# ✅ Fast
printf '%s\n' {1..1000} > numbers.txt
```

### **Parallel Processing:**

```bash
#!/bin/bash

# Sequential (slow)
for file in *.jpg; do
    convert "$file" "${file%.jpg}.png"
done

# Parallel (fast)
for file in *.jpg; do
    convert "$file" "${file%.jpg}.png" &
done
wait

# With limit
MAX_JOBS=4
for file in *.jpg; do
    while [ $(jobs -r | wc -l) -ge $MAX_JOBS ]; do
        sleep 0.1
    done
    convert "$file" "${file%.jpg}.png" &
done
wait
```

---

## 🔧 Shell Options

### **Useful Shell Options:**

```bash
#!/bin/bash

# Stop on error
set -e
set -o errexit

# Exit on undefined variable
set -u
set -o nounset

# Fail on pipe errors
set -o pipefail

# Print commands before execution
set -x
set -o xtrace

# Verbose mode
set -v
set -o verbose

# No clobber (prevent overwriting files)
set -C
set -o noclobber

# Disable filename expansion (globbing)
set -f
set -o noglob

# Check options
shopt -s nullglob      # Expand to nothing if no match
shopt -s dotglob       # Include hidden files in *
shopt -s extglob       # Extended pattern matching
```

---

## 📋 Script Templates

### **Minimal Template:**

```bash
#!/bin/bash
set -euo pipefail

# Script logic here
echo "Hello, World!"
```

### **Production Template:**

```bash
#!/bin/bash

###############################################################################
# Script Name: script_name.sh
# Description: What this script does
# Author: Your Name
# Date: 2024-01-15
# Version: 1.0
###############################################################################

set -euo pipefail

#------------------------------------------------------------------------------
# CONFIGURATION
#------------------------------------------------------------------------------

SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
SCRIPT_NAME=$(basename "$0")
LOG_FILE="${SCRIPT_DIR}/logs/${SCRIPT_NAME%.sh}.log"

# Defaults
VERBOSE=false
DRY_RUN=false

#------------------------------------------------------------------------------
# FUNCTIONS
#------------------------------------------------------------------------------

usage() {
    cat << EOF
Usage: $SCRIPT_NAME [OPTIONS] [ARGUMENTS]

Description:
    Detailed description of what this script does

Options:
    -v, --verbose    Enable verbose output
    -d, --dry-run    Dry run mode
    -h, --help       Show this help message

Examples:
    $SCRIPT_NAME -v input.txt
    $SCRIPT_NAME --dry-run
EOF
    exit 0
}

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    echo "[ERROR] $*" >&2
    exit 1
}

cleanup() {
    log "Cleanup..."
    # Remove temp files, etc.
}

#------------------------------------------------------------------------------
# SIGNAL HANDLERS
#------------------------------------------------------------------------------

trap cleanup EXIT
trap 'error "Script interrupted"' INT TERM

#------------------------------------------------------------------------------
# ARGUMENT PARSING
#------------------------------------------------------------------------------

while [[ $# -gt 0 ]]; do
    case $1 in
        -v|--verbose) VERBOSE=true; shift;;
        -d|--dry-run) DRY_RUN=true; shift;;
        -h|--help) usage;;
        -*) error "Unknown option: $1";;
        *) break;;
    esac
done

#------------------------------------------------------------------------------
# VALIDATION
#------------------------------------------------------------------------------

# Validate arguments
if [ $# -lt 1 ]; then
    error "Missing required argument"
fi

#------------------------------------------------------------------------------
# MAIN SCRIPT
#------------------------------------------------------------------------------

main() {
    log "Script started"
    
    # Your logic here
    
    log "Script completed successfully"
}

# Run main function
main "$@"
```

---

## 🎯 Best Practices Checklist

```bash
#!/bin/bash

# ✅ Use strict mode
set -euo pipefail

# ✅ Add shebang
#!/bin/bash

# ✅ Quote variables
echo "$VAR"

# ✅ Use local in functions
func() {
    local var="value"
}

# ✅ Check commands exist
command -v git &> /dev/null || error "git not installed"

# ✅ Validate input
[ $# -eq 0 ] && usage

# ✅ Use meaningful names
USER_COUNT=10  # Good
uc=10          # Bad

# ✅ Add comments for complex logic
# Calculate days until expiry
DAYS=$(( (EXPIRY - NOW) / 86400 ))

# ✅ Use functions for reusable code
validate_email() { ... }

# ✅ Handle errors
command || error "Command failed"

# ✅ Clean up resources
trap cleanup EXIT

# ✅ Log important events
log "Processing started"

# ✅ Use shellcheck
# shellcheck script.sh
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you parse command-line options?**
   <details>
   <summary>Answer</summary>
   Use getopts in a while loop
   </details>

2. **How do you enable debug tracing?**
   <details>
   <summary>Answer</summary>
   set -x or bash -x script.sh
   </details>

3. **What's faster: $(basename file) or ${file##*/}?**
   <details>
   <summary>Answer</summary>
   ${file##*/} (no subprocess)
   </details>

4. **How do you make a script fail on pipe errors?**
   <details>
   <summary>Answer</summary>
   set -o pipefail
   </details>

5. **How do you get the script's directory?**
   <details>
   <summary>Answer</summary>
   SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
   </details>

---

## 📝 Key Takeaways

✅ **Use getopts for option parsing**  
✅ **Source config files for settings**  
✅ **set -x for debugging**  
✅ **Avoid subshells for performance**  
✅ **Use built-ins over external commands**  
✅ **Always include cleanup traps**  
✅ **Follow consistent template structure**  

---

## 🚀 Next Steps

You've mastered advanced techniques!

**Next lesson:** [21 - Regular Expressions](21-regular-expressions.md) - Deep dive into regex patterns

---

## 💡 Pro Tips

**Benchmark commands:**
```bash
time command    # See execution time
```

**Profile script:**
```bash
PS4='+ $(date "+%s.%N")\011 '
set -x
# Your code
set +x
```

**Portable shebang:**
```bash
#!/usr/bin/env bash
```

Advanced techniques = professional scripts! 🚀
