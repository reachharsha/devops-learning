---
render_with_liquid: false
---
# 29 - Advanced Topics

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Subshells and command grouping
- Co-processes and named pipes (FIFOs)
- Job control and signal handling
- Here documents and here strings
- Terminal control and colors
- Shell options and modes
- Advanced parameter expansion
- Performance optimization

---

## 🔄 Subshells and Command Grouping

### **Subshells ( ):**

```bash
#!/bin/bash

# Subshell - runs in separate process
(
    cd /tmp
    echo "In subshell: $(pwd)"
    VAR="subshell value"
)

echo "Outside: $(pwd)"        # Original directory
echo "VAR: ${VAR:-not set}"   # Variable not set outside subshell

# Practical use: Preserve environment
ORIGINAL_DIR=$(pwd)
(
    cd /some/path
    # Do work
) 
# Back in original directory automatically

# Parallel execution in subshells
(sleep 2; echo "Task 1") &
(sleep 1; echo "Task 2") &
wait
```

### **Command Grouping { }:**

```bash
#!/bin/bash

# Group commands - runs in current shell
{
    cd /tmp
    echo "In group: $(pwd)"
    VAR="group value"
}

echo "Outside: $(pwd)"     # /tmp (changed)
echo "VAR: $VAR"           # group value (set)

# Redirect group output
{
    echo "Line 1"
    echo "Line 2"
    echo "Line 3"
} > output.txt

# Pipe group output
{
    echo "Name: John"
    echo "Age: 30"
    echo "City: NYC"
} | grep "Age"
```

---

## 🔗 Named Pipes (FIFOs)

### **Create and Use FIFOs:**

```bash
#!/bin/bash

# Create named pipe
FIFO="/tmp/myfifo"
mkfifo "$FIFO"

# Writer (background)
{
    for i in {1..5}; do
        echo "Message $i"
        sleep 1
    done
} > "$FIFO" &

# Reader
while read line; do
    echo "Received: $line"
done < "$FIFO"

# Cleanup
rm "$FIFO"
```

### **Process Communication:**

```bash
#!/bin/bash

# Two-way communication using FIFOs
FIFO1="/tmp/fifo1"
FIFO2="/tmp/fifo2"

mkfifo "$FIFO1" "$FIFO2"

# Process 1
{
    echo "Request from P1" > "$FIFO1"
    read response < "$FIFO2"
    echo "P1 got: $response"
} &

# Process 2
{
    read request < "$FIFO1"
    echo "P2 got: $request"
    echo "Response from P2" > "$FIFO2"
} &

wait

rm "$FIFO1" "$FIFO2"
```

---

## 🎮 Advanced Job Control

### **Job Control Commands:**

```bash
#!/bin/bash

# Start background jobs
sleep 100 &
JOB1=$!

sleep 200 &
JOB2=$!

# List jobs
jobs -l

# Job states
# Running: Currently executing
# Stopped: Suspended with Ctrl+Z
# Done: Completed
# Terminated: Killed

# Control jobs
fg %1          # Bring job 1 to foreground
bg %1          # Resume job 1 in background
kill %1        # Kill job 1
disown %1      # Remove from job table

# Wait for specific job
wait $JOB1
echo "Job 1 completed"

# Get job status
if jobs -r %1 >/dev/null 2>&1; then
    echo "Job 1 is running"
fi
```

### **Co-processes:**

```bash
#!/bin/bash

# Start co-process (bash 4.0+)
coproc MYCOPROC {
    while read line; do
        echo "Received: $line"
        echo "Response: $(date)"
    done
}

# Write to co-process
echo "Hello" >&${MYCOPROC[1]}

# Read from co-process
read response <&${MYCOPROC[0]}
echo "$response"

# Close co-process
eval "exec ${MYCOPROC[1]}>&-"  # Close input
wait $MYCOPROC_PID
```

---

## 📜 Advanced Here Documents

### **Here Document Features:**

```bash
#!/bin/bash

# Basic here doc
cat << 'EOF'
This is a here document
Variables like $HOME are NOT expanded
EOF

# With variable expansion
cat << EOF
User: $USER
Home: $HOME
EOF

# Strip leading tabs
cat <<- EOF
	This line has a leading tab
	But it will be removed
EOF

# Redirect to file
cat << EOF > config.txt
server=localhost
port=8080
EOF

# Here doc as function argument
mysql -u root -p password << EOF
USE mydb;
SELECT * FROM users;
EOF

# Multi-line command
ssh user@server << 'ENDSSH'
cd /var/www
git pull
npm install
pm2 restart app
ENDSSH
```

### **Here Strings:**

```bash
#!/bin/bash

# Here string (<<<)
grep "pattern" <<< "text to search"

# Read into variable
read var <<< "hello world"
echo "$var"

# Multiple lines
while IFS= read -r line; do
    echo "Line: $line"
done <<< "Line 1
Line 2
Line 3"

# With command substitution
grep "error" <<< "$(cat logfile.txt)"
```

---

## 🎨 Terminal Control and Colors

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
WHITE='\033[0;37m'
BOLD='\033[1m'
UNDERLINE='\033[4m'
RESET='\033[0m'

# Usage
echo -e "${RED}This is red${RESET}"
echo -e "${GREEN}This is green${RESET}"
echo -e "${BOLD}${BLUE}Bold blue${RESET}"
echo -e "${UNDERLINE}Underlined${RESET}"

# Background colors
BG_RED='\033[41m'
BG_GREEN='\033[42m'
BG_BLUE='\033[44m'

echo -e "${BG_RED}${WHITE}Red background${RESET}"

# 256 colors
for i in {0..255}; do
    printf "\033[38;5;%dmColor %d\033[0m\n" "$i" "$i"
done
```

### **Terminal Control:**

```bash
#!/bin/bash

# Clear screen
clear
# or
printf "\033[2J"

# Move cursor
# \033[row;colH
printf "\033[10;20H"  # Move to row 10, col 20

# Save/restore cursor position
printf "\033[s"       # Save position
printf "\033[u"       # Restore position

# Hide/show cursor
printf "\033[?25l"    # Hide cursor
printf "\033[?25h"    # Show cursor

# Terminal size
ROWS=$(tput lines)
COLS=$(tput cols)
echo "Terminal: ${COLS}x${ROWS}"
```

### **Progress Bar:**

```bash
#!/bin/bash

progress_bar() {
    local current=$1
    local total=$2
    local width=50
    
    local percentage=$((current * 100 / total))
    local completed=$((width * current / total))
    local remaining=$((width - completed))
    
    printf "\r["
    printf "%${completed}s" | tr ' ' '='
    printf "%${remaining}s" | tr ' ' ' '
    printf "] %d%%" "$percentage"
}

# Usage
TOTAL=100
for i in $(seq 1 $TOTAL); do
    progress_bar $i $TOTAL
    sleep 0.1
done
printf "\n"
```

---

## ⚙️ Shell Options and Modes

### **set Options:**

```bash
#!/bin/bash

# Error handling
set -e          # Exit on error
set -u          # Error on undefined variable
set -o pipefail # Pipe failure detection
set -x          # Debug mode (print commands)
set -v          # Verbose (print input lines)

# Combined
set -euxo pipefail

# Disable options
set +e          # Don't exit on error
set +x          # Disable debug mode

# Check if option is set
if [[ $- == *e* ]]; then
    echo "errexit is enabled"
fi
```

### **shopt Options (Bash):**

```bash
#!/bin/bash

# Null glob (expand to nothing if no match)
shopt -s nullglob
for file in *.txt; do
    echo "$file"  # No error if no .txt files
done

# Dot glob (include hidden files)
shopt -s dotglob
for file in *; do
    echo "$file"  # Includes .hidden files
done

# Extended glob
shopt -s extglob
# !(pattern) - match anything except pattern
# ?(pattern) - match zero or one occurrence
# *(pattern) - match zero or more occurrences
# +(pattern) - match one or more occurrences
# @(pattern) - match exactly one occurrence

rm !(*.txt)     # Remove all except .txt files

# Globstar (recursive **)
shopt -s globstar
for file in **/*.txt; do
    echo "$file"  # All .txt files recursively
done

# Check shopt option
if shopt -q nullglob; then
    echo "nullglob is enabled"
fi
```

---

## 🔧 Advanced Parameter Expansion

### **String Manipulation:**

```bash
#!/bin/bash

VAR="Hello World"

# Length
echo ${#VAR}                    # 11

# Substring
echo ${VAR:0:5}                 # Hello
echo ${VAR:6}                   # World
echo ${VAR: -5}                 # World (note the space)

# Remove from start
FILE="path/to/file.txt"
echo ${FILE#*/}                 # to/file.txt (remove shortest)
echo ${FILE##*/}                # file.txt (remove longest)

# Remove from end
echo ${FILE%/*}                 # path/to (remove shortest)
echo ${FILE%%/*}                # path (remove longest)

# Replace
echo ${VAR/World/Universe}      # Replace first
echo ${VAR//o/0}                # Replace all

# Case modification
echo ${VAR^}                    # Hello World (first char upper)
echo ${VAR^^}                   # HELLO WORLD (all upper)
echo ${VAR,}                    # hello World (first char lower)
echo ${VAR,,}                   # hello world (all lower)

# Default values
echo ${UNDEFINED:-"default"}    # default (if unset)
echo ${UNDEFINED:="default"}    # Set and return default
echo ${DEFINED:+"exists"}       # Return if set
echo ${UNDEFINED:?"error msg"}  # Error if unset
```

### **Array Manipulation:**

```bash
#!/bin/bash

# Array operations
ARR=(one two three four five)

# Slice
echo ${ARR[@]:1:3}              # two three four

# Length
echo ${#ARR[@]}                 # 5

# Replace in array
echo ${ARR[@]/o/0}              # 0ne tw0 three f0ur five

# Keys
echo ${!ARR[@]}                 # 0 1 2 3 4

# Append
ARR+=(six seven)

# Remove element
unset 'ARR[2]'

# Associative arrays
declare -A MAP
MAP[key1]="value1"
MAP[key2]="value2"

# Keys
echo ${!MAP[@]}                 # key1 key2

# Values
echo ${MAP[@]}                  # value1 value2
```

---

## ⚡ Performance Optimization

### **Optimization Techniques:**

```bash
#!/bin/bash

# ❌ SLOW: Multiple process spawns
for file in *.txt; do
    COUNT=$(wc -l < "$file")
    echo "$file: $COUNT"
done

# ✅ FAST: Single process
wc -l *.txt

# ❌ SLOW: Subshell in loop
while read line; do
    echo "$line" | awk '{print $1}'
done < file.txt

# ✅ FAST: awk once
awk '{print $1}' file.txt

# ❌ SLOW: External commands
BASENAME=$(basename "$FILE")
DIRNAME=$(dirname "$FILE")

# ✅ FAST: Parameter expansion
BASENAME=${FILE##*/}
DIRNAME=${FILE%/*}

# ❌ SLOW: cat | grep
cat file.txt | grep pattern

# ✅ FAST: grep directly
grep pattern file.txt

# Use built-ins
# ✅ FAST
[[ $var = "value" ]]
# vs
# ❌ SLOW
test "$var" = "value"
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between ( ) and { }?**
   <details>
   <summary>Answer</summary>
   ( ) runs in subshell, { } runs in current shell
   </details>

2. **How do you create a named pipe?**
   <details>
   <summary>Answer</summary>
   mkfifo /path/to/pipe
   </details>

3. **What does set -e do?**
   <details>
   <summary>Answer</summary>
   Exit script immediately if any command fails
   </details>

4. **How do you enable recursive globbing?**
   <details>
   <summary>Answer</summary>
   shopt -s globstar
   </details>

5. **What's faster: $(basename file) or ${file##*/}?**
   <details>
   <summary>Answer</summary>
   ${file##*/} (no subprocess)
   </details>

---

## 📝 Key Takeaways

✅ **( ) for subshells, { } for grouping**  
✅ **FIFOs for inter-process communication**  
✅ **Use set -euo pipefail for safety**  
✅ **shopt for bash-specific features**  
✅ **Parameter expansion for performance**  
✅ **Avoid unnecessary subshells**  
✅ **Use built-ins over external commands**  
✅ **ANSI codes for colors**  

---

## 🚀 Next Steps

You've mastered advanced techniques!

**Next lesson:** [30 - Best Practices and Pitfalls](30-best-practices-pitfalls.md) - Avoid common mistakes

---

## 💡 Pro Tips

**Debug complex pipelines:**
```bash
set -o pipefail
cmd1 | cmd2 | cmd3
echo "Exit: $?"
```

**Benchmark scripts:**
```bash
time ./script.sh
```

**Use here docs for configs:**
```bash
cat > config.yml << EOF
server:
  host: localhost
  port: 8080
EOF
```

Advanced techniques = expert level! 🚀
