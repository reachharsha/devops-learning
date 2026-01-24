---
render_with_liquid: false
---
# 08 - User Input & Output

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Reading user input with `read`
- Output with `echo`, `printf`, and `cat`
- Command-line arguments ($1, $2, $@, $*)
- Here documents and here strings
- Input validation and error handling
- Creating interactive scripts
- Colored output and formatting

---

## ⌨️ Reading User Input

### **Basic `read` Command:**

```bash
#!/bin/bash

# Simple input
read NAME
echo "Hello, $NAME!"

# With prompt
read -p "Enter your name: " NAME
echo "Hello, $NAME!"

# Multiple variables
read -p "Enter first and last name: " FIRST LAST
echo "First: $FIRST, Last: $LAST"

# Read entire line
read -p "Enter a sentence: " SENTENCE
echo "You said: $SENTENCE"
```

### **`read` Options:**

```bash
# -p: Prompt
read -p "Enter name: " NAME

# -s: Silent (for passwords)
read -sp "Enter password: " PASSWORD
echo  # New line after silent input

# -n: Read N characters (no Enter needed)
read -n1 -p "Press any key to continue..."

# -t: Timeout in seconds
if read -t 5 -p "Enter name (5 sec): " NAME; then
    echo "Hello, $NAME!"
else
    echo "Too slow!"
fi

# -r: Raw mode (don't interpret backslashes)
read -r LINE  # Won't interpret \n, \t, etc.

# -a: Read into array
read -a ARRAY -p "Enter numbers: "
echo "First: ${ARRAY[0]}, Second: ${ARRAY[1]}"

# -d: Custom delimiter
read -d ',' -p "Enter until comma: " INPUT
```

---

## 📤 Output: echo vs printf

### **echo Command:**

```bash
# Basic echo
echo "Hello, World!"

# Multiple arguments
echo Hello World        # Hello World

# Variables
NAME="Alice"
echo "Hello, $NAME"

# Without newline (-n)
echo -n "Loading..."
sleep 2
echo " Done!"

# Escape sequences (-e)
echo -e "Line 1\nLine 2"         # Newline
echo -e "Name:\tAlice"            # Tab
echo -e "Red\rGreen"              # Carriage return
echo -e "\aBeep"                  # Alert (beep)

# Multiple lines
echo "Line 1"
echo "Line 2"
echo "Line 3"
```

### **printf Command (More Control):**

```bash
# Basic printf (like C printf)
printf "Hello, World!\n"

# Format strings
printf "Name: %s\n" "Alice"
printf "Age: %d\n" 25
printf "Price: %.2f\n" 19.99

# Multiple values
printf "%-10s %-5d %.2f\n" "Alice" 25 19.99
printf "%-10s %-5d %.2f\n" "Bob" 30 25.50

# Formatted table
printf "%-15s %-10s %10s\n" "Name" "Age" "Salary"
printf "%-15s %-10s %10s\n" "Alice" "25" "50000"
printf "%-15s %-10s %10s\n" "Bob" "30" "60000"

# Padding and alignment
printf "%10s\n" "Right"          #      Right
printf "%-10s\n" "Left"          # Left      
printf "%010d\n" 42              # 0000000042

# Hexadecimal and octal
printf "Hex: %x\n" 255           # ff
printf "Octal: %o\n" 8           # 10
```

### **echo vs printf:**

```bash
# echo: Simple, adds newline automatically
echo "Hello"                     # ✅ Simple

# printf: Precise control, no automatic newline
printf "Hello\n"                 # ✅ More control

# Use echo for simple output
# Use printf for formatted output (tables, numbers, etc.)
```

---

## 🎯 Command-Line Arguments

### **Positional Parameters:**

```bash
#!/bin/bash

# $0 = Script name
# $1 = First argument
# $2 = Second argument
# ...
# $9 = Ninth argument
# ${10} = Tenth argument (use braces for > 9)

echo "Script name: $0"
echo "First argument: $1"
echo "Second argument: $2"
echo "Third argument: $3"

# Run: ./script.sh arg1 arg2 arg3
# Output:
# Script name: ./script.sh
# First argument: arg1
# Second argument: arg2
# Third argument: arg3
```

### **Special Parameters:**

```bash
#!/bin/bash

# $# = Number of arguments
echo "Number of arguments: $#"

# $@ = All arguments as separate words
echo "All arguments (@): $@"

# $* = All arguments as single word
echo "All arguments (*): $*"

# $? = Exit code of last command
echo "test"
echo "Exit code: $?"              # 0 (success)

# $$ = Current process ID
echo "Script PID: $$"

# $! = PID of last background process
sleep 100 &
echo "Background PID: $!"
```

### **$@ vs $*:**

```bash
#!/bin/bash

# Test difference
show_args() {
    echo "Number of args: $#"
    for arg in "$@"; do
        echo "  - $arg"
    done
}

# With "$@" - preserves individual arguments
show_args "$@"

# Run: ./script.sh "arg 1" "arg 2"
# Output:
# Number of args: 2
#   - arg 1
#   - arg 2

# With "$*" - combines into one
show_args "$*"
# Output:
# Number of args: 1
#   - arg 1 arg 2

# Always use "$@" to preserve arguments!
```

### **shift Command:**

```bash
#!/bin/bash

echo "First: $1"
shift                    # Shift arguments left
echo "First after shift: $1"

# Process all arguments
while [[ $# -gt 0 ]]; do
    echo "Processing: $1"
    shift
done
```

---

## 📋 Here Documents

### **Basic Here Document:**

```bash
#!/bin/bash

# Create multi-line content
cat << EOF
This is line 1
This is line 2
Variables work: $USER
This is line 4
EOF

# Write to file
cat << EOF > output.txt
Line 1
Line 2
Line 3
EOF

# With variable expansion
NAME="Alice"
cat << EOF
Hello, $NAME!
Welcome to bash scripting.
Current date: $(date)
EOF
```

### **Quoted Here Document (No Expansion):**

```bash
#!/bin/bash

# Use 'EOF' to prevent variable expansion
cat << 'EOF'
This is $USER
This is $(date)
These won't be expanded!
EOF

# Useful for scripts that contain $
cat << 'EOF' > script.sh
#!/bin/bash
echo "User: $USER"
EOF
```

### **Here Document with Indentation:**

```bash
#!/bin/bash

# Use <<- to ignore leading tabs
if true; then
    cat <<- EOF
	This is indented in source
	But appears left-aligned
	EOF
fi
```

### **Here String (<<<):**

```bash
#!/bin/bash

# Feed string directly to command
grep "pattern" <<< "text to search in pattern here"

# With variable
TEXT="Hello World"
wc -w <<< "$TEXT"        # Count words

# Multiple lines
tr ' ' '\n' <<< "one two three four"
```

---

## ✅ Input Validation

### **Check Number of Arguments:**

```bash
#!/bin/bash

if [[ $# -ne 2 ]]; then
    echo "Usage: $0 <source> <destination>"
    exit 1
fi

SOURCE=$1
DEST=$2

echo "Copying $SOURCE to $DEST"
```

### **Validate Input Type:**

```bash
#!/bin/bash

read -p "Enter a number: " NUM

# Check if numeric
if ! [[ "$NUM" =~ ^[0-9]+$ ]]; then
    echo "Error: Not a valid number"
    exit 1
fi

echo "Valid number: $NUM"
```

### **Validate File Existence:**

```bash
#!/bin/bash

FILE=$1

if [[ ! -f "$FILE" ]]; then
    echo "Error: File '$FILE' does not exist"
    exit 1
fi

echo "Processing $FILE..."
```

### **Menu with Validation:**

```bash
#!/bin/bash

while true; do
    echo "Select an option:"
    echo "1) Option 1"
    echo "2) Option 2"
    echo "3) Exit"
    
    read -p "Enter choice [1-3]: " CHOICE
    
    case $CHOICE in
        1)
            echo "You selected Option 1"
            ;;
        2)
            echo "You selected Option 2"
            ;;
        3)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid choice. Please try again."
            ;;
    esac
    echo ""
done
```

---

## 🎨 Colored Output

### **ANSI Color Codes:**

```bash
#!/bin/bash

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
MAGENTA='\033[0;35m'
CYAN='\033[0;36m'
NC='\033[0m'  # No Color

# Usage
echo -e "${RED}This is red${NC}"
echo -e "${GREEN}This is green${NC}"
echo -e "${YELLOW}This is yellow${NC}"
echo -e "${BLUE}This is blue${NC}"

# Bold colors
BOLD_RED='\033[1;31m'
BOLD_GREEN='\033[1;32m'

echo -e "${BOLD_RED}Bold red${NC}"
echo -e "${BOLD_GREEN}Bold green${NC}"

# Background colors
BG_RED='\033[41m'
BG_GREEN='\033[42m'

echo -e "${BG_RED}Red background${NC}"
echo -e "${BG_GREEN}Green background${NC}"
```

### **Practical Status Messages:**

```bash
#!/bin/bash

# Define colors
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

# Status functions
success() {
    echo -e "${GREEN}✓${NC} $1"
}

error() {
    echo -e "${RED}✗${NC} $1"
}

warning() {
    echo -e "${YELLOW}!${NC} $1"
}

info() {
    echo -e "${CYAN}ℹ${NC} $1"
}

# Usage
success "Operation completed"
error "Operation failed"
warning "Low disk space"
info "Processing file..."
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you read user input silently?**
   <details>
   <summary>Answer</summary>
   read -sp "Enter password: " PASSWORD
   </details>

2. **What's the difference between $@ and $*?**
   <details>
   <summary>Answer</summary>
   "$@" preserves individual args, "$*" combines into one
   </details>

3. **How do you print formatted numbers?**
   <details>
   <summary>Answer</summary>
   printf "%.2f\n" 19.99
   </details>

4. **What is $#?**
   <details>
   <summary>Answer</summary>
   Number of command-line arguments
   </details>

5. **How do you create multi-line text?**
   <details>
   <summary>Answer</summary>
   cat << EOF ... EOF
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Interactive Script**

```bash
#!/bin/bash

echo "=== User Information ==="
read -p "Enter your name: " NAME
read -p "Enter your age: " AGE
read -sp "Enter password: " PASSWORD
echo

echo ""
echo "Name: $NAME"
echo "Age: $AGE"
echo "Password: ********"
```

### **Exercise 2: Calculator with Arguments**

```bash
#!/bin/bash

if [[ $# -ne 3 ]]; then
    echo "Usage: $0 <num1> <operator> <num2>"
    echo "Example: $0 10 + 5"
    exit 1
fi

NUM1=$1
OP=$2
NUM2=$3

case $OP in
    +)
        RESULT=$(( NUM1 + NUM2 ))
        ;;
    -)
        RESULT=$(( NUM1 - NUM2 ))
        ;;
    \*)
        RESULT=$(( NUM1 * NUM2 ))
        ;;
    /)
        RESULT=$(( NUM1 / NUM2 ))
        ;;
    *)
        echo "Invalid operator: $OP"
        exit 1
        ;;
esac

echo "$NUM1 $OP $NUM2 = $RESULT"
```

### **Exercise 3: Formatted Table**

```bash
#!/bin/bash

printf "%-15s %-10s %10s\n" "Product" "Quantity" "Price"
printf "%-15s %-10s %10s\n" "-------" "--------" "-----"
printf "%-15s %-10d %10.2f\n" "Apple" 10 1.99
printf "%-15s %-10d %10.2f\n" "Banana" 5 0.99
printf "%-15s %-10d %10.2f\n" "Orange" 8 1.49
```

### **Exercise 4: Configuration Generator**

```bash
#!/bin/bash

read -p "Enter server name: " SERVER
read -p "Enter port: " PORT
read -p "Enter database name: " DB

cat << EOF > config.ini
[server]
name = $SERVER
port = $PORT

[database]
name = $DB
host = localhost

[general]
created = $(date)
EOF

echo "Configuration file created: config.ini"
cat config.ini
```

---

## 📝 Key Takeaways

✅ **read** for user input  
✅ **read -p** for prompt  
✅ **read -s** for passwords  
✅ **echo** for simple output  
✅ **printf** for formatted output  
✅ **$1, $2** for arguments  
✅ **$@** for all arguments  
✅ **$#** for argument count  
✅ **<< EOF** for here documents  
✅ **Always validate input**  

---

## 🚀 Next Steps

You can now create interactive scripts!

**Next lesson:** [09-conditionals.md](09-conditionals.md) - if/else and case statements

---

## 💡 Pro Tips

**Safe Input Reading:**
```bash
# Always quote read variables
read -p "Name: " NAME
echo "Hello, $NAME"  # ✅ Quoted

# Validate before using
if [[ -z "$NAME" ]]; then
    echo "Name cannot be empty"
    exit 1
fi
```

**Better Prompts:**
```bash
# Provide defaults
read -p "Port [8080]: " PORT
PORT=${PORT:-8080}

# Show options
read -p "Continue? (y/n): " ANSWER
```

**Progress Indicators:**
```bash
# Simple spinner
spin='-\|/'
for i in {1..10}; do
    printf "\r${spin:i%4:1} Processing..."
    sleep 0.5
done
echo ""
```

Your scripts can now interact with users! 💬
