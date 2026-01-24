---
render_with_liquid: false
---
# 31 - Shell Scripting Cheat Sheet

## 🎯 Quick Reference Guide

Your comprehensive quick reference for shell scripting!

---

## 📋 Script Template

```bash
#!/bin/bash
set -euo pipefail

# Constants
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly SCRIPT_NAME="$(basename "$0")"

# Variables
VERBOSE=false

# Functions
log() { echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"; }
error() { echo "[ERROR] $*" >&2; exit 1; }

# Cleanup
cleanup() { rm -f /tmp/*.tmp; }
trap cleanup EXIT

# Main
main() {
    log "Script started"
    # Your code here
    log "Script completed"
}

main "$@"
```

---

## 🔤 Variables

```bash
# Declaration
VAR="value"
readonly CONST="constant"
declare -i INT=42
declare -a ARRAY=(1 2 3)
declare -A MAP=([key]="value")

# Access
echo "$VAR"
echo "${VAR}"

# Default values
${VAR:-default}        # Use default if unset
${VAR:=default}        # Set and use default
${VAR:+"text"}         # Use text if set
${VAR:?"error"}        # Error if unset

# String operations
${#VAR}                # Length
${VAR:0:5}             # Substring
${VAR#pattern}         # Remove from start (shortest)
${VAR##pattern}        # Remove from start (longest)
${VAR%pattern}         # Remove from end (shortest)
${VAR%%pattern}        # Remove from end (longest)
${VAR/old/new}         # Replace first
${VAR//old/new}        # Replace all
${VAR^}                # Uppercase first char
${VAR^^}               # Uppercase all
${VAR,}                # Lowercase first char
${VAR,,}               # Lowercase all
```

---

## 🔢 Arithmetic

```bash
# Basic operations
$((1 + 2))             # Addition
$((5 - 3))             # Subtraction
$((4 * 3))             # Multiplication
$((10 / 3))            # Division (integer)
$((10 % 3))            # Modulo
$((2 ** 3))            # Exponentiation

# Increment/Decrement
((COUNT++))
((COUNT--))
((COUNT += 5))
((COUNT -= 3))

# Comparison
((a > b))
((a >= b))
((a < b))
((a <= b))
((a == b))
((a != b))
```

---

## ✅ Conditionals

```bash
# If statement
if [ condition ]; then
    echo "true"
elif [ condition2 ]; then
    echo "also true"
else
    echo "false"
fi

# [[ ]] (recommended)
if [[ condition ]]; then
    echo "true"
fi

# File tests
[ -e file ]            # Exists
[ -f file ]            # Regular file
[ -d dir ]             # Directory
[ -L link ]            # Symbolic link
[ -r file ]            # Readable
[ -w file ]            # Writable
[ -x file ]            # Executable
[ -s file ]            # Not empty
[ file1 -nt file2 ]    # Newer than
[ file1 -ot file2 ]    # Older than

# String tests
[ -z "$str" ]          # Empty string
[ -n "$str" ]          # Not empty
[ "$a" = "$b" ]        # Equal
[ "$a" != "$b" ]       # Not equal
[[ $str =~ regex ]]    # Regex match

# Numeric tests
[ "$a" -eq "$b" ]      # Equal
[ "$a" -ne "$b" ]      # Not equal
[ "$a" -lt "$b" ]      # Less than
[ "$a" -le "$b" ]      # Less or equal
[ "$a" -gt "$b" ]      # Greater than
[ "$a" -ge "$b" ]      # Greater or equal

# Logical operators
[ cond1 ] && [ cond2 ] # AND
[ cond1 ] || [ cond2 ] # OR
! [ cond ]             # NOT
[[ cond1 && cond2 ]]   # AND (in [[]])
[[ cond1 || cond2 ]]   # OR (in [[]])

# Case statement
case "$var" in
    pattern1)
        echo "match 1"
        ;;
    pattern2|pattern3)
        echo "match 2 or 3"
        ;;
    *)
        echo "default"
        ;;
esac
```

---

## 🔁 Loops

```bash
# For loop
for i in {1..10}; do
    echo "$i"
done

for file in *.txt; do
    echo "$file"
done

for ((i=0; i<10; i++)); do
    echo "$i"
done

# While loop
while [ condition ]; do
    echo "loop"
done

while read -r line; do
    echo "$line"
done < file.txt

# Until loop
until [ condition ]; do
    echo "loop"
done

# Loop control
break              # Exit loop
continue          # Next iteration
```

---

## 🔧 Functions

```bash
# Definition
function name() {
    echo "Hello"
}

# Or
name() {
    echo "Hello"
}

# With arguments
greet() {
    local name=$1
    echo "Hello, $name"
}
greet "John"

# Return value
add() {
    local result=$(($1 + $2))
    echo "$result"
}
sum=$(add 5 3)

# Return code
check() {
    [ condition ] && return 0 || return 1
}
if check; then
    echo "success"
fi

# Arguments
$0                 # Script name
$1, $2, ...       # Positional args
$#                # Argument count
$@                # All arguments
$*                # All arguments (as string)
$?                # Last exit code
$$                # Current PID
$!                # Last background PID
```

---

## 📁 File Operations

```bash
# Read file
cat file.txt
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Write file
echo "text" > file.txt     # Overwrite
echo "text" >> file.txt    # Append

# Create file
touch file.txt
: > file.txt               # Empty file

# Copy/Move/Delete
cp source dest
mv source dest
rm file.txt
rm -rf directory

# Permissions
chmod 755 file.sh
chmod u+x file.sh
chown user:group file.txt

# Find files
find . -name "*.txt"
find . -type f -mtime -7   # Modified in last 7 days
```

---

## 🔍 Text Processing

```bash
# grep
grep "pattern" file.txt
grep -i "pattern"          # Case-insensitive
grep -r "pattern" dir/     # Recursive
grep -v "pattern"          # Invert match
grep -E "regex"            # Extended regex
grep -o "pattern"          # Only matching

# sed
sed 's/old/new/' file.txt
sed 's/old/new/g'          # Global replace
sed -i 's/old/new/g'       # In-place edit
sed '/pattern/d'           # Delete lines
sed -n '5,10p'             # Print lines 5-10

# awk
awk '{print $1}' file.txt
awk -F: '{print $1}'       # Custom delimiter
awk '$3 > 100 {print}'     # Conditional
awk '{sum+=$1} END {print sum}'  # Sum column

# cut
cut -d: -f1 /etc/passwd
cut -c1-10 file.txt

# sort
sort file.txt
sort -n                    # Numeric sort
sort -r                    # Reverse
sort -u                    # Unique
sort -k2                   # Sort by column 2

# uniq
sort file.txt | uniq
sort file.txt | uniq -c    # Count occurrences
```

---

## 🌐 Networking

```bash
# curl
curl https://example.com
curl -o file.txt URL       # Save to file
curl -L URL                # Follow redirects
curl -I URL                # Headers only
curl -X POST URL           # POST request
curl -d "data" URL         # Send data
curl -H "Header: value"    # Custom header

# wget
wget URL
wget -O file.txt URL
wget -c URL                # Continue download
wget -r URL                # Recursive

# SSH
ssh user@host
ssh user@host "command"
scp file.txt user@host:/path/
rsync -avz source/ dest/
```

---

## ⚙️ Process Management

```bash
# Background/Foreground
command &              # Run in background
jobs                   # List jobs
fg %1                  # Foreground job 1
bg %1                  # Background job 1
kill %1                # Kill job 1

# Process control
ps aux                 # List processes
pgrep name             # Find process by name
pkill name             # Kill by name
kill PID               # Send SIGTERM
kill -9 PID            # Send SIGKILL

# Wait
wait                   # Wait for all background jobs
wait $PID              # Wait for specific process

# Signals
trap 'cleanup' EXIT
trap 'echo Interrupted' INT
```

---

## 📊 Input/Output

```bash
# Redirection
command > file         # Stdout to file
command 2> file        # Stderr to file
command &> file        # Both to file
command >> file        # Append
command < file         # Input from file
command1 | command2    # Pipe

# Here document
cat << EOF
Line 1
Line 2
EOF

# Here string
grep "pattern" <<< "text"

# Read input
read -p "Enter name: " name
read -s password       # Silent (for passwords)
read -t 5 input        # Timeout
```

---

## 🔐 Security

```bash
# Never hardcode credentials
PASSWORD="${DB_PASSWORD}"

# Validate input
[[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]

# Sanitize filename
filename=$(basename "$user_input")
filename=${filename//[^a-zA-Z0-9._-]/}

# Secure permissions
chmod 600 sensitive.conf
umask 077

# Avoid eval
# Never: eval "$user_input"
```

---

## 🐛 Debugging

```bash
# Enable debugging
set -x                 # Print commands
set -v                 # Print input
bash -x script.sh      # Run with debug

# Check syntax
bash -n script.sh

# Strict mode
set -e                 # Exit on error
set -u                 # Error on undefined
set -o pipefail        # Pipe failure

# Combined
set -euo pipefail

# Shellcheck
shellcheck script.sh
```

---

## 📅 Date and Time

```bash
# Current date/time
date
date '+%Y-%m-%d'
date '+%Y-%m-%d %H:%M:%S'

# Timestamp
date +%s

# Calculate dates (GNU date)
date -d '1 day ago'
date -d '2024-01-15' +%s

# macOS
date -v-1d
```

---

## 🎨 Colors

```bash
# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
RESET='\033[0m'

# Usage
echo -e "${RED}Error${RESET}"
echo -e "${GREEN}Success${RESET}"
```

---

## 🔧 Arrays

```bash
# Declaration
arr=(1 2 3)
declare -a arr=(1 2 3)
declare -A map=([key]="value")

# Access
${arr[0]}              # First element
${arr[@]}              # All elements
${arr[*]}              # All (as string)
${#arr[@]}             # Length
${!arr[@]}             # Indices

# Append
arr+=(4)

# Iterate
for item in "${arr[@]}"; do
    echo "$item"
done

# Slice
${arr[@]:1:3}          # Elements 1-3
```

---

## 🎯 Common Patterns

```bash
# Check command exists
command -v git >/dev/null 2>&1 || { echo "git required"; exit 1; }

# Retry logic
for i in {1..3}; do
    command && break || sleep 2
done

# Timeout
timeout 10 command

# Find and execute
find . -name "*.txt" -exec command {} \;

# Parallel execution
for i in {1..10}; do
    command &
done
wait

# Lock file
LOCKFILE="/var/lock/script.lock"
if ! mkdir "$LOCKFILE" 2>/dev/null; then
    echo "Already running"
    exit 1
fi
trap "rmdir '$LOCKFILE'" EXIT

# Dry run
if $DRY_RUN; then
    echo "Would execute: command"
else
    command
fi
```

---

## 📝 Quick Tips

```bash
# Quote variables
echo "$VAR"            # ✅ Good
echo $VAR              # ❌ Bad

# Check exit codes
command || exit 1

# Use local in functions
local var="value"

# Use [[ ]] for tests
[[ condition ]]        # ✅ Good
[ condition ]          # ⚠️  OK but limited

# Avoid cat
grep pattern < file    # ✅ Good
cat file | grep        # ❌ Useless cat

# Use built-ins
${file##*/}            # ✅ Fast
$(basename "$file")    # ❌ Slow
```

---

## 🚀 Production Checklist

```bash
✅ Shebang: #!/bin/bash
✅ Strict mode: set -euo pipefail
✅ Quote all variables: "$VAR"
✅ Use local in functions
✅ Check exit codes
✅ Validate input
✅ Handle errors (trap)
✅ Log important events
✅ Comment complex logic
✅ Run shellcheck
✅ Test edge cases
✅ Document usage
```

---

## 📚 Resources

- **Man pages:** `man bash`, `man test`
- **Shellcheck:** https://shellcheck.net
- **Bash Guide:** https://mywiki.wooledge.org/BashGuide
- **Advanced Bash:** https://tldp.org/LDP/abs/html/

---

## 🎓 Congratulations!

You've completed the comprehensive shell scripting course!

You now have:
- ✅ Solid foundation in shell scripting
- ✅ Production-ready scripting skills
- ✅ DevOps automation expertise
- ✅ Security best practices knowledge
- ✅ Debugging and testing abilities

**Keep this cheat sheet handy and keep scripting!** 🚀

---

## 💡 Final Pro Tips

**Learn by doing:**
Write scripts for your daily tasks

**Read code:**
Study well-written scripts on GitHub

**Practice:**
Solve problems on coding challenges

**Stay updated:**
Shell features and best practices evolve

**Share knowledge:**
Help others learn shell scripting

**Happy scripting!** 🎉
