# 16 - Text Processing: grep & sed

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- grep for pattern matching and searching
- Regular expressions (basic, extended, PCRE)
- sed for stream editing
- Find and replace operations
- Line filtering and extraction
- Practical text processing patterns

---

## 🔍 grep - Global Regular Expression Print

### **Basic grep Usage:**

```bash
#!/bin/bash

# Search for pattern in file
grep "error" logfile.txt

# Case-insensitive search
grep -i "error" logfile.txt         # Matches Error, ERROR, error

# Show line numbers
grep -n "error" logfile.txt         # 42: error message

# Count matches
grep -c "error" logfile.txt         # 15

# Show only matching part
grep -o "error" logfile.txt         # error (one per line)

# Invert match (lines NOT containing pattern)
grep -v "error" logfile.txt         # All lines except errors
```

### **Searching Multiple Files:**

```bash
#!/bin/bash

# Search in multiple files
grep "TODO" *.sh

# Recursive search
grep -r "function" /path/to/directory

# Show only filenames with matches
grep -l "error" *.log               # error.log, app.log

# Show only filenames without matches
grep -L "success" *.log

# Show filename and line number
grep -Hn "error" *.log              # app.log:42:error message
```

### **grep Options:**

| Option | Meaning | Example |
|--------|---------|---------|
| `-i` | Ignore case | `grep -i "error"` |
| `-v` | Invert match | `grep -v "success"` |
| `-n` | Line numbers | `grep -n "pattern"` |
| `-c` | Count matches | `grep -c "error"` |
| `-l` | Files with matches | `grep -l "TODO" *.sh` |
| `-r` | Recursive | `grep -r "pattern" dir/` |
| `-w` | Whole word | `grep -w "cat"` (not "category") |
| `-x` | Whole line | `grep -x "exact line"` |
| `-A n` | n lines after | `grep -A 3 "error"` |
| `-B n` | n lines before | `grep -B 3 "error"` |
| `-C n` | n lines context | `grep -C 3 "error"` |
| `-o` | Only matching part | `grep -o "[0-9]+"` |
| `-q` | Quiet (no output) | `grep -q "pattern" && echo "found"` |

### **Context Lines:**

```bash
#!/bin/bash

# Show 2 lines after match
grep -A 2 "ERROR" logfile.txt

# Show 2 lines before match
grep -B 2 "ERROR" logfile.txt

# Show 2 lines before and after (context)
grep -C 2 "ERROR" logfile.txt

# Example output with -A 2:
# ERROR: Connection failed
# Timestamp: 2024-01-15 10:00:00
# User: admin
```

---

## 🎯 Regular Expressions with grep

### **Basic Regular Expressions:**

```bash
#!/bin/bash

# ^ - Start of line
grep "^ERROR" file.txt              # Lines starting with ERROR

# $ - End of line
grep "failed$" file.txt             # Lines ending with failed

# . - Any single character
grep "e.r" file.txt                 # ear, eer, e3r, etc.

# * - Zero or more of previous
grep "erro*r" file.txt              # err, error, errorr

# [abc] - Character class
grep "[aeiou]" file.txt             # Lines with vowels

# [^abc] - Negated character class
grep "[^0-9]" file.txt              # Lines with non-digits

# [a-z] - Range
grep "[a-zA-Z]" file.txt            # Lines with letters
```

### **Extended Regular Expressions (grep -E):**

```bash
#!/bin/bash

# + - One or more
grep -E "erro+" file.txt            # error, erroo, errooo

# ? - Zero or one
grep -E "colou?r" file.txt          # color or colour

# | - Alternation (OR)
grep -E "error|warning|critical" file.txt

# () - Grouping
grep -E "(error|warn):" file.txt    # error: or warn:

# {n} - Exactly n occurrences
grep -E "[0-9]{3}" file.txt         # Exactly 3 digits

# {n,} - n or more
grep -E "[0-9]{3,}" file.txt        # 3 or more digits

# {n,m} - Between n and m
grep -E "[0-9]{3,5}" file.txt       # 3 to 5 digits

# \b - Word boundary
grep -E "\bcat\b" file.txt          # "cat" but not "category"
```

### **Practical grep Examples:**

```bash
#!/bin/bash

# Extract IP addresses
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" access.log

# Extract email addresses
grep -oE "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" file.txt

# Find URLs
grep -oE "https?://[a-zA-Z0-9./?=_%:-]*" file.txt

# Extract phone numbers (xxx-xxx-xxxx)
grep -oE "[0-9]{3}-[0-9]{3}-[0-9]{4}" contacts.txt

# Find lines with dates (YYYY-MM-DD)
grep -E "[0-9]{4}-[0-9]{2}-[0-9]{2}" log.txt

# Multiple patterns (OR)
grep -E "ERROR|CRITICAL|FATAL" app.log

# Case-insensitive pattern
grep -iE "error|warning" log.txt
```

---

## ✏️ sed - Stream Editor

### **Basic sed Substitution:**

```bash
#!/bin/bash

# Substitute first occurrence on each line
sed 's/old/new/' file.txt

# Substitute all occurrences (global)
sed 's/old/new/g' file.txt

# Substitute on specific line
sed '3s/old/new/' file.txt          # Only line 3

# Substitute on line range
sed '1,5s/old/new/g' file.txt       # Lines 1-5

# Case-insensitive substitution
sed 's/old/new/gi' file.txt         # i flag for case-insensitive

# In-place editing (modify file)
sed -i 's/old/new/g' file.txt       # ⚠️ No backup
sed -i.bak 's/old/new/g' file.txt   # ✅ Creates .bak backup
```

### **sed Addressing:**

```bash
#!/bin/bash

# Specific line
sed '5d' file.txt                   # Delete line 5
sed '5p' file.txt                   # Print line 5 (duplicates it)
sed -n '5p' file.txt                # Print ONLY line 5 (-n suppresses auto-print)

# Range of lines
sed '10,20d' file.txt               # Delete lines 10-20
sed -n '10,20p' file.txt            # Print only lines 10-20

# From line to end
sed '10,$d' file.txt                # Delete from line 10 to end

# Pattern matching
sed '/error/d' file.txt             # Delete lines containing "error"
sed '/^$/d' file.txt                # Delete empty lines
sed '/^#/d' file.txt                # Delete comment lines

# Range from pattern to pattern
sed '/START/,/END/d' file.txt       # Delete from START to END
```

### **sed Commands:**

| Command | Action | Example |
|---------|--------|---------|
| `s/old/new/` | Substitute | `sed 's/foo/bar/'` |
| `d` | Delete | `sed '/pattern/d'` |
| `p` | Print | `sed -n '5p'` |
| `a` | Append after | `sed '5a\text'` |
| `i` | Insert before | `sed '5i\text'` |
| `c` | Change line | `sed '5c\newtext'` |
| `y` | Transform chars | `sed 'y/abc/ABC/'` |
| `q` | Quit | `sed '10q'` (like head -10) |

### **Advanced sed Substitution:**

```bash
#!/bin/bash

# Use different delimiter
sed 's|/old/path|/new/path|g' file.txt    # Use | instead of /
sed 's#old#new#g' file.txt                # Use # instead of /

# Back-references (capture groups)
# \1, \2, etc. refer to captured groups in ()
sed 's/\([0-9]*\)-\([0-9]*\)/\2-\1/' file.txt    # Swap numbers

# Remove leading spaces
sed 's/^[[:space:]]*//' file.txt

# Remove trailing spaces
sed 's/[[:space:]]*$//' file.txt

# Remove both leading and trailing spaces
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt

# Delete blank lines
sed '/^$/d' file.txt

# Delete lines with only whitespace
sed '/^[[:space:]]*$/d' file.txt

# Add text before pattern
sed '/pattern/i\This is inserted before' file.txt

# Add text after pattern
sed '/pattern/a\This is inserted after' file.txt

# Replace entire line
sed '/pattern/c\This replaces the entire line' file.txt
```

### **sed with Regular Expressions:**

```bash
#!/bin/bash

# Remove HTML tags
sed 's/<[^>]*>//g' file.html

# Extract text between tags
sed -n 's/.*<title>\(.*\)<\/title>.*/\1/p' file.html

# Comment out lines
sed 's/^/# /' file.txt                   # Add # to start of each line

# Uncomment lines
sed 's/^# //' file.txt                   # Remove # from start

# Number lines
sed = file.txt | sed 'N;s/\n/\t/'       # Add line numbers

# Double-space file
sed 'G' file.txt                         # Add blank line after each line

# Remove duplicate blank lines
sed '/^$/N;/^\n$/D' file.txt
```

---

## 🎯 Practical Examples

### **Example 1: Log Analyzer**

```bash
#!/bin/bash

LOGFILE="app.log"

echo "Log Analysis Report"
echo "==================="

# Count errors
ERROR_COUNT=$(grep -c "ERROR" "$LOGFILE")
echo "Errors: $ERROR_COUNT"

# Count warnings
WARN_COUNT=$(grep -c "WARN" "$LOGFILE")
echo "Warnings: $WARN_COUNT"

# Extract unique error messages
echo ""
echo "Unique Errors:"
grep "ERROR" "$LOGFILE" | sed 's/.*ERROR: //' | sort | uniq

# Top 10 IPs
echo ""
echo "Top 10 IP Addresses:"
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" "$LOGFILE" | \
    sort | uniq -c | sort -rn | head -10

# Errors in last hour
echo ""
echo "Recent Errors (last hour):"
HOUR_AGO=$(date -d '1 hour ago' '+%Y-%m-%d %H:' 2>/dev/null || date -v-1H '+%Y-%m-%d %H:')
grep "$HOUR_AGO" "$LOGFILE" | grep "ERROR"
```

### **Example 2: Configuration File Editor**

```bash
#!/bin/bash

CONFIG_FILE="app.conf"

# Backup original
cp "$CONFIG_FILE" "${CONFIG_FILE}.bak"

# Update port number
sed -i 's/^port=.*/port=8080/' "$CONFIG_FILE"

# Update host
sed -i 's/^host=.*/host=0.0.0.0/' "$CONFIG_FILE"

# Enable debug mode
sed -i 's/^debug=.*/debug=true/' "$CONFIG_FILE"

# Add new configuration if not exists
grep -q "^timeout=" "$CONFIG_FILE" || echo "timeout=30" >> "$CONFIG_FILE"

# Comment out deprecated settings
sed -i '/^old_setting=/s/^/# /' "$CONFIG_FILE"

echo "Configuration updated"
```

### **Example 3: CSV Data Processor**

```bash
#!/bin/bash

# Sample CSV
cat > users.csv << 'EOF'
name,email,age,city
John Doe,john@example.com,30,NYC
Jane Smith,jane@example.com,25,LA
Bob Johnson,bob@example.com,35,Chicago
EOF

# Extract emails only
echo "Email addresses:"
sed '1d' users.csv | cut -d, -f2

# Filter users over 30
echo ""
echo "Users over 30:"
sed '1d' users.csv | awk -F, '$3 > 30 {print $1, "-", $3}'

# Convert to tab-separated
echo ""
echo "Tab-separated format:"
sed 's/,/\t/g' users.csv

# Add prefix to emails
echo ""
echo "Prefixed emails:"
sed 's/@/ at /g' users.csv

# Clean up
rm users.csv
```

### **Example 4: Code Refactoring**

```bash
#!/bin/bash

# Rename function across all files
find . -name "*.sh" -type f -exec \
    sed -i 's/oldFunctionName/newFunctionName/g' {} \;

# Update import statements
sed -i 's|from old.module|from new.module|g' *.py

# Fix indentation (replace 4 spaces with tab)
sed -i 's/^    /\t/g' *.sh

# Remove trailing whitespace from all scripts
find . -name "*.sh" -exec sed -i 's/[[:space:]]*$//' {} \;

# Add header to all scripts
for file in *.sh; do
    sed -i '1i\#!/bin/bash' "$file"
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you search case-insensitively with grep?**
   <details>
   <summary>Answer</summary>
   grep -i "pattern" file
   </details>

2. **How do you replace all occurrences with sed?**
   <details>
   <summary>Answer</summary>
   sed 's/old/new/g' file (g flag for global)
   </details>

3. **How do you show context lines with grep?**
   <details>
   <summary>Answer</summary>
   grep -C 3 "pattern" file (3 lines before and after)
   </details>

4. **How do you edit a file in-place with sed?**
   <details>
   <summary>Answer</summary>
   sed -i 's/old/new/g' file (or sed -i.bak for backup)
   </details>

5. **How do you extract IP addresses from a log?**
   <details>
   <summary>Answer</summary>
   grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" log.txt
   </details>

---

## 🏋️ Hands-On Exercise

Practice exercises included in the practical examples above!

---

## 📝 Key Takeaways

✅ **grep searches for patterns in files**  
✅ **grep -E for extended regex (ERE)**  
✅ **sed 's/old/new/g' replaces text**  
✅ **sed -i modifies files in-place**  
✅ **Use -n with sed p to print specific lines**  
✅ **Combine grep and sed in pipelines**  
✅ **Always backup before sed -i**  

---

## 🚀 Next Steps

You now know grep and sed!

**Next lesson:** [17 - Text Processing: awk](17-text-processing-awk.md) - Advanced data processing

---

## 💡 Pro Tips

**Grep quiet mode for scripts:**
```bash
if grep -q "pattern" file; then
    echo "Found"
fi
```

**Sed with multiple commands:**
```bash
sed -e 's/old/new/g' -e 's/foo/bar/g' file
# or
sed 's/old/new/g; s/foo/bar/g' file
```

**Preview sed changes:**
```bash
sed 's/old/new/g' file    # Preview
sed -i 's/old/new/g' file # Apply
```

grep and sed are text processing powerhouses! 🔥
