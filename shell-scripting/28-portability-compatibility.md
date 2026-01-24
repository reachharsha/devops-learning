# 28 - Portability and Compatibility

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- POSIX compliance for maximum portability
- Differences between shells (bash, sh, zsh, dash)
- Cross-platform scripting (Linux, macOS, BSD)
- Detecting and handling OS differences
- Writing portable scripts
- Avoiding bashisms

---

## 🌍 Shell Differences

### **Common Shells:**

| Shell | Path | POSIX | Features |
|-------|------|-------|----------|
| sh | /bin/sh | ✅ Yes | Minimal, standard |
| bash | /bin/bash | Mostly | Arrays, [[]], extended |
| dash | /bin/dash | ✅ Yes | Fast, minimal |
| zsh | /bin/zsh | Mostly | Advanced features |
| ksh | /bin/ksh | ✅ Yes | Korn shell |

### **Check Current Shell:**

```bash
#!/bin/sh

# Method 1: Check $0
echo "Shell: $0"

# Method 2: Check $SHELL
echo "Shell: $SHELL"

# Method 3: ps command
ps -p $$

# Method 4: Check specific shell
if [ -n "$BASH_VERSION" ]; then
    echo "Running in Bash $BASH_VERSION"
elif [ -n "$ZSH_VERSION" ]; then
    echo "Running in Zsh $ZSH_VERSION"
else
    echo "Running in unknown shell"
fi
```

---

## ✅ POSIX Compliance

### **POSIX-Compliant Script:**

```bash
#!/bin/sh

# ✅ POSIX compliant
# Works on: sh, bash, dash, zsh, ksh

# Variables
NAME="John"
COUNT=0

# Arithmetic
COUNT=$((COUNT + 1))

# Conditionals
if [ "$NAME" = "John" ]; then
    echo "Hello, John"
fi

# Loops
for file in *.txt; do
    echo "$file"
done

# Functions
greet() {
    echo "Hello, $1"
}

greet "$NAME"

# Case statement
case "$1" in
    start)
        echo "Starting..."
        ;;
    stop)
        echo "Stopping..."
        ;;
    *)
        echo "Unknown command"
        ;;
esac
```

### **Common Bashisms to Avoid:**

```bash
#!/bin/bash

# ❌ Bashisms (NOT portable)

# [[ ]] - bash specific
[[ $var = "value" ]]

# Arrays
arr=(1 2 3)
echo "${arr[@]}"

# [[...]] conditionals
[[ $var =~ regex ]]

# Process substitution
diff <(cmd1) <(cmd2)

# Brace expansion
echo {1..10}

# $'' quoting
echo $'Hello\nWorld'

# &> redirection
command &> file


# ✅ POSIX alternatives

# Use [ ] instead
[ "$var" = "value" ]

# Use set -- for positional parameters
set -- 1 2 3

# Use grep with regex
echo "$var" | grep -E "regex"

# Use temporary files
cmd1 > /tmp/out1
cmd2 > /tmp/out2
diff /tmp/out1 /tmp/out2

# Use seq
for i in $(seq 1 10); do echo "$i"; done

# Use printf
printf "Hello\nWorld\n"

# Use separate redirections
command > file 2>&1
```

---

## 🖥️ Operating System Detection

### **Detect OS:**

```bash
#!/bin/sh

detect_os() {
    case "$(uname -s)" in
        Linux*)
            OS="Linux"
            ;;
        Darwin*)
            OS="macOS"
            ;;
        FreeBSD*)
            OS="FreeBSD"
            ;;
        OpenBSD*)
            OS="OpenBSD"
            ;;
        SunOS*)
            OS="Solaris"
            ;;
        *)
            OS="Unknown"
            ;;
    esac
    
    echo "$OS"
}

# Usage
OS=$(detect_os)
echo "Operating System: $OS"

# OS-specific commands
case "$OS" in
    Linux)
        PACKAGE_MANAGER="apt-get"  # or yum, dnf
        ;;
    macOS)
        PACKAGE_MANAGER="brew"
        ;;
    FreeBSD)
        PACKAGE_MANAGER="pkg"
        ;;
esac
```

### **Detect Linux Distribution:**

```bash
#!/bin/sh

detect_distro() {
    if [ -f /etc/os-release ]; then
        # shellcheck disable=SC1091
        . /etc/os-release
        echo "$ID"
    elif [ -f /etc/redhat-release ]; then
        echo "rhel"
    elif [ -f /etc/debian_version ]; then
        echo "debian"
    else
        echo "unknown"
    fi
}

DISTRO=$(detect_distro)
echo "Distribution: $DISTRO"

# Distro-specific package manager
case "$DISTRO" in
    ubuntu|debian)
        PKG_MANAGER="apt-get"
        PKG_INSTALL="apt-get install -y"
        ;;
    centos|rhel|fedora)
        PKG_MANAGER="yum"
        PKG_INSTALL="yum install -y"
        ;;
    arch)
        PKG_MANAGER="pacman"
        PKG_INSTALL="pacman -S --noconfirm"
        ;;
esac
```

---

## 🔧 Platform-Specific Commands

### **Cross-Platform Equivalents:**

```bash
#!/bin/sh

# Detect OS first
OS=$(uname -s)

# sed in-place editing
case "$OS" in
    Darwin*)
        # macOS requires empty string after -i
        sed -i '' 's/old/new/' file.txt
        ;;
    *)
        # Linux
        sed -i 's/old/new/' file.txt
        ;;
esac

# stat command
get_file_size() {
    local file=$1
    case "$OS" in
        Darwin*)
            stat -f %z "$file"
            ;;
        *)
            stat -c %s "$file"
            ;;
    esac
}

# date command
get_date_yesterday() {
    case "$OS" in
        Darwin*)
            date -v-1d '+%Y-%m-%d'
            ;;
        *)
            date -d '1 day ago' '+%Y-%m-%d'
            ;;
    esac
}

# readlink
get_real_path() {
    local file=$1
    case "$OS" in
        Darwin*)
            # macOS doesn't have readlink -f
            python -c "import os; print(os.path.realpath('$file'))"
            ;;
        *)
            readlink -f "$file"
            ;;
    esac
}

# mktemp
create_temp_file() {
    case "$OS" in
        Darwin*)
            mktemp /tmp/tempfile.XXXXXX
            ;;
        *)
            mktemp
            ;;
    esac
}

# ps command
get_process_list() {
    case "$OS" in
        Darwin*|*BSD*)
            ps aux
            ;;
        *)
            ps -ef
            ;;
    esac
}
```

---

## 📦 Package Management Abstraction

### **Universal Package Installer:**

```bash
#!/bin/sh

install_package() {
    local package=$1
    
    # Detect package manager
    if command -v apt-get >/dev/null 2>&1; then
        sudo apt-get update
        sudo apt-get install -y "$package"
    elif command -v yum >/dev/null 2>&1; then
        sudo yum install -y "$package"
    elif command -v dnf >/dev/null 2>&1; then
        sudo dnf install -y "$package"
    elif command -v brew >/dev/null 2>&1; then
        brew install "$package"
    elif command -v pkg >/dev/null 2>&1; then
        sudo pkg install -y "$package"
    elif command -v pacman >/dev/null 2>&1; then
        sudo pacman -S --noconfirm "$package"
    else
        echo "No supported package manager found"
        return 1
    fi
}

# Usage
install_package "curl"
```

---

## 🎯 Portable Script Template

### **Maximum Portability Template:**

```bash
#!/bin/sh

###############################################################################
# Portable Script Template
# Compatible with: sh, bash, dash, zsh, ksh
# OS Support: Linux, macOS, BSD
###############################################################################

# Strict mode (POSIX)
set -eu

#------------------------------------------------------------------------------
# CONFIGURATION
#------------------------------------------------------------------------------

SCRIPT_NAME="$(basename "$0")"
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"

#------------------------------------------------------------------------------
# PLATFORM DETECTION
#------------------------------------------------------------------------------

detect_platform() {
    OS=$(uname -s)
    ARCH=$(uname -m)
    
    case "$OS" in
        Linux*)   OS_TYPE="Linux" ;;
        Darwin*)  OS_TYPE="macOS" ;;
        FreeBSD*) OS_TYPE="FreeBSD" ;;
        *)        OS_TYPE="Unknown" ;;
    esac
    
    # Export for use in functions
    export OS_TYPE
}

#------------------------------------------------------------------------------
# UTILITY FUNCTIONS
#------------------------------------------------------------------------------

# Logging
log() {
    printf "[%s] %s\n" "$(date '+%Y-%m-%d %H:%M:%S')" "$*"
}

error() {
    log "ERROR: $*" >&2
    exit 1
}

# Check if command exists (portable)
command_exists() {
    command -v "$1" >/dev/null 2>&1
}

# Require command
require_command() {
    if ! command_exists "$1"; then
        error "Required command not found: $1"
    fi
}

# Portable sed in-place
sed_inplace() {
    local pattern=$1
    local file=$2
    
    case "$OS_TYPE" in
        macOS)
            sed -i '' "$pattern" "$file"
            ;;
        *)
            sed -i "$pattern" "$file"
            ;;
    esac
}

# Portable realpath
get_realpath() {
    local path=$1
    
    if command_exists realpath; then
        realpath "$path"
    elif command_exists readlink; then
        readlink -f "$path" 2>/dev/null || echo "$path"
    else
        # Fallback
        ( cd "$(dirname "$path")" && pwd )/$(basename "$path")
    fi
}

# Portable mktemp
create_temp_file() {
    if command_exists mktemp; then
        mktemp
    else
        # Fallback
        local tmp="/tmp/tmp.$$"
        touch "$tmp"
        echo "$tmp"
    fi
}

#------------------------------------------------------------------------------
# MAIN FUNCTIONS
#------------------------------------------------------------------------------

main() {
    log "Script started on $OS_TYPE"
    
    # Your logic here
    
    log "Script completed"
}

#------------------------------------------------------------------------------
# ENTRY POINT
#------------------------------------------------------------------------------

# Detect platform first
detect_platform

# Run main function
main "$@"
```

---

## 🔍 Testing Across Platforms

### **Test Script:**

```bash
#!/bin/sh

# test_portability.sh
# Test script on multiple shells and platforms

SCRIPT="./myscript.sh"

echo "Testing script portability..."
echo ""

# Test with different shells
for shell in sh bash dash zsh ksh; do
    if command -v "$shell" >/dev/null 2>&1; then
        echo "Testing with $shell..."
        if "$shell" "$SCRIPT" >/dev/null 2>&1; then
            echo "✓ $shell: PASS"
        else
            echo "✗ $shell: FAIL"
        fi
    else
        echo "- $shell: NOT INSTALLED"
    fi
done

echo ""

# Check for bashisms
if command -v checkbashisms >/dev/null 2>&1; then
    echo "Checking for bashisms..."
    checkbashisms "$SCRIPT"
else
    echo "Install 'devscripts' for bashism checking"
fi

# Shellcheck
if command -v shellcheck >/dev/null 2>&1; then
    echo ""
    echo "Running shellcheck..."
    shellcheck -s sh "$SCRIPT"
fi
```

---

## 🎯 Common Portability Pitfalls

### **Avoid These:**

```bash
#!/bin/sh

# ❌ WRONG: Bashisms

# Don't use [[ ]]
[[ $var = "value" ]]

# Don't use arrays
arr=(1 2 3)

# Don't use $(( )) for strings
result=$((var + 1))  # OK for numbers only

# Don't use =~ operator
[[ $var =~ regex ]]

# Don't use {1..10}
for i in {1..10}; do echo $i; done

# Don't use $'' quoting
echo $'line1\nline2'


# ✅ CORRECT: POSIX compliant

# Use [ ]
[ "$var" = "value" ]

# Use positional parameters
set -- 1 2 3
echo "$1 $2 $3"

# Use expr or $(( ))
result=$(expr "$var" + 1)
result=$((var + 1))  # For numbers

# Use grep
echo "$var" | grep -E "regex"

# Use seq
for i in $(seq 1 10); do echo "$i"; done

# Use printf
printf "line1\nline2\n"
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the most portable shebang?**
   <details>
   <summary>Answer</summary>
   #!/bin/sh
   </details>

2. **What's a bashism?**
   <details>
   <summary>Answer</summary>
   Bash-specific feature not available in POSIX sh
   </details>

3. **How to detect the OS in a portable way?**
   <details>
   <summary>Answer</summary>
   uname -s
   </details>

4. **What's the portable test command?**
   <details>
   <summary>Answer</summary>
   [ ] not [[ ]]
   </details>

5. **How to check if a command exists portably?**
   <details>
   <summary>Answer</summary>
   command -v cmd >/dev/null 2>&1
   </details>

---

## 📝 Key Takeaways

✅ **Use #!/bin/sh for maximum portability**  
✅ **Avoid bashisms: [[]], arrays, =~**  
✅ **Use [ ] instead of [[ ]]**  
✅ **Detect OS and handle differences**  
✅ **Test on multiple shells**  
✅ **Use command -v to check commands**  
✅ **Handle platform-specific sed/date**  
✅ **Use POSIX-compliant syntax**  

---

## 🚀 Next Steps

Your scripts now run anywhere!

**Next lesson:** [29 - Advanced Topics](29-advanced-topics.md) - Deep shell techniques

---

## 💡 Pro Tips

**Test portability:**
```bash
# Install checkbashisms
apt-get install devscripts

# Check script
checkbashisms script.sh
```

**Shebang choice:**
```bash
#!/bin/sh           # Maximum portability
#!/bin/bash         # Bash features OK
#!/usr/bin/env bash # Find bash in PATH
```

**Document requirements:**
```bash
# Requirements:
# - POSIX sh
# - GNU coreutils (or BSD equivalents)
# - curl
```

Portable = universal! 🌍
