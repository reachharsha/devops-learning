# 13 - I/O Redirection & Pipes

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Standard input, output, and error streams
- Redirecting output (>, >>)
- Redirecting input (<)
- Piping commands (|)
- Combining commands effectively
- Using tee for dual output

---

## 🌊 Understanding Streams

**Real-World Analogy:**
```
Command = Factory Machine

stdin (0)   Input conveyor belt (raw materials)
stdout (1)  Output conveyor belt (finished product)
stderr (2)  Alert system (errors/warnings)

Default behavior:
stdin  ← keyboard
stdout → screen
stderr → screen

Redirection:
Change where these go!
```

**Technical View:**
```
File Descriptor    Stream           Default
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
0                  stdin (input)    Keyboard
1                  stdout (output)  Screen
2                  stderr (errors)  Screen
```

---

## 📤 Output Redirection

### **> - Redirect Output (Overwrite)**

```bash
# Send output to file (overwrites existing)
ls > files.txt

# Example:
echo "Hello" > greeting.txt
cat greeting.txt
# Output: Hello

# Overwrite previous content
echo "New content" > greeting.txt
cat greeting.txt
# Output: New content (previous content gone!)

# Redirect command output
ls -la > directory-listing.txt
ps aux > processes.txt
df -h > disk-usage.txt
```

### **>> - Redirect Output (Append)**

```bash
# Append to file (doesn't overwrite)
ls >> files.txt

# Example:
echo "Line 1" > file.txt
echo "Line 2" >> file.txt
echo "Line 3" >> file.txt

cat file.txt
# Output:
# Line 1
# Line 2
# Line 3

# Practical example: logging
echo "$(date): Server started" >> /var/log/myapp.log
echo "$(date): User logged in" >> /var/log/myapp.log
```

**Overwrite vs Append:**
```bash
# >  - DANGER! Overwrites entire file
# >> - SAFE! Adds to end of file

# Wrong ❌
echo "new" > important-file.txt    # Oops, lost everything!

# Right ✅
echo "new" >> important-file.txt   # Adds to existing content
```

---

## 🚨 Error Redirection

### **2> - Redirect Errors**

```bash
# Redirect errors to file
command 2> errors.txt

# Example:
ls /nonexistent 2> errors.txt
cat errors.txt
# Output: ls: cannot access '/nonexistent': No such file or directory

# Ignore errors (send to /dev/null - the black hole)
command 2> /dev/null

# Example:
find / -name "config" 2> /dev/null
# Shows results, hides "Permission denied" errors
```

### **&> or >& - Redirect Both Output and Errors**

```bash
# Redirect both stdout and stderr
command &> output.txt
command >& output.txt        # Same thing

# Example:
ls -la / &> all-output.txt

# Append both
command &>> output.txt

# More explicit method (preferred):
command > output.txt 2>&1

# Explanation:
# > output.txt      Send stdout to output.txt
# 2>&1              Send stderr to where stdout goes
```

---

## 🎯 Practical Redirection Examples

```bash
# Separate output and errors
command > output.txt 2> errors.txt

# Combine output, discard errors
command 2> /dev/null > output.txt

# Discard output, keep errors
command > /dev/null 2> errors.txt

# Discard everything
command > /dev/null 2>&1
command &> /dev/null

# Save successful output, see errors on screen
command > success.txt

# Save errors, see output on screen
command 2> errors.txt
```

**Real-world example:**
```bash
# Backup script with logging
mysqldump database > backup.sql 2> backup-errors.log

# Cron job (silence output)
0 2 * * * /scripts/backup.sh > /dev/null 2>&1

# Installation (save all output)
./install.sh &> installation-log.txt
```

---

## 📥 Input Redirection

### **< - Redirect Input**

```bash
# Read input from file instead of keyboard
command < input.txt

# Example:
# Send email with file contents
mail -s "Report" user@example.com < report.txt

# Sort file contents
sort < unsorted.txt

# Count lines
wc -l < file.txt

# Multiple redirections
command < input.txt > output.txt

# Example:
sort < unsorted.txt > sorted.txt
```

### **<< - Here Document**

**Embed multi-line input in script**

```bash
# Basic syntax
command << DELIMITER
text
more text
DELIMITER

# Example: Create file with multi-line content
cat << EOF > config.txt
server {
    listen 80;
    server_name example.com;
}
EOF

# Example: Email with body
mail -s "Subject" user@example.com << EOF
Dear User,

This is the email body.

Regards,
Admin
EOF

# Example: SQL commands
mysql database << EOF
CREATE TABLE users (id INT, name VARCHAR(100));
INSERT INTO users VALUES (1, 'John');
EOF
```

### **<<< - Here String**

**Single line input**

```bash
# Pass string as input
command <<< "string"

# Example:
cat <<< "Hello World"

# Example: Base64 encode
base64 <<< "password123"

# Example: Python
python3 <<< "print(2 + 2)"
```

---

## 🔗 Pipes

**| - Connect output of one command to input of another**

```bash
# Basic syntax
command1 | command2

# command1's output becomes command2's input
```

### **Simple Pipe Examples**

```bash
# Count files in directory
ls | wc -l

# Find specific process
ps aux | grep nginx

# Sort and paginate
ls -la | sort | less

# Search command history
history | grep git

# Top 10 largest files
ls -lh | sort -k5 -h | tail -10

# Unique lines
cat file.txt | sort | uniq

# Count unique lines
cat file.txt | sort | uniq | wc -l
```

---

## 🎭 Combining Multiple Pipes

**Chain commands together**

```bash
# Multiple pipes
command1 | command2 | command3 | command4

# Example: Log analysis
cat /var/log/nginx/access.log | grep "404" | wc -l
# Count 404 errors

# Example: Top 10 IPs
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head -10
# Show 10 most frequent IP addresses

# Example: Disk usage
du -h /var | sort -rh | head -20
# Top 20 largest directories

# Example: Network connections
netstat -an | grep ESTABLISHED | wc -l
# Count established connections

# Example: Memory usage
ps aux | awk '{print $4, $11}' | sort -rn | head -10
# Top 10 processes by memory

# Example: Find large log files
find /var/log -type f -exec du -h {} \; | sort -rh | head -10
```

---

## 🔧 tee - Dual Output

**Send output to file AND screen**

```bash
# Basic usage
command | tee file.txt

# Example:
ls -la | tee directory-listing.txt
# Shows on screen AND saves to file

# Append instead of overwrite
command | tee -a file.txt

# Multiple files
command | tee file1.txt file2.txt file3.txt

# Combine with pipes
command | tee output.txt | grep pattern

# Example: Installation log
./install.sh | tee install.log
# Watch installation AND save log

# Example: Debugging
python script.py | tee debug.log | grep ERROR
# See all output, save it, and highlight errors
```

---

## 🎯 Advanced Redirection

### **Process Substitution**

```bash
# Treat command output as file
command <(command1) <(command2)

# Example: Compare outputs
diff <(ls dir1) <(ls dir2)

# Example: Compare sorted files
diff <(sort file1.txt) <(sort file2.txt)

# Example: Compare two command outputs
diff <(ps aux | grep nginx) <(ps aux | grep apache)
```

### **File Descriptors**

```bash
# Custom file descriptors (3-9 available)

# Open file descriptor for reading
exec 3< file.txt

# Read from it
cat <&3

# Close it
exec 3<&-

# Open for writing
exec 4> output.txt

# Write to it
echo "data" >&4

# Close it
exec 4>&-
```

---

## 📚 Practical Workflow Examples

### **Log Analysis**

```bash
# Count error types
cat /var/log/app.log | grep ERROR | cut -d' ' -f4 | sort | uniq -c | sort -rn

# Extract unique errors in last hour
grep ERROR /var/log/app.log | grep "$(date +%Y-%m-%d\ %H)" | sort | uniq > recent-errors.txt

# Top 10 most common errors
cat /var/log/app.log | grep ERROR | awk -F':' '{print $3}' | sort | uniq -c | sort -rn | head -10
```

### **System Monitoring**

```bash
# Monitor top CPU processes continuously
while true; do
    ps aux --sort=-%cpu | head -6 | tee cpu-usage.log
    sleep 5
done

# Track memory over time
while true; do
    date >> memory.log
    free -h >> memory.log
    sleep 60
done

# Network monitoring
netstat -an | grep ESTABLISHED | wc -l | tee -a connections.log
```

### **Data Processing**

```bash
# CSV processing
cat data.csv | cut -d',' -f1,3 | grep "pattern" | sort | uniq > filtered.csv

# Extract emails from file
grep -oE '\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b' file.txt | sort | uniq > emails.txt

# Find duplicate lines
sort file.txt | uniq -d > duplicates.txt

# Remove duplicates
sort file.txt | uniq > unique-lines.txt
```

### **Backup & Compression**

```bash
# Backup with progress
tar -czf - /data | tee backup.tar.gz | sha256sum > backup.sha256

# Remote backup with progress
tar -czf - /data | tee >(ssh user@backup 'cat > backup.tar.gz') | pv > /dev/null
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you save command output to a file?**
   <details>
   <summary>Answer</summary>
   command > file.txt
   </details>

2. **How do you append output to a file?**
   <details>
   <summary>Answer</summary>
   command >> file.txt
   </details>

3. **How do you ignore error messages?**
   <details>
   <summary>Answer</summary>
   command 2> /dev/null
   </details>

4. **How do you pipe output of one command to another?**
   <details>
   <summary>Answer</summary>
   command1 | command2
   </details>

5. **How do you save output to file AND see it on screen?**
   <details>
   <summary>Answer</summary>
   command | tee file.txt
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Create test file
echo "Line 1" > test.txt

# 2. Append more lines
echo "Line 2" >> test.txt
echo "Line 3" >> test.txt

# 3. View and save
cat test.txt | tee output.txt

# 4. Count lines
cat test.txt | wc -l

# 5. Try non-existent file (see error)
cat nonexistent.txt

# 6. Redirect error
cat nonexistent.txt 2> error.log

# 7. View error log
cat error.log

# 8. List and count files
ls -la | wc -l > file-count.txt
cat file-count.txt

# 9. Multiple pipes
ls -la | grep "txt" | wc -l

# 10. Create multi-line file
cat << EOF > config.txt
Setting 1: Value1
Setting 2: Value2
Setting 3: Value3
EOF

# 11. View it
cat config.txt

# 12. Clean up
rm test.txt output.txt error.log file-count.txt config.txt
```

---

## 📝 Key Takeaways

✅ **>** - redirect output (overwrite)  
✅ **>>** - redirect output (append)  
✅ **2>** - redirect errors  
✅ **&>** - redirect both output and errors  
✅ **<** - redirect input  
✅ **|** - pipe (connect commands)  
✅ **| tee** - save to file AND display  
✅ **/dev/null** - discard output  
✅ **2>&1** - redirect stderr to stdout  

---

## 🚀 Next Steps

You can now chain commands like a pro!

**Next lesson:** [14-text-editors.md](14-text-editors.md) - Nano and Vim basics

---

## 💡 Pro Tips

**Common patterns:**
```bash
# Silent execution
command > /dev/null 2>&1

# Save everything
command &> all-output.txt

# See AND save
command | tee output.txt

# Separate output and errors
command > success.txt 2> errors.txt

# Append to log with timestamp
echo "$(date): Event" >> app.log
```

**Debug with tee:**
```bash
# See intermediate results
cat file.txt | tee step1.txt | sort | tee step2.txt | uniq > final.txt

# Debug pipeline
command1 | tee /tmp/debug1 | command2 | tee /tmp/debug2 | command3
```

**Error handling in scripts:**
```bash
#!/bin/bash

# Log all output
exec > >(tee -a script.log)
exec 2>&1

# Or redirect script output
{
    command1
    command2
    command3
} > output.log 2>&1
```

**Remember:**
```
>   Overwrite (dangerous!)
>>  Append (safe!)
|   Pipe (chain commands)
tee Both (screen + file)
```

Master I/O redirection = Power user! 🚀
