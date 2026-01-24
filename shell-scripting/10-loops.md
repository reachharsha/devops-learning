---
render_with_liquid: false
---
# 10 - Loops

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- for loops (C-style and list iteration)
- while loops
- until loops
- Loop control (break, continue)
- Iterating over files and directories
- Reading files line by line
- Infinite loops and when to use them
- Nested loops

---

## 🔁 The for Loop

### **List-Based for Loop:**

```bash
#!/bin/bash

# Iterate over a list
for item in apple banana cherry; do
    echo "Fruit: $item"
done

# Output:
# Fruit: apple
# Fruit: banana
# Fruit: cherry

# Iterate over variable
FRUITS="apple banana cherry"
for fruit in $FRUITS; do
    echo "Processing: $fruit"
done

# Iterate over command output
for user in $(cat /etc/passwd | cut -d: -f1); do
    echo "User: $user"
done

# Iterate over numbers
for num in 1 2 3 4 5; do
    echo "Number: $num"
done

# Brace expansion
for i in {1..10}; do
    echo "Count: $i"
done

# Step by 2
for i in {0..20..2}; do
    echo "Even: $i"
done

# Iterate over files
for file in *.txt; do
    echo "Processing: $file"
done
```

### **C-Style for Loop:**

```bash
#!/bin/bash

# Standard C-style
for ((i=0; i<10; i++)); do
    echo "Iteration: $i"
done

# Count backwards
for ((i=10; i>=0; i--)); do
    echo "Countdown: $i"
done

# Step by 5
for ((i=0; i<=100; i+=5)); do
    echo "Value: $i"
done

# Multiple variables
for ((i=0, j=10; i<10; i++, j--)); do
    echo "i=$i, j=$j"
done
```

---

## ⏰ The while Loop

### **Basic while Loop:**

```bash
#!/bin/bash

# Count to 5
COUNT=1
while [ $COUNT -le 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Using (( ))
COUNT=1
while (( COUNT <= 5 )); do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Read user input
while true; do
    read -p "Enter command (quit to exit): " CMD
    if [ "$CMD" = "quit" ]; then
        break
    fi
    echo "You entered: $CMD"
done
```

### **Reading Files Line by Line:**

```bash
#!/bin/bash

# Method 1: while read (best)
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Method 2: Process with field separator
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < users.csv

# Method 3: Read from command output
ps aux | while read user pid cpu mem vsz rss tty stat start time command; do
    if (( $(echo "$cpu > 50" | bc -l) )); then
        echo "High CPU process: $command ($cpu%)"
    fi
done

# Method 4: Read from here-doc
while read -r line; do
    echo "Processing: $line"
done << 'EOF'
line1
line2
line3
EOF
```

---

## 🔄 The until Loop

**Runs until condition becomes TRUE (opposite of while):**

```bash
#!/bin/bash

# Count to 5
COUNT=1
until [ $COUNT -gt 5 ]; do
    echo "Count: $COUNT"
    ((COUNT++))
done

# Wait for file to appear
until [ -f "/tmp/ready.txt" ]; do
    echo "Waiting for ready signal..."
    sleep 5
done
echo "Ready file found!"

# Wait for service to be up
until curl -s http://localhost:8080 > /dev/null; do
    echo "Waiting for service..."
    sleep 2
done
echo "Service is up!"

# Retry with limit
ATTEMPTS=0
MAX_ATTEMPTS=5

until curl -s https://api.example.com || [ $ATTEMPTS -eq $MAX_ATTEMPTS ]; do
    ((ATTEMPTS++))
    echo "Attempt $ATTEMPTS failed, retrying..."
    sleep 5
done

if [ $ATTEMPTS -eq $MAX_ATTEMPTS ]; then
    echo "Failed after $MAX_ATTEMPTS attempts"
    exit 1
fi
```

---

## ⚡ Loop Control: break and continue

### **break - Exit Loop:**

```bash
#!/bin/bash

# Exit loop early
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        echo "Breaking at 5"
        break
    fi
    echo "Number: $i"
done
# Output: 1, 2, 3, 4, then breaks

# Search for file
for file in *.txt; do
    if grep -q "ERROR" "$file"; then
        echo "Found error in: $file"
        break
    fi
done

# Break from nested loops
for i in {1..3}; do
    for j in {1..3}; do
        if [ $i -eq 2 ] && [ $j -eq 2 ]; then
            echo "Breaking at i=$i, j=$j"
            break 2    # Break out of 2 levels
        fi
        echo "i=$i, j=$j"
    done
done
```

### **continue - Skip to Next Iteration:**

```bash
#!/bin/bash

# Skip even numbers
for i in {1..10}; do
    if (( i % 2 == 0 )); then
        continue    # Skip even numbers
    fi
    echo "Odd: $i"
done
# Output: 1, 3, 5, 7, 9

# Process only .txt files
for file in *; do
    if [[ ! $file == *.txt ]]; then
        continue
    fi
    echo "Processing: $file"
done

# Skip comments in file
while read -r line; do
    # Skip empty lines and comments
    if [[ -z "$line" || "$line" =~ ^# ]]; then
        continue
    fi
    echo "Config: $line"
done < config.txt
```

---

## 📁 Iterating Over Files

### **Files in Current Directory:**

```bash
#!/bin/bash

# All files
for file in *; do
    echo "Item: $file"
done

# Only .txt files
for file in *.txt; do
    if [ -f "$file" ]; then    # Check it's a file
        echo "Text file: $file"
    fi
done

# Multiple patterns
for file in *.{txt,md,log}; do
    if [ -f "$file" ]; then
        echo "Document: $file"
    fi
done

# Only directories
for dir in */; do
    echo "Directory: $dir"
done
```

### **Recursive File Iteration:**

```bash
#!/bin/bash

# Using find (best for recursive)
find /path/to/dir -name "*.txt" -type f | while read -r file; do
    echo "Processing: $file"
done

# Or with find -exec
find /path/to/dir -name "*.log" -type f -exec echo "Log: {}" \;

# Or store in array (for small sets)
while IFS= read -r -d '' file; do
    echo "File: $file"
done < <(find /path/to/dir -name "*.txt" -type f -print0)
```

### **Safe File Iteration (Handles Spaces):**

```bash
#!/bin/bash

# ❌ UNSAFE (breaks with spaces/special chars)
for file in $(ls *.txt); do
    cat "$file"
done

# ✅ SAFE
for file in *.txt; do
    if [ -f "$file" ]; then
        cat "$file"
    fi
done

# ✅ SAFE with find
find . -name "*.txt" -type f -print0 | while IFS= read -r -d '' file; do
    cat "$file"
done
```

---

## 🔄 Infinite Loops

### **When to Use:**

```bash
#!/bin/bash

# Infinite loop - must use break or kill to exit
while true; do
    echo "Running..."
    sleep 5
done

# Server/daemon pattern
while true; do
    # Check something
    DISK_USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
    
    if [ $DISK_USAGE -gt 90 ]; then
        echo "ALERT: Disk usage is ${DISK_USAGE}%"
        # Send notification
    fi
    
    sleep 300    # Check every 5 minutes
done

# Menu loop
while true; do
    echo "Menu:"
    echo "1. Start"
    echo "2. Stop"
    echo "3. Exit"
    read -p "Choice: " CHOICE
    
    case $CHOICE in
        1) echo "Starting...";;
        2) echo "Stopping...";;
        3) echo "Goodbye!"; break;;
        *) echo "Invalid choice";;
    esac
done

# Retry until success
while true; do
    if curl -s https://api.example.com > /dev/null; then
        echo "Service is up!"
        break
    fi
    echo "Service down, retrying in 10 seconds..."
    sleep 10
done
```

---

## 🎯 Practical Examples

### **Example 1: Batch File Processor**

```bash
#!/bin/bash
# process_images.sh - Resize all images

echo "Processing images..."

COUNT=0
for image in *.jpg *.png; do
    # Skip if no files match
    [ -f "$image" ] || continue
    
    echo "Processing: $image"
    
    # Create thumbnail
    convert "$image" -resize 200x200 "thumb_${image}"
    
    ((COUNT++))
done

echo "Processed $COUNT images"
```

### **Example 2: Log Analyzer**

```bash
#!/bin/bash
# analyze_logs.sh - Find errors in logs

ERROR_COUNT=0
WARNING_COUNT=0

while IFS= read -r line; do
    if [[ "$line" =~ ERROR ]]; then
        ((ERROR_COUNT++))
        echo "Error: $line"
    elif [[ "$line" =~ WARN ]]; then
        ((WARNING_COUNT++))
    fi
done < app.log

echo "---"
echo "Errors: $ERROR_COUNT"
echo "Warnings: $WARNING_COUNT"
```

### **Example 3: Server Monitor**

```bash
#!/bin/bash
# monitor_servers.sh - Check multiple servers

SERVERS=("web1.example.com" "web2.example.com" "db1.example.com")

for server in "${SERVERS[@]}"; do
    echo "Checking $server..."
    
    if ping -c 1 -W 1 "$server" > /dev/null 2>&1; then
        echo "  ✅ $server is UP"
    else
        echo "  ❌ $server is DOWN"
        # Send alert
        echo "Server $server is down!" | mail -s "Alert" admin@example.com
    fi
done
```

### **Example 4: Backup Rotation**

```bash
#!/bin/bash
# rotate_backups.sh - Keep only last 7 backups

BACKUP_DIR="/backups"
MAX_BACKUPS=7

# Count current backups
COUNT=$(ls -1 "$BACKUP_DIR"/backup_*.tar.gz 2>/dev/null | wc -l)

if [ $COUNT -le $MAX_BACKUPS ]; then
    echo "Only $COUNT backups, no rotation needed"
    exit 0
fi

# Calculate how many to delete
TO_DELETE=$((COUNT - MAX_BACKUPS))

echo "Found $COUNT backups, deleting oldest $TO_DELETE..."

# Delete oldest backups
ls -1t "$BACKUP_DIR"/backup_*.tar.gz | tail -n $TO_DELETE | while read -r backup; do
    echo "Deleting: $backup"
    rm -f "$backup"
done

echo "Rotation complete"
```

---

## 🔢 Nested Loops

```bash
#!/bin/bash

# Multiplication table
for i in {1..5}; do
    for j in {1..5}; do
        RESULT=$((i * j))
        printf "%2d " $RESULT
    done
    echo    # New line
done

# Process matrix data
while IFS= read -r row; do
    for item in $row; do
        echo "Processing: $item"
    done
done < matrix.txt

# Check multiple servers and services
SERVERS=("server1" "server2")
SERVICES=("nginx" "mysql" "redis")

for server in "${SERVERS[@]}"; do
    echo "Checking $server..."
    for service in "${SERVICES[@]}"; do
        ssh "$server" "systemctl is-active --quiet $service"
        if [ $? -eq 0 ]; then
            echo "  ✅ $service is running"
        else
            echo "  ❌ $service is down"
        fi
    done
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between while and until?**
   <details>
   <summary>Answer</summary>
   while runs while condition is TRUE
   until runs until condition becomes TRUE (while it's FALSE)
   </details>

2. **How do you safely iterate over files with spaces?**
   <details>
   <summary>Answer</summary>
   for file in *.txt; do ... (glob expansion is safe)
   Avoid: for file in $(ls *.txt)
   </details>

3. **What does break do vs continue?**
   <details>
   <summary>Answer</summary>
   break exits the loop entirely
   continue skips to next iteration
   </details>

4. **How do you read a file line by line?**
   <details>
   <summary>Answer</summary>
   while IFS= read -r line; do ... done < file.txt
   </details>

5. **What's the best loop for C-style counting?**
   <details>
   <summary>Answer</summary>
   for ((i=0; i<10; i++)); do ... done
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Number Processor**

```bash
#!/bin/bash
# number_processor.sh

echo "Processing numbers 1-20:"
echo "---"

for i in {1..20}; do
    # Skip multiples of 3
    if (( i % 3 == 0 )); then
        continue
    fi
    
    # Stop at 15
    if [ $i -eq 15 ]; then
        echo "Stopping at 15"
        break
    fi
    
    # Check if even or odd
    if (( i % 2 == 0 )); then
        echo "$i is even"
    else
        echo "$i is odd"
    fi
done
```

### **Exercise 2: File Backup**

```bash
#!/bin/bash
# backup_files.sh

SOURCE_DIR="$HOME/documents"
BACKUP_DIR="$HOME/backups/$(date +%Y%m%d)"

mkdir -p "$BACKUP_DIR"

echo "Backing up files from $SOURCE_DIR"
echo "---"

COUNT=0
for file in "$SOURCE_DIR"/*.txt; do
    [ -f "$file" ] || continue
    
    FILENAME=$(basename "$file")
    echo "Backing up: $FILENAME"
    cp "$file" "$BACKUP_DIR/"
    
    ((COUNT++))
done

echo "---"
echo "Backed up $COUNT files to $BACKUP_DIR"
```

### **Exercise 3: Menu System**

```bash
#!/bin/bash
# menu.sh

while true; do
    clear
    echo "================================"
    echo "        Main Menu"
    echo "================================"
    echo "1. Show date and time"
    echo "2. Show disk usage"
    echo "3. Show logged-in users"
    echo "4. Show system uptime"
    echo "5. Exit"
    echo "================================"
    read -p "Enter choice [1-5]: " CHOICE
    
    case $CHOICE in
        1)
            echo "Current date and time: $(date)"
            ;;
        2)
            echo "Disk usage:"
            df -h /
            ;;
        3)
            echo "Logged-in users:"
            who
            ;;
        4)
            echo "System uptime:"
            uptime
            ;;
        5)
            echo "Goodbye!"
            exit 0
            ;;
        *)
            echo "Invalid choice!"
            ;;
    esac
    
    echo ""
    read -p "Press Enter to continue..."
done
```

### **Exercise 4: CSV Processor**

```bash
#!/bin/bash
# process_csv.sh

# Create sample CSV
cat > users.csv << 'EOF'
name,age,city
John Doe,30,New York
Jane Smith,25,Los Angeles
Bob Johnson,35,Chicago
Alice Williams,28,Houston
EOF

echo "User Report"
echo "==========="

# Skip header
tail -n +2 users.csv | while IFS=',' read -r name age city; do
    echo "Name: $name"
    echo "  Age: $age"
    echo "  City: $city"
    
    # Age classification
    if [ $age -lt 30 ]; then
        echo "  Category: Young Adult"
    else
        echo "  Category: Adult"
    fi
    echo "---"
done

# Clean up
rm users.csv
```

---

## 📝 Key Takeaways

✅ **for loop for lists: for item in list; do ... done**  
✅ **C-style for counting: for ((i=0; i<10; i++)); do ... done**  
✅ **while loop: while [ condition ]; do ... done**  
✅ **until loop (opposite of while): until [ condition ]; do ... done**  
✅ **Read files: while read -r line; do ... done < file**  
✅ **break exits loop, continue skips to next iteration**  
✅ **Safe file iteration: for file in *.txt (not $(ls))**  
✅ **Infinite loops: while true; do ... done**  
✅ **Use find for recursive file processing**  

---

## 🚀 Next Steps

You now know how to automate repetitive tasks!

**Next lesson:** [11 - Functions](11-functions.md) - Organize code into reusable blocks

---

## 💡 Pro Tips

**Count processed items:**
```bash
COUNT=0
for file in *.txt; do
    # Process file
    ((COUNT++))
done
echo "Processed $COUNT files"
```

**Read file safely:**
```bash
# Best practice
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

**Iterate with index:**
```bash
i=0
for item in "${ARRAY[@]}"; do
    echo "[$i] $item"
    ((i++))
done
```

**Progress indicator:**
```bash
TOTAL=100
for ((i=1; i<=TOTAL; i++)); do
    PERCENT=$((i * 100 / TOTAL))
    printf "\rProgress: %d%%" $PERCENT
    # Do work
done
echo ""
```

Loops are your automation superpower! 🔄
