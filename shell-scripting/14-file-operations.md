---
render_with_liquid: false
---
# 14 - File Operations

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Reading files (line by line, entire file)
- Writing to files (overwrite, append)
- File descriptors and redirection
- Temporary files and cleanup
- File locking
- Processing CSV, JSON, and XML files
- Advanced file manipulation techniques

---

## 📖 Reading Files

### **Method 1: Read Line by Line (Best):**

```bash
#!/bin/bash

# Read file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt

# Preserve leading/trailing whitespace
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Read with line numbers
LINE_NUM=0
while IFS= read -r line; do
    ((LINE_NUM++))
    echo "$LINE_NUM: $line"
done < file.txt
```

### **Method 2: Read Entire File:**

```bash
#!/bin/bash

# Read entire file into variable
CONTENT=$(cat file.txt)
echo "$CONTENT"

# Read into array (one element per line)
mapfile -t LINES < file.txt
# or
readarray -t LINES < file.txt

# Access lines
echo "${LINES[0]}"     # First line
echo "${LINES[-1]}"    # Last line

# Iterate
for line in "${LINES[@]}"; do
    echo "$line"
done
```

### **Method 3: Read with Field Separator:**

```bash
#!/bin/bash

# CSV file: name,age,city
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < users.csv

# /etc/passwd (colon-separated)
while IFS=':' read -r username password uid gid comment home shell; do
    echo "User: $username, Home: $home, Shell: $shell"
done < /etc/passwd

# Space-separated
while read -r col1 col2 col3; do
    echo "Columns: $col1 | $col2 | $col3"
done < data.txt
```

### **Method 4: Read Specific Lines:**

```bash
#!/bin/bash

# Read first line
head -n 1 file.txt

# Read last line
tail -n 1 file.txt

# Read lines 5-10
sed -n '5,10p' file.txt

# Read line 42
sed -n '42p' file.txt

# Skip header, process data
tail -n +2 file.csv | while IFS=',' read -r col1 col2; do
    echo "$col1 : $col2"
done
```

---

## ✍️ Writing to Files

### **Overwrite File:**

```bash
#!/bin/bash

# Simple write (overwrites existing content)
echo "Hello, World!" > output.txt

# Multiple lines
cat > output.txt << 'EOF'
Line 1
Line 2
Line 3
EOF

# Write variable
NAME="John"
echo "Hello, $NAME" > greeting.txt

# Write command output
date > timestamp.txt
ls -l > directory_listing.txt
```

### **Append to File:**

```bash
#!/bin/bash

# Append (preserves existing content)
echo "New line" >> output.txt

# Append multiple lines
cat >> output.txt << 'EOF'
Additional line 1
Additional line 2
EOF

# Log to file
echo "[$(date)] Application started" >> app.log
```

### **Write with printf:**

```bash
#!/bin/bash

# Formatted output
printf "%-20s | %5s | %10s\n" "Name" "Age" "City" > report.txt
printf "%-20s | %5d | %10s\n" "John Doe" 30 "New York" >> report.txt
printf "%-20s | %5d | %10s\n" "Jane Smith" 25 "Los Angeles" >> report.txt

# Fixed-width numbers
printf "%05d\n" 42 > number.txt    # 00042
```

---

## 🔀 File Descriptors

**File descriptors: 0 (stdin), 1 (stdout), 2 (stderr)**

### **Basic Usage:**

```bash
#!/bin/bash

# Redirect stdout to file
ls -l 1> output.txt
# or
ls -l > output.txt

# Redirect stderr to file
ls /nonexistent 2> errors.txt

# Redirect both stdout and stderr
ls -l /etc /nonexistent > output.txt 2>&1
# or (bash 4+)
ls -l /etc /nonexistent &> output.txt

# Separate files for stdout and stderr
command > output.txt 2> errors.txt

# Append stderr
command 2>> errors.log

# Discard output
command > /dev/null 2>&1
```

### **Advanced File Descriptors:**

```bash
#!/bin/bash

# Open file for reading (fd 3)
exec 3< input.txt

# Read from fd 3
while IFS= read -r line <&3; do
    echo "Read: $line"
done

# Close fd 3
exec 3<&-

# Open file for writing (fd 4)
exec 4> output.txt

# Write to fd 4
echo "Hello" >&4
echo "World" >&4

# Close fd 4
exec 4>&-

# Read and write same file
exec 5<> file.txt
# Read from fd 5
read -r line <&5
# Write to fd 5
echo "New content" >&5
exec 5>&-
```

---

## 🔒 File Locking

**Prevent concurrent access:**

```bash
#!/bin/bash

LOCKFILE="/tmp/myapp.lock"

# Acquire lock
exec 200> "$LOCKFILE"
if ! flock -n 200; then
    echo "Another instance is running"
    exit 1
fi

# Critical section (protected by lock)
echo "Processing..."
sleep 5

# Lock is automatically released when script exits
```

### **Lock with Timeout:**

```bash
#!/bin/bash

LOCKFILE="/tmp/myapp.lock"

exec 200> "$LOCKFILE"

# Try to acquire lock, wait max 10 seconds
if ! flock -w 10 200; then
    echo "Could not acquire lock after 10 seconds"
    exit 1
fi

echo "Lock acquired, processing..."
# ... do work ...
```

---

## 📄 Temporary Files

### **Create Temp Files Safely:**

```bash
#!/bin/bash

# Create temporary file
TEMPFILE=$(mktemp)
echo "Temporary file: $TEMPFILE"

# Use it
echo "Some data" > "$TEMPFILE"

# Clean up
trap "rm -f '$TEMPFILE'" EXIT

# Create in specific directory
TEMPFILE=$(mktemp /tmp/myapp.XXXXXX)

# Create temporary directory
TEMPDIR=$(mktemp -d)
echo "Temporary directory: $TEMPDIR"
# Clean up
trap "rm -rf '$TEMPDIR'" EXIT
```

### **Atomic File Updates:**

```bash
#!/bin/bash

update_config() {
    local config_file=$1
    local new_content=$2
    
    # Create temp file
    local temp_file=$(mktemp)
    
    # Write new content
    echo "$new_content" > "$temp_file"
    
    # Atomic replace (rename is atomic on most systems)
    mv "$temp_file" "$config_file"
}

update_config "/etc/app.conf" "new configuration"
```

---

## 📊 Processing Structured Data

### **CSV Processing:**

```bash
#!/bin/bash

# Read CSV with header
{
    # Read header
    IFS=',' read -r -a HEADERS
    
    # Process data rows
    while IFS=',' read -r -a ROW; do
        for i in "${!HEADERS[@]}"; do
            echo "${HEADERS[$i]}: ${ROW[$i]}"
        done
        echo "---"
    done
} < data.csv

# Generate CSV
cat > report.csv << 'EOF'
Name,Age,City
John Doe,30,NYC
Jane Smith,25,LA
EOF

# Process and filter
awk -F',' '$2 > 25 {print $1}' report.csv
```

### **JSON Processing (with jq):**

```bash
#!/bin/bash

# Create JSON file
cat > data.json << 'EOF'
{
  "users": [
    {"name": "John", "age": 30},
    {"name": "Jane", "age": 25}
  ]
}
EOF

# Extract data with jq
jq '.users[0].name' data.json              # "John"
jq '.users[].name' data.json               # All names
jq '.users[] | select(.age > 26)' data.json  # Filter

# Without jq (simple cases)
extract_json_field() {
    local json=$1
    local field=$2
    echo "$json" | grep -o "\"$field\"[[:space:]]*:[[:space:]]*\"[^\"]*\"" | \
                   head -1 | sed 's/.*:"\(.*\)"/\1/'
}

JSON='{"name":"John","age":30}'
NAME=$(extract_json_field "$JSON" "name")
echo "Name: $NAME"
```

### **XML Processing:**

```bash
#!/bin/bash

# Simple XML parsing
cat > data.xml << 'EOF'
<users>
  <user>
    <name>John</name>
    <age>30</age>
  </user>
</users>
EOF

# Extract with xmllint
xmllint --xpath '//user/name/text()' data.xml

# Extract with grep/sed (simple cases)
grep -oP '(?<=<name>)[^<]+' data.xml

# Using awk
awk -F'[<>]' '/<name>/{print $3}' data.xml
```

---

## 🎯 Practical Examples

### **Example 1: Log Rotator**

```bash
#!/bin/bash

rotate_log() {
    local logfile=$1
    local max_size_mb=$2
    local keep_backups=${3:-5}
    
    # Check if file exists and size
    if [ ! -f "$logfile" ]; then
        return 0
    fi
    
    local size_mb=$(du -m "$logfile" | cut -f1)
    
    if [ $size_mb -gt $max_size_mb ]; then
        # Rotate existing backups
        for ((i=$keep_backups-1; i>0; i--)); do
            if [ -f "${logfile}.$i" ]; then
                mv "${logfile}.$i" "${logfile}.$((i+1))"
            fi
        done
        
        # Compress and rotate current log
        gzip -c "$logfile" > "${logfile}.1.gz"
        > "$logfile"  # Truncate current log
        
        # Delete old backups
        if [ -f "${logfile}.$((keep_backups+1)).gz" ]; then
            rm "${logfile}.$((keep_backups+1)).gz"
        fi
        
        echo "Log rotated: $logfile"
    fi
}

rotate_log "/var/log/app.log" 100 5
```

### **Example 2: Configuration File Manager**

```bash
#!/bin/bash

CONFIG_FILE="app.conf"

# Read config
read_config() {
    declare -gA CONFIG
    
    while IFS='=' read -r key value; do
        # Skip comments and empty lines
        [[ $key =~ ^#.*$ || -z $key ]] && continue
        
        # Trim whitespace
        key=$(echo "$key" | xargs)
        value=$(echo "$value" | xargs)
        
        CONFIG[$key]=$value
    done < "$CONFIG_FILE"
}

# Write config
write_config() {
    {
        echo "# Application Configuration"
        echo "# Generated: $(date)"
        echo ""
        
        for key in "${!CONFIG[@]}"; do
            echo "$key = ${CONFIG[$key]}"
        done
    } > "$CONFIG_FILE"
}

# Update value
update_config() {
    local key=$1
    local value=$2
    
    read_config
    CONFIG[$key]=$value
    write_config
}

# Usage
update_config "port" "8080"
update_config "host" "localhost"
```

### **Example 3: Merge Multiple Files**

```bash
#!/bin/bash

merge_files() {
    local output=$1
    shift
    local files=("$@")
    
    # Clear output file
    > "$output"
    
    for file in "${files[@]}"; do
        if [ -f "$file" ]; then
            echo "# Content from: $file" >> "$output"
            cat "$file" >> "$output"
            echo "" >> "$output"
        else
            echo "Warning: $file not found" >&2
        fi
    done
    
    echo "Merged ${#files[@]} files into $output"
}

merge_files "combined.txt" file1.txt file2.txt file3.txt
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you read a file line by line?**
   <details>
   <summary>Answer</summary>
   while IFS= read -r line; do ... done < file.txt
   </details>

2. **What's the difference between > and >>?**
   <details>
   <summary>Answer</summary>
   > overwrites the file, >> appends to the file
   </details>

3. **How do you redirect stderr?**
   <details>
   <summary>Answer</summary>
   command 2> errors.txt
   </details>

4. **How do you create a temporary file?**
   <details>
   <summary>Answer</summary>
   TEMPFILE=$(mktemp)
   </details>

5. **How do you prevent race conditions when writing files?**
   <details>
   <summary>Answer</summary>
   Use file locking with flock or atomic operations with mktemp + mv
   </details>

---

## 🏋️ Hands-On Exercise

See exercises in the content above for hands-on practice with file operations!

---

## 📝 Key Takeaways

✅ **Read files line by line: while read -r line; do ... done < file**  
✅ **Write: > (overwrite), >> (append)**  
✅ **File descriptors: 0 (stdin), 1 (stdout), 2 (stderr)**  
✅ **Temp files: mktemp**  
✅ **File locking: flock**  
✅ **Always clean up temp files with trap**  

---

## 🚀 Next Steps

You now master file operations!

**Next lesson:** [15 - I/O Redirection](15-io-redirection.md) - Advanced input/output techniques

---

## 💡 Pro Tips

**Always use IFS= read -r:**
```bash
while IFS= read -r line; do
    # IFS= preserves whitespace
    # -r prevents backslash interpretation
done < file.txt
```

**Safe temp file pattern:**
```bash
TEMP=$(mktemp)
trap "rm -f '$TEMP'" EXIT
# Use $TEMP
```

Files are everywhere in scripting! 📁
