---
render_with_liquid: false
---
# 11 - Functions

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- How to define and call functions
- Function parameters and arguments
- Return values and exit codes
- Local vs global variables in functions
- Recursion
- Function libraries and code reuse

---

## 📦 What are Functions?

**Real-World Analogy:**  
Functions are like recipes - you define them once, then use them many times. Instead of writing the same code repeatedly, you create a function and call it whenever needed.

```bash
#!/bin/bash

# Define a function
greet() {
    echo "Hello, World!"
}

# Call the function
greet
greet
greet

# Output:
# Hello, World!
# Hello, World!
# Hello, World!
```

---

## 🎨 Function Syntax

### **Method 1: function keyword:**

```bash
#!/bin/bash

function say_hello() {
    echo "Hello from function!"
}

say_hello
```

### **Method 2: Without function keyword (POSIX, preferred):**

```bash
#!/bin/bash

say_hello() {
    echo "Hello from function!"
}

say_hello
```

### **Where to Define Functions:**

```bash
#!/bin/bash

# ✅ Define functions at top of script
greet() {
    echo "Hello!"
}

calculate() {
    echo $((5 + 3))
}

# Main script execution
greet
result=$(calculate)
echo "Result: $result"
```

---

## 📥 Function Parameters

### **Positional Parameters:**

```bash
#!/bin/bash

# Function with parameters
greet_user() {
    echo "Hello, $1!"
}

greet_user "Alice"
# Output: Hello, Alice!

greet_user "Bob"
# Output: Hello, Bob!

# Multiple parameters
full_name() {
    echo "First: $1, Last: $2"
}

full_name "John" "Doe"
# Output: First: John, Last: Doe
```

### **All Function Parameters:**

```bash
#!/bin/bash

show_args() {
    echo "Number of arguments: $#"
    echo "All arguments: $@"
    echo "All arguments (single): $*"
    echo "Function name: $FUNCNAME"
    
    echo "Arguments individually:"
    for arg in "$@"; do
        echo "  - $arg"
    done
}

show_args apple banana cherry

# Output:
# Number of arguments: 3
# All arguments: apple banana cherry
# All arguments (single): apple banana cherry
# Function name: show_args
# Arguments individually:
#   - apple
#   - banana
#   - cherry
```

### **Parameter Validation:**

```bash
#!/bin/bash

create_user() {
    # Check if parameters provided
    if [ $# -ne 2 ]; then
        echo "Error: Usage: create_user <username> <email>"
        return 1
    fi
    
    local username=$1
    local email=$2
    
    echo "Creating user: $username"
    echo "Email: $email"
    # ... user creation logic ...
}

# Correct usage
create_user "john" "john@example.com"

# Incorrect usage
create_user "john"
# Output: Error: Usage: create_user <username> <email>
```

---

## 🔄 Return Values

### **Return Codes (0-255):**

```bash
#!/bin/bash

# Return 0 for success
is_even() {
    local num=$1
    if (( num % 2 == 0 )); then
        return 0    # Success/True
    else
        return 1    # Failure/False
    fi
}

# Use in conditions
if is_even 4; then
    echo "4 is even"
fi

if ! is_even 3; then
    echo "3 is not even"
fi

# Check return code
is_even 10
if [ $? -eq 0 ]; then
    echo "Function returned success"
fi
```

### **Returning Data (echo/printf):**

```bash
#!/bin/bash

# Return data via echo
get_full_name() {
    local first=$1
    local last=$2
    echo "${first} ${last}"
}

# Capture output
name=$(get_full_name "John" "Doe")
echo "Name: $name"
# Output: Name: John Doe

# Return multiple values (space-separated)
get_user_info() {
    echo "John 30 Engineer"
}

read name age job <<< $(get_user_info)
echo "Name: $name, Age: $age, Job: $job"

# Return array-like data
get_colors() {
    echo "red green blue yellow"
}

for color in $(get_colors); do
    echo "Color: $color"
done
```

---

## 🌍 Local vs Global Variables

### **Global Variables (default):**

```bash
#!/bin/bash

# Global variable
COUNTER=0

increment() {
    COUNTER=$((COUNTER + 1))    # Modifies global
}

increment
increment
increment

echo "Counter: $COUNTER"
# Output: Counter: 3
```

### **Local Variables:**

```bash
#!/bin/bash

GLOBAL_VAR="I'm global"

my_function() {
    local LOCAL_VAR="I'm local"
    GLOBAL_VAR="Modified global"
    
    echo "Inside function:"
    echo "  Local: $LOCAL_VAR"
    echo "  Global: $GLOBAL_VAR"
}

my_function

echo "Outside function:"
echo "  Local: $LOCAL_VAR"          # Empty (doesn't exist)
echo "  Global: $GLOBAL_VAR"        # Modified value
```

### **Best Practice: Always Use local:**

```bash
#!/bin/bash

calculate_tax() {
    local price=$1
    local tax_rate=0.08
    local tax=$(echo "scale=2; $price * $tax_rate" | bc)
    
    echo "$tax"
}

# tax_rate is not accessible here
result=$(calculate_tax 100)
echo "Tax: \$$result"
```

---

## 🔁 Recursion

**Function calling itself:**

```bash
#!/bin/bash

# Factorial using recursion
factorial() {
    local n=$1
    
    if [ $n -le 1 ]; then
        echo 1
    else
        local prev=$(factorial $((n - 1)))
        echo $((n * prev))
    fi
}

result=$(factorial 5)
echo "5! = $result"
# Output: 5! = 120

# Countdown
countdown() {
    local num=$1
    
    if [ $num -le 0 ]; then
        echo "Blast off!"
        return
    fi
    
    echo $num
    sleep 1
    countdown $((num - 1))
}

countdown 5
```

---

## 📚 Function Libraries

### **Creating a Library:**

```bash
# File: lib/string_utils.sh

uppercase() {
    echo "${1^^}"
}

lowercase() {
    echo "${1,,}"
}

trim() {
    local var=$1
    var="${var#"${var%%[![:space:]]*}"}"
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

string_length() {
    echo "${#1}"
}
```

### **Using the Library:**

```bash
#!/bin/bash
# main_script.sh

# Source the library
source "lib/string_utils.sh"

# Use library functions
TEXT="hello world"
echo "Original: $TEXT"
echo "Uppercase: $(uppercase "$TEXT")"
echo "Length: $(string_length "$TEXT")"

MESSY="  spaces  "
echo "Trimmed: '$(trim "$MESSY")'"
```

---

## 🎯 Practical Function Examples

### **Example 1: Logging Functions:**

```bash
#!/bin/bash

# Logging library
log_info() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [INFO] $*"
}

log_error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [ERROR] $*" >&2
}

log_warn() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [WARN] $*" >&2
}

# Usage
log_info "Starting application"
log_warn "Low disk space"
log_error "Failed to connect to database"
```

### **Example 2: Validation Functions:**

```bash
#!/bin/bash

is_valid_email() {
    local email=$1
    if [[ "$email" =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

is_valid_ip() {
    local ip=$1
    if [[ "$ip" =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
        return 0
    else
        return 1
    fi
}

# Usage
if is_valid_email "user@example.com"; then
    echo "Valid email"
fi

if ! is_valid_ip "999.999.999.999"; then
    echo "Invalid IP"
fi
```

### **Example 3: File Operations:**

```bash
#!/bin/bash

backup_file() {
    local file=$1
    
    if [ ! -f "$file" ]; then
        echo "Error: File not found: $file"
        return 1
    fi
    
    local backup="${file}.backup.$(date +%Y%m%d_%H%M%S)"
    cp "$file" "$backup"
    
    echo "Backed up to: $backup"
    return 0
}

rotate_logs() {
    local logfile=$1
    local max_size=$2    # in MB
    
    if [ ! -f "$logfile" ]; then
        return 0
    fi
    
    local size=$(du -m "$logfile" | cut -f1)
    
    if [ $size -gt $max_size ]; then
        mv "$logfile" "${logfile}.old"
        touch "$logfile"
        echo "Log rotated: $logfile"
    fi
}

# Usage
backup_file "/etc/nginx/nginx.conf"
rotate_logs "/var/log/app.log" 100
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you define a function?**
   <details>
   <summary>Answer</summary>
   function_name() { commands; } or function function_name() { commands; }
   </details>

2. **How do you access function parameters?**
   <details>
   <summary>Answer</summary>
   $1, $2, $3, etc. for individual parameters; $@ for all parameters
   </details>

3. **What does return do?**
   <details>
   <summary>Answer</summary>
   Sets exit code (0-255) for the function. Use echo/printf to return data.
   </details>

4. **What's the difference between local and global variables?**
   <details>
   <summary>Answer</summary>
   Local variables (declared with local) only exist inside the function.
   Global variables exist throughout the script and can be modified by functions.
   </details>

5. **How do you get output from a function?**
   <details>
   <summary>Answer</summary>
   result=$(function_name args) - captures what the function echoes
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Temperature Converter**

```bash
#!/bin/bash
# temp_converter.sh

celsius_to_fahrenheit() {
    local celsius=$1
    local fahrenheit=$(echo "scale=2; ($celsius * 9/5) + 32" | bc)
    echo "$fahrenheit"
}

fahrenheit_to_celsius() {
    local fahrenheit=$1
    local celsius=$(echo "scale=2; ($fahrenheit - 32) * 5/9" | bc)
    echo "$celsius"
}

# Test
echo "25°C = $(celsius_to_fahrenheit 25)°F"
echo "77°F = $(fahrenheit_to_celsius 77)°C"
```

### **Exercise 2: String Utilities**

```bash
#!/bin/bash
# string_utils.sh

reverse_string() {
    local str=$1
    echo "$str" | rev
}

count_words() {
    local str=$1
    echo "$str" | wc -w
}

capitalize_first() {
    local str=$1
    echo "${str^}"
}

# Test
TEXT="hello world"
echo "Original: $TEXT"
echo "Reversed: $(reverse_string "$TEXT")"
echo "Words: $(count_words "$TEXT")"
echo "Capitalized: $(capitalize_first "$TEXT")"
```

### **Exercise 3: File Size Checker**

```bash
#!/bin/bash
# file_checker.sh

get_file_size() {
    local file=$1
    
    if [ ! -f "$file" ]; then
        echo "0"
        return 1
    fi
    
    local size=$(stat -f%z "$file" 2>/dev/null || stat -c%s "$file" 2>/dev/null)
    echo "$size"
    return 0
}

format_size() {
    local bytes=$1
    
    if [ $bytes -lt 1024 ]; then
        echo "${bytes}B"
    elif [ $bytes -lt 1048576 ]; then
        echo "$(( bytes / 1024 ))KB"
    else
        echo "$(( bytes / 1048576 ))MB"
    fi
}

# Usage
FILE="/etc/passwd"
SIZE=$(get_file_size "$FILE")
echo "File: $FILE"
echo "Size: $(format_size $SIZE)"
```

### **Exercise 4: Menu System with Functions**

```bash
#!/bin/bash
# menu_system.sh

show_menu() {
    echo "===================="
    echo "   System Menu"
    echo "===================="
    echo "1. Show Date"
    echo "2. Show Uptime"
    echo "3. Show Users"
    echo "4. Exit"
    echo "===================="
}

show_date() {
    echo "Current date: $(date)"
}

show_uptime() {
    echo "System uptime: $(uptime -p)"
}

show_users() {
    echo "Logged in users:"
    who
}

# Main loop
while true; do
    show_menu
    read -p "Enter choice: " choice
    
    case $choice in
        1) show_date ;;
        2) show_uptime ;;
        3) show_users ;;
        4) echo "Goodbye!"; exit 0 ;;
        *) echo "Invalid choice!" ;;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
done
```

---

## 📝 Key Takeaways

✅ **Define functions before calling them**  
✅ **Use local variables inside functions**  
✅ **Return 0 for success, non-zero for failure**  
✅ **Echo/printf to return data, not return statement**  
✅ **Use $1, $2, etc. for parameters**  
✅ **Check parameter count with $#**  
✅ **Create libraries for reusable functions**  
✅ **Use descriptive function names (verb_noun)**  

---

## 🚀 Next Steps

You now know how to organize code with functions!

**Next lesson:** [12 - Arrays](12-arrays.md) - Working with lists of data

---

## 💡 Pro Tips

**Function template:**
```bash
function_name() {
    local param1=$1
    local param2=$2
    
    # Validation
    if [ $# -lt 2 ]; then
        echo "Usage: function_name <param1> <param2>"
        return 1
    fi
    
    # Logic here
    
    return 0
}
```

**Error handling:**
```bash
deploy() {
    local app=$1
    
    stop_app "$app" || return 1
    update_code "$app" || return 1
    start_app "$app" || return 1
    
    return 0
}
```

**Source library files:**
```bash
# At top of script
source "${BASH_SOURCE%/*}/lib/utils.sh"
```

Functions make scripts maintainable! 🎯
