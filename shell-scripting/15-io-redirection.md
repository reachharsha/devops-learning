---
render_with_liquid: false
---
# 15 - I/O Redirection

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Standard streams (stdin, stdout, stderr)
- Redirection operators (>, >>, <, <<)
- Pipes and command chaining
- Here documents and here strings
- Process substitution
- Named pipes (FIFOs)
- Combining multiple redirections

---

## 🌊 The Three Standard Streams

```bash
# 0 = stdin  (standard input)   - keyboard input
# 1 = stdout (standard output)  - normal output
# 2 = stderr (standard error)   - error messages
```

**Visual Representation:**
```
     ┌──────────┐
 0 →│          │→ 1 (success output)
    │ PROGRAM  │
    │          │→ 2 (error output)
     └──────────┘
```

---

## ➡️ Output Redirection

### **Redirect stdout (>):**

```bash
#!/bin/bash

# Redirect stdout to file (overwrites)
echo "Hello" > output.txt

ls -l > file_list.txt

date > timestamp.txt

# Overwrite examples
echo "Line 1" > test.txt
echo "Line 2" > test.txt     # Only "Line 2" remains
```

### **Append stdout (>>):**

```bash
#!/bin/bash

# Append to file (preserves existing content)
echo "Line 1" >> output.txt
echo "Line 2" >> output.txt
echo "Line 3" >> output.txt

# Log file pattern
echo "[$(date)] Application started" >> app.log
echo "[$(date)] User logged in" >> app.log
```

### **Redirect stderr (2>):**

```bash
#!/bin/bash

# Redirect only errors
ls /nonexistent 2> errors.txt
# Screen: (nothing)
# errors.txt contains error message

# Example with both success and error
ls /etc /nonexistent > output.txt 2> errors.txt
# output.txt: listing of /etc
# errors.txt: error about /nonexistent
```

### **Redirect both stdout and stderr:**

```bash
#!/bin/bash

# Method 1: Redirect stderr to stdout, then to file
command > output.txt 2>&1

# Method 2: Bash shorthand (bash 4+)
command &> output.txt

# Append both
command >> output.txt 2>&1
command &>> output.txt

# Practical example: Silent execution
command > /dev/null 2>&1
# or
command &> /dev/null
```

### **Separate Files for stdout and stderr:**

```bash
#!/bin/bash

# Different files
command > success.txt 2> errors.txt

# Real example: compilation
gcc program.c -o program > output.log 2> errors.log

if [ -s errors.log ]; then
    echo "Compilation failed:"
    cat errors.log
else
    echo "Compilation successful"
fi
```

---

## ⬅️ Input Redirection

### **Redirect stdin (<):**

```bash
#!/bin/bash

# Read from file instead of keyboard
wc -l < file.txt

# Sort file contents
sort < unsorted.txt > sorted.txt

# Read file in loop
while read -r line; do
    echo "Line: $line"
done < input.txt
```

---

## 📜 Here Documents (<<)

**Multi-line input without external file:**

### **Basic Here Document:**

```bash
#!/bin/bash

# Write multi-line content
cat << EOF
This is line 1
This is line 2
This is line 3
EOF

# Save to file
cat << EOF > output.txt
Line 1
Line 2
Line 3
EOF

# With variables expanded
NAME="John"
cat << EOF
Hello, $NAME!
Today is $(date)
EOF
```

### **Literal Here Document (no expansion):**

```bash
#!/bin/bash

# Single-quote the delimiter for literal text
cat << 'EOF'
$HOME will not be expanded
$(date) will not be executed
EOF

# Useful for generating scripts
cat << 'SCRIPT' > generate.sh
#!/bin/bash
echo "This is a generated script"
echo "Current directory: $(pwd)"
SCRIPT

chmod +x generate.sh
```

### **Indented Here Document:**

```bash
#!/bin/bash

# Use <<- to ignore leading tabs (not spaces!)
if true; then
    cat <<- EOF
	This line is indented with tabs
	Tabs will be removed
	But content remains aligned
	EOF
fi
```

### **Practical Here Document Examples:**

```bash
#!/bin/bash

# Generate configuration file
cat << EOF > /etc/app.conf
# Application Configuration
PORT=8080
HOST=localhost
DATABASE=mydb
DEBUG=false
EOF

# Send email
mail -s "Server Alert" admin@example.com << EOF
Server: $(hostname)
Time: $(date)
Status: Critical disk space

Please investigate immediately.
EOF

# SQL query
mysql -u root -p database << 'EOSQL'
SELECT * FROM users WHERE active = 1;
UPDATE stats SET count = count + 1;
EOSQL
```

---

## 📝 Here Strings (<<<)

**Single-line input:**

```bash
#!/bin/bash

# Feed string as input
wc -w <<< "Hello World"        # Output: 2

# Read into variables
read -r name age <<< "John 30"
echo "Name: $name, Age: $age"

# With loops
while read -r line; do
    echo "Line: $line"
done <<< "Single line"

# Process variable
DATA="apple,banana,cherry"
IFS=',' read -ra FRUITS <<< "$DATA"
echo "${FRUITS[@]}"
```

---

## 🔀 Pipes (|)

**Chain commands together:**

### **Basic Piping:**

```bash
#!/bin/bash

# stdout of first → stdin of second
ls -l | grep "\.txt"

# Multiple pipes
cat file.txt | grep "error" | wc -l

# Common patterns
ps aux | grep nginx | grep -v grep

# Sort and unique
cat names.txt | sort | uniq

# Count unique IPs in log
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn
```

### **Pipe stderr:**

```bash
#!/bin/bash

# Pipe stderr to another command
command 2>&1 | grep "error"

# Separate stdout and stderr in pipes
{
    command 2>&1 1>&3 | grep "ERROR" > errors.txt
} 3>&1 | grep "SUCCESS" > success.txt
```

---

## 🔄 Process Substitution

**Treat command output as a file:**

### **Basic Process Substitution:**

```bash
#!/bin/bash

# <(command) creates a temporary "file" with command output

# Compare outputs of two commands
diff <(ls dir1) <(ls dir2)

# Read from process
while read -r line; do
    echo "Line: $line"
done < <(find . -name "*.txt")

# Multiple inputs
paste <(seq 1 5) <(seq 6 10)
# Output:
# 1	6
# 2	7
# 3	8
# 4	9
# 5	10
```

### **Output Process Substitution (>(...)):**

```bash
#!/bin/bash

# Send output to multiple commands
tee >(grep "ERROR" > errors.log) \
    >(grep "WARN" > warnings.log) \
    < input.log > all.log

# Copy and process simultaneously
tar czf - directory | tee >(sha256sum > checksum.txt) > backup.tar.gz
```

---

## 🚰 Named Pipes (FIFOs)

**Persistent pipes for inter-process communication:**

### **Creating and Using Named Pipes:**

```bash
#!/bin/bash

# Create named pipe
mkfifo /tmp/mypipe

# Writer (background)
{
    for i in {1..5}; do
        echo "Message $i"
        sleep 1
    done
} > /tmp/mypipe &

# Reader (foreground)
while read -r line; do
    echo "Received: $line"
done < /tmp/mypipe

# Clean up
rm /tmp/mypipe
```

### **Practical FIFO Example:**

```bash
#!/bin/bash

# Monitor log in real-time with processing
FIFO=/tmp/log_processor

mkfifo "$FIFO"
trap "rm -f '$FIFO'" EXIT

# Processor (background)
{
    while read -r line; do
        # Process each log line
        if [[ $line =~ ERROR ]]; then
            echo "[ALERT] $line" >> alerts.log
        fi
        echo "$line" >> processed.log
    done
} < "$FIFO" &

# Feed log to processor
tail -f /var/log/app.log > "$FIFO"
```

---

## 🎯 Advanced Redirection Patterns

### **Swap stdout and stderr:**

```bash
#!/bin/bash

# Swap streams
command 3>&1 1>&2 2>&3 3>&-

# Example
{
    echo "This is stdout" 1>&2
    echo "This is stderr" 2>&1
} 3>&1 1>&2 2>&3 3>&-
```

### **Duplicate streams:**

```bash
#!/bin/bash

# Duplicate output to file and screen
command | tee output.txt

# Duplicate including stderr
command 2>&1 | tee output.txt

# Append
command | tee -a output.txt

# Multiple files
command | tee file1.txt file2.txt file3.txt
```

### **Close file descriptors:**

```bash
#!/bin/bash

# Close stdout
exec 1>&-

# Close stdin
exec 0<&-

# Close custom fd
exec 3>&-
```

---

## 🎯 Practical Examples

### **Example 1: Logging System**

```bash
#!/bin/bash

LOGFILE="app.log"

# Log function with timestamp
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOGFILE"
}

log_error() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] [ERROR] $*" | tee -a "$LOGFILE" >&2
}

# Usage
log "Application started"
log "Processing data..."
log_error "Failed to connect to database"
log "Application stopped"
```

### **Example 2: Data Pipeline**

```bash
#!/bin/bash

# Complex data processing pipeline
cat access.log |
    grep "GET" |                           # Filter GET requests
    awk '{print $1}' |                     # Extract IP addresses
    sort |                                  # Sort
    uniq -c |                              # Count unique
    sort -rn |                             # Sort by count
    head -10 |                             # Top 10
    while read count ip; do
        printf "%-15s : %5d requests\n" "$ip" "$count"
    done > top_ips.txt
```

### **Example 3: Parallel Processing with Named Pipes**

```bash
#!/bin/bash

WORKERS=4
FIFO_DIR=/tmp/parallel_fifos

mkdir -p "$FIFO_DIR"
trap "rm -rf '$FIFO_DIR'" EXIT

# Create worker pipes
for i in $(seq 1 $WORKERS); do
    mkfifo "$FIFO_DIR/pipe_$i"
    
    # Start worker
    {
        while read -r task; do
            echo "Worker $i processing: $task"
            sleep 1  # Simulate work
        done
    } < "$FIFO_DIR/pipe_$i" &
done

# Distribute tasks
TASKS=("task1" "task2" "task3" "task4" "task5" "task6")
worker_idx=1

for task in "${TASKS[@]}"; do
    echo "$task" > "$FIFO_DIR/pipe_$worker_idx"
    worker_idx=$(( (worker_idx % WORKERS) + 1 ))
done

# Wait for workers
wait
```

---

## 🎯 Quick Check: Do You Understand?

1. **What are the three standard streams?**
   <details>
   <summary>Answer</summary>
   0 = stdin, 1 = stdout, 2 = stderr
   </details>

2. **How do you redirect stderr to a file?**
   <details>
   <summary>Answer</summary>
   command 2> errors.txt
   </details>

3. **What's the difference between > and >>?**
   <details>
   <summary>Answer</summary>
   > overwrites file, >> appends to file
   </details>

4. **How do you create a here document?**
   <details>
   <summary>Answer</summary>
   cat << EOF ... EOF
   </details>

5. **What does | do?**
   <details>
   <summary>Answer</summary>
   Pipe - connects stdout of one command to stdin of another
   </details>

---

## 📝 Key Takeaways

✅ **> redirects stdout (overwrite)**  
✅ **>> redirects stdout (append)**  
✅ **2> redirects stderr**  
✅ **&> redirects both stdout and stderr**  
✅ **< redirects stdin**  
✅ **<< here document for multi-line input**  
✅ **<<< here string for single-line input**  
✅ **| pipes output between commands**  
✅ **<(...) process substitution**  

---

## 🚀 Next Steps

You're now an I/O redirection master!

**Next lesson:** [16 - Text Processing grep & sed](16-text-processing-grep-sed.md) - Pattern matching and stream editing

---

## 💡 Pro Tips

**Silent execution:**
```bash
command &> /dev/null
```

**Save output and see it:**
```bash
command | tee output.txt
```

**Both streams to different files:**
```bash
command > out.txt 2> err.txt
```

**Redirect in the middle of pipeline:**
```bash
cat file | grep pattern > matches.txt | wc -l
```

Redirection is powerful! 🌊
