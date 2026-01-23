# 06 - Text Processing

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Viewing file contents (cat, less, more, head, tail)
- Searching text and files (grep, find)
- Text manipulation (sort, uniq, cut, wc, tr)
- Advanced text tools (awk, sed)

---

## 📖 Viewing File Contents

### **cat - Concatenate and Display**

```bash
# Display file
cat file.txt

# Display multiple files
cat file1.txt file2.txt

# Show line numbers
cat -n file.txt

# Show non-printing characters
cat -A file.txt

# Combine files into new file
cat file1.txt file2.txt > combined.txt

# Append to file
cat file1.txt >> existing.txt

# Create file with content
cat > newfile.txt << EOF
Line 1
Line 2
EOF
```

---

### **less - Page Through Files**

```bash
# View file (better than more)
less file.txt

# Navigation:
Space       Next page
b           Previous page
/pattern    Search forward
?pattern    Search backward
n           Next match
N           Previous match
g           Go to beginning
G           Go to end
q           Quit

# Follow file (like tail -f)
less +F logfile.txt

# Show line numbers
less -N file.txt

# Multiple files
less file1.txt file2.txt
:n   (next file)
:p   (previous file)
```

---

### **head - View Beginning of File**

```bash
# First 10 lines (default)
head file.txt

# First N lines
head -n 20 file.txt
head -20 file.txt

# First N bytes
head -c 100 file.txt

# Multiple files
head file1.txt file2.txt

# All except last N lines
head -n -5 file.txt  # All except last 5
```

---

### **tail - View End of File**

```bash
# Last 10 lines (default)
tail file.txt

# Last N lines
tail -n 20 file.txt
tail -20 file.txt

# Follow file in real-time (VERY USEFUL!)
tail -f /var/log/syslog

# Follow with retry (if file doesn't exist yet)
tail -F logfile.log

# Multiple files
tail -f /var/log/*.log

# Last N lines and follow
tail -n 50 -f application.log
```

---

## 🔍 Searching

### **grep - Search Text**

**Basic usage:**
```bash
# Search for pattern
grep "error" logfile.txt

# Case-insensitive
grep -i "error" logfile.txt

# Show line numbers
grep -n "error" logfile.txt

# Invert match (lines NOT containing pattern)
grep -v "success" logfile.txt

# Count matches
grep -c "error" logfile.txt

# Show only matching part
grep -o "error" logfile.txt
```

**Advanced grep:**
```bash
# Recursive search in directory
grep -r "TODO" .

# Recursive, ignore case, line numbers
grep -rin "error" /var/log/

# Search multiple patterns
grep -e "error" -e "warning" file.txt

# Extended regex
grep -E "error|warning|critical" file.txt

# Whole word only
grep -w "error" file.txt
# Matches "error" but not "errors"

# Before/After context
grep -A 3 "error" file.txt   # 3 lines after
grep -B 3 "error" file.txt   # 3 lines before
grep -C 3 "error" file.txt   # 3 lines before and after

# Color output
grep --color=auto "error" file.txt

# List only filenames
grep -l "error" *.log

# List files without match
grep -L "error" *.log
```

**Practical examples:**
```bash
# Find failed SSH logins
grep "Failed password" /var/log/auth.log

# Find errors in last hour
grep "error" /var/log/syslog | grep "$(date +%H:)"

# Find large files mentioned in log
grep -E "[0-9]+MB" logfile.txt

# Find IP addresses
grep -E "\b[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\b" file.txt

# Find lines starting with #
grep "^#" file.txt

# Find empty lines
grep "^$" file.txt
```

---

### **find - Search Files (Covered in Lesson 04)**

Quick recap:
```bash
# Find files by name
find . -name "*.log"

# Find and search content
find . -name "*.txt" -exec grep -l "error" {} \;

# Find files modified today
find . -mtime 0

# Find large files
find . -size +100M
```

---

## 🛠️ Text Manipulation

### **sort - Sort Lines**

```bash
# Sort alphabetically
sort file.txt

# Sort numerically
sort -n numbers.txt

# Sort in reverse
sort -r file.txt

# Sort by column
sort -k 2 file.txt       # Sort by 2nd column

# Sort unique (remove duplicates)
sort -u file.txt

# Sort by file size
ls -l | sort -k 5 -n

# Sort IP addresses properly
sort -t . -k 1,1n -k 2,2n -k 3,3n -k 4,4n ips.txt

# Case-insensitive sort
sort -f file.txt
```

---

### **uniq - Remove Duplicates**

**Must be used with sorted data!**

```bash
# Remove adjacent duplicates
sort file.txt | uniq

# Count occurrences
sort file.txt | uniq -c

# Show only duplicates
sort file.txt | uniq -d

# Show only unique lines
sort file.txt | uniq -u

# Ignore case
sort file.txt | uniq -i

# Practical: Find most common errors
grep "error" /var/log/syslog | sort | uniq -c | sort -rn | head
```

---

### **wc - Word Count**

```bash
# Count lines, words, characters
wc file.txt

# Count lines only
wc -l file.txt

# Count words only
wc -w file.txt

# Count characters only
wc -c file.txt

# Multiple files
wc -l *.txt

# Count files in directory
ls | wc -l

# Count logged errors
grep "error" logfile.txt | wc -l
```

---

### **cut - Extract Columns**

```bash
# Cut by character position
cut -c 1-10 file.txt      # Characters 1-10

# Cut by field (default delimiter: tab)
cut -f 1,3 file.txt

# Cut with different delimiter
cut -d ':' -f 1 /etc/passwd    # Get usernames

# Cut range of fields
cut -d ',' -f 1-3 data.csv

# Examples:
# Get first column of CSV
cut -d ',' -f 1 data.csv

# Get usernames and home directories
cut -d ':' -f 1,6 /etc/passwd

# Get disk usage, sorted
df -h | tail -n +2 | cut -d ' ' -f 5 | sort -rn
```

---

### **tr - Translate Characters**

```bash
# Lowercase to uppercase
echo "hello" | tr 'a-z' 'A-Z'

# Replace characters
echo "hello world" | tr 'l' 'L'

# Delete characters
echo "hello123world" | tr -d '0-9'    # Remove numbers

# Squeeze repeats
echo "hello    world" | tr -s ' '     # Single spaces

# ROT13 encoding
echo "hello" | tr 'a-zA-Z' 'n-za-mN-ZA-M'

# Remove newlines
cat file.txt | tr -d '\n'

# Convert Windows line endings to Unix
tr -d '\r' < windows.txt > unix.txt
```

---

## 🚀 Advanced Text Processing

### **awk - Pattern Scanning and Processing**

```bash
# Print specific column
awk '{print $1}' file.txt          # First column
awk '{print $2, $3}' file.txt      # Columns 2 and 3

# Print with custom separator
awk -F ':' '{print $1}' /etc/passwd

# Conditional printing
awk '$3 > 100' file.txt            # Lines where col 3 > 100

# Sum column
awk '{sum += $1} END {print sum}' numbers.txt

# Average
awk '{sum += $1; count++} END {print sum/count}' numbers.txt

# Print line numbers
awk '{print NR, $0}' file.txt

# Print lines matching pattern
awk '/error/' logfile.txt

# Complex example: Print users with UID > 1000
awk -F ':' '$3 > 1000 {print $1}' /etc/passwd

# Format output
awk '{printf "%-10s %s\n", $1, $2}' file.txt

# Multiple conditions
awk '$1 == "error" && $3 > 100' file.txt
```

---

### **sed - Stream Editor**

```bash
# Replace first occurrence
sed 's/old/new/' file.txt

# Replace all occurrences
sed 's/old/new/g' file.txt

# Replace in-place (modify file)
sed -i 's/old/new/g' file.txt

# Delete lines
sed '/pattern/d' file.txt          # Delete matching lines
sed '5d' file.txt                  # Delete line 5
sed '2,5d' file.txt                # Delete lines 2-5

# Print specific lines
sed -n '1,10p' file.txt            # Print lines 1-10
sed -n '/error/p' file.txt         # Print lines with error

# Insert before line
sed '5i\New line' file.txt

# Append after line
sed '5a\New line' file.txt

# Multiple replacements
sed -e 's/foo/bar/g' -e 's/hello/world/g' file.txt

# Backup before in-place edit
sed -i.bak 's/old/new/g' file.txt

# Examples:
# Remove comments and blank lines
sed '/^#/d; /^$/d' config.conf

# Add line numbers
sed = file.txt | sed 'N;s/\n/\t/'

# Extract between patterns
sed -n '/START/,/END/p' file.txt
```

---

## 🎯 Practical Combinations

```bash
# Top 10 largest files
find . -type f -exec ls -lh {} \; | sort -k 5 -h -r | head -10

# Most common errors in log
grep "error" /var/log/syslog | awk '{print $5}' | sort | uniq -c | sort -rn | head

# Count HTTP status codes
cut -d ' ' -f 9 access.log | sort | uniq -c | sort -rn

# Find duplicate files by size
find . -type f -exec ls -l {} \; | awk '{print $5}' | sort | uniq -d

# Extract email addresses
grep -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b" file.txt

# CSV to JSON (simple)
awk -F',' '{print "{\"name\":\""$1"\", \"age\":\""$2"\"}"}' data.csv

# Monitor log for errors
tail -f application.log | grep --line-buffered "ERROR"

# Count code lines (excluding comments and blanks)
find . -name "*.py" -exec cat {} \; | sed '/^\s*#/d;/^\s*$/d' | wc -l
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you view the last 20 lines of a file?**
   <details>
   <summary>Answer</summary>
   tail -n 20 file.txt or tail -20 file.txt
   </details>

2. **How do you search for "error" case-insensitively in all .log files?**
   <details>
   <summary>Answer</summary>
   grep -i "error" *.log
   </details>

3. **How do you count the number of lines in a file?**
   <details>
   <summary>Answer</summary>
   wc -l file.txt
   </details>

4. **How do you remove duplicate lines from a file?**
   <details>
   <summary>Answer</summary>
   sort file.txt | uniq
   </details>

---

## 📝 Key Takeaways

✅ **cat** - display entire file  
✅ **less** - paginate through large files  
✅ **head/tail** - view beginning/end of file  
✅ **tail -f** - follow log files in real-time  
✅ **grep** - search text patterns  
✅ **sort** - sort lines  
✅ **uniq** - remove duplicates (use after sort!)  
✅ **wc** - count lines/words/characters  
✅ **cut** - extract columns  
✅ **awk** - powerful text processing  
✅ **sed** - stream editing and replacement  

---

## 🚀 Next Steps

**Next lesson:** [07-process-management.md](07-process-management.md)

---

## 💡 Pro Tip

**Combine tools with pipes:**
```bash
cat file.txt | grep "error" | sort | uniq -c | sort -rn | head -10

This one line:
1. Displays file
2. Filters errors
3. Sorts them
4. Counts duplicates
5. Sorts by count
6. Shows top 10

Master pipes = master Linux! 🚀
```
