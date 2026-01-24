# 17 - Text Processing: awk

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- awk syntax and structure
- Built-in variables ($1, $2, NR, NF, etc.)
- Patterns and actions
- Conditional processing
- awk arrays and loops
- Mathematical operations
- Practical data processing patterns

---

## 🎨 What is awk?

**awk is a powerful text processing language for:**
- Processing columnar data (CSV, logs, reports)
- Calculating statistics
- Generating reports
- Data transformation

**Basic Structure:**
```bash
awk 'pattern { action }' file
```

---

## 🔤 Basic awk Syntax

### **Print Columns:**

```bash
#!/bin/bash

# Create sample data
cat > data.txt << 'EOF'
John 30 Engineer
Jane 25 Designer
Bob 35 Manager
EOF

# Print first column
awk '{print $1}' data.txt
# Output:
# John
# Jane
# Bob

# Print multiple columns
awk '{print $1, $3}' data.txt
# Output:
# John Engineer
# Jane Designer
# Bob Manager

# Print all columns
awk '{print $0}' data.txt

# Custom separator in output
awk '{print $1 ":" $2}' data.txt
# Output:
# John:30
# Jane:25
# Bob:35
```

### **Field Separator:**

```bash
#!/bin/bash

# Default: whitespace (space/tab)
awk '{print $1}' file.txt

# Custom separator: -F option
awk -F':' '{print $1}' /etc/passwd     # Colon separator
awk -F',' '{print $2}' data.csv         # Comma separator

# Multiple character separator
awk -F'::' '{print $1}' file.txt

# Regular expression separator
awk -F'[,:]' '{print $1}' file.txt     # Comma OR colon
```

---

## 📊 Built-in Variables

| Variable | Meaning | Example |
|----------|---------|---------|
| `$0` | Entire line | `awk '{print $0}'` |
| `$1, $2, ...` | Field 1, 2, ... | `awk '{print $1}'` |
| `NF` | Number of fields | `awk '{print NF}'` |
| `NR` | Current record number | `awk '{print NR}'` |
| `FNR` | Record number in current file | Multiple files |
| `FS` | Field separator (input) | `FS=","` |
| `OFS` | Output field separator | `OFS="\t"` |
| `RS` | Record separator (default: newline) | `RS=";"` |
| `ORS` | Output record separator | `ORS="\n\n"` |
| `FILENAME` | Current filename | `awk '{print FILENAME}'` |

### **Using Built-in Variables:**

```bash
#!/bin/bash

# Print with line numbers
awk '{print NR, $0}' file.txt

# Print number of fields
awk '{print NF}' file.txt

# Print last field
awk '{print $NF}' file.txt

# Print second-to-last field
awk '{print $(NF-1)}' file.txt

# Change output separator
awk 'BEGIN {OFS="\t"} {print $1, $2, $3}' file.txt

# Print filename with content
awk '{print FILENAME ":", $0}' file1.txt file2.txt
```

---

## 🎯 Patterns and Actions

### **Patterns:**

```bash
#!/bin/bash

# Match pattern
awk '/error/ {print}' log.txt           # Lines containing "error"

# Regular expressions
awk '/^ERROR/ {print}' log.txt          # Lines starting with ERROR
awk '/failed$/ {print}' log.txt         # Lines ending with failed

# Comparison operators
awk '$3 > 30 {print $1, $3}' data.txt   # Third column > 30
awk '$2 == "active" {print}' file.txt   # Second column equals "active"

# Multiple patterns
awk '/error/ || /warning/ {print}' log.txt
awk '$3 > 30 && $2 == "admin" {print}' data.txt

# Range patterns
awk '/START/,/END/ {print}' file.txt    # From START to END

# Negation
awk '!/error/ {print}' log.txt          # Lines NOT containing error
```

### **BEGIN and END Blocks:**

```bash
#!/bin/bash

# BEGIN: executed once before processing
awk 'BEGIN {print "=== Report ==="} {print $0}' file.txt

# END: executed once after processing
awk '{sum += $1} END {print "Total:", sum}' numbers.txt

# Combined
awk 'BEGIN {print "Name\tAge"} {print $1"\t"$2} END {print "---"}' data.txt

# Initialize variables
awk 'BEGIN {FS=":"; OFS="\t"} {print $1, $NF}' /etc/passwd
```

---

## 🔢 Mathematical Operations

```bash
#!/bin/bash

# Create sample data
cat > numbers.txt << 'EOF'
10
20
30
40
50
EOF

# Sum
awk '{sum += $1} END {print "Sum:", sum}' numbers.txt
# Output: Sum: 150

# Average
awk '{sum += $1; count++} END {print "Average:", sum/count}' numbers.txt
# Output: Average: 30

# Maximum
awk 'BEGIN {max = 0} {if ($1 > max) max = $1} END {print "Max:", max}' numbers.txt

# Minimum
awk 'NR == 1 {min = $1} {if ($1 < min) min = $1} END {print "Min:", min}' numbers.txt

# Count lines
awk 'END {print NR}' file.txt

# Arithmetic in output
awk '{print $1, $2, $1*$2}' data.txt    # Multiply columns
```

---

## 🎨 Conditional Statements

### **if-else:**

```bash
#!/bin/bash

# Simple if
awk '{if ($3 > 30) print $1}' data.txt

# if-else
awk '{if ($2 == "active") print $1 " is active"; else print $1 " is inactive"}' data.txt

# if-else if-else
awk '{
    if ($3 >= 90) grade = "A"
    else if ($3 >= 80) grade = "B"
    else if ($3 >= 70) grade = "C"
    else grade = "F"
    print $1, grade
}' scores.txt

# Ternary operator
awk '{status = ($2 == "active") ? "ACTIVE" : "INACTIVE"; print $1, status}' data.txt
```

---

## 🔁 Loops

### **for Loop:**

```bash
#!/bin/bash

# Print all fields
awk '{for (i=1; i<=NF; i++) print $i}' file.txt

# Print with index
awk '{for (i=1; i<=NF; i++) print i ":", $i}' file.txt

# Process range
awk '{for (i=2; i<=NF; i++) sum += $i; print sum}' file.txt
```

### **while Loop:**

```bash
#!/bin/bash

# Print fields
awk '{i=1; while (i<=NF) {print $i; i++}}' file.txt
```

---

## 📋 Arrays

### **Associative Arrays:**

```bash
#!/bin/bash

# Count occurrences
awk '{count[$1]++} END {for (word in count) print word, count[word]}' file.txt

# Sum by category
awk '{sum[$1] += $2} END {for (cat in sum) print cat, sum[cat]}' data.txt

# Create lookup table
awk 'BEGIN {
    names["john"] = "John Doe"
    names["jane"] = "Jane Smith"
    print names["john"]
}'

# Multi-dimensional (using concatenation)
awk '{matrix[$1,$2] = $3}' data.txt
```

---

## 🎯 Practical Examples

### **Example 1: Process /etc/passwd**

```bash
#!/bin/bash

# Extract usernames and home directories
awk -F':' '{print $1, $6}' /etc/passwd

# Find users with bash shell
awk -F':' '$NF == "/bin/bash" {print $1}' /etc/passwd

# Users with UID >= 1000 (regular users)
awk -F':' '$3 >= 1000 {print $1, $3}' /etc/passwd

# Count shells
awk -F':' '{shells[$NF]++} END {for (s in shells) print s, shells[s]}' /etc/passwd
```

### **Example 2: Log Analysis**

```bash
#!/bin/bash

# Create sample log
cat > access.log << 'EOF'
192.168.1.1 GET /index.html 200 1024
192.168.1.2 GET /about.html 200 2048
192.168.1.1 GET /contact.html 404 512
192.168.1.3 POST /api/login 200 256
192.168.1.2 GET /index.html 200 1024
EOF

# Count requests per IP
awk '{ip[$1]++} END {for (i in ip) print i, ip[i]}' access.log

# Sum bytes transferred
awk '{total += $5} END {print "Total bytes:", total}' access.log

# Count 404 errors
awk '$4 == 404 {count++} END {print "404 errors:", count}' access.log

# Top requested pages
awk '{pages[$3]++} END {for (p in pages) print p, pages[p]}' access.log | sort -k2 -rn

# Clean up
rm access.log
```

### **Example 3: CSV Processing**

```bash
#!/bin/bash

# Create CSV
cat > sales.csv << 'EOF'
Product,Quantity,Price
Apple,10,1.50
Banana,20,0.80
Orange,15,1.20
Apple,5,1.50
EOF

# Calculate revenue
awk -F',' 'NR > 1 {revenue = $2 * $3; print $1, revenue}' sales.csv

# Total revenue
awk -F',' 'NR > 1 {total += $2 * $3} END {print "Total: $" total}' sales.csv

# Group by product
awk -F',' 'NR > 1 {qty[$1] += $2} END {for (p in qty) print p ":", qty[p]}' sales.csv

# Format as table
awk -F',' 'BEGIN {printf "%-10s %10s %10s\n", "Product", "Qty", "Price"}
           NR > 1 {printf "%-10s %10d %10.2f\n", $1, $2, $3}' sales.csv

rm sales.csv
```

### **Example 4: Generate Report**

```bash
#!/bin/bash

cat > grades.txt << 'EOF'
Alice Math 95
Alice Science 88
Bob Math 78
Bob Science 92
Charlie Math 85
Charlie Science 90
EOF

# Calculate average per student
awk '{
    scores[$1] += $3
    count[$1]++
}
END {
    print "Student Averages:"
    print "================="
    for (student in scores) {
        avg = scores[student] / count[student]
        printf "%-10s: %.2f\n", student, avg
    }
}' grades.txt

# Subject averages
awk '{
    scores[$2] += $3
    count[$2]++
}
END {
    print "\nSubject Averages:"
    print "================="
    for (subject in scores) {
        avg = scores[subject] / count[subject]
        printf "%-10s: %.2f\n", subject, avg
    }
}' grades.txt

rm grades.txt
```

### **Example 5: System Monitoring**

```bash
#!/bin/bash

# Analyze disk usage
df -h | awk 'NR > 1 {
    gsub(/%/, "", $5)
    if ($5 > 80)
        print "WARNING:", $6, "is", $5 "% full"
    else if ($5 > 90)
        print "CRITICAL:", $6, "is", $5 "% full"
}'

# Process top CPU consumers
ps aux | awk 'NR > 1 {
    if ($3 > 50)
        printf "High CPU: %s (%.1f%%)\n", $11, $3
}'

# Memory usage summary
free -h | awk '/^Mem:/ {
    printf "Total: %s\nUsed: %s\nFree: %s\n", $2, $3, $4
}'
```

---

## 🎯 Advanced awk Techniques

### **Functions:**

```bash
#!/bin/bash

# Built-in functions
awk '{print length($0)}' file.txt              # String length
awk '{print substr($1, 1, 3)}' file.txt        # Substring
awk '{print toupper($1)}' file.txt             # Uppercase
awk '{print tolower($1)}' file.txt             # Lowercase
awk '{print sqrt($1)}' numbers.txt             # Square root
awk '{print int($1)}' numbers.txt              # Integer part

# User-defined functions
awk '
function double(x) {
    return x * 2
}
{
    print $1, double($1)
}' numbers.txt
```

### **Multiple Input Files:**

```bash
#!/bin/bash

# Process multiple files differently
awk '
FNR == 1 {print "Processing:", FILENAME}
{print $0}
' file1.txt file2.txt

# Merge files by key
awk '
NR == FNR {a[$1] = $2; next}
{print $1, $2, a[$1]}
' lookup.txt data.txt
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you print the third column?**
   <details>
   <summary>Answer</summary>
   awk '{print $3}' file.txt
   </details>

2. **How do you sum all values in first column?**
   <details>
   <summary>Answer</summary>
   awk '{sum += $1} END {print sum}' file.txt
   </details>

3. **How do you use a custom field separator?**
   <details>
   <summary>Answer</summary>
   awk -F',' '{print $1}' file.csv (comma separator)
   </details>

4. **What does NR represent?**
   <details>
   <summary>Answer</summary>
   Number of Records (line number)
   </details>

5. **How do you print the last field?**
   <details>
   <summary>Answer</summary>
   awk '{print $NF}' file.txt
   </details>

---

## 📝 Key Takeaways

✅ **awk 'pattern {action}' file**  
✅ **$1, $2, ... are columns; $0 is entire line**  
✅ **NR = line number, NF = number of fields**  
✅ **-F to set field separator**  
✅ **BEGIN and END for initialization/summary**  
✅ **Arrays for aggregation and grouping**  
✅ **Built-in math and string functions**  

---

## 🚀 Next Steps

You're now an awk expert!

**Next lesson:** [18 - Process Management](18-process-management.md) - Managing processes and signals

---

## 💡 Pro Tips

**One-liner sum:**
```bash
awk '{s+=$1} END {print s}' numbers.txt
```

**Pretty table:**
```bash
awk '{printf "%-20s %10s\n", $1, $2}' file.txt
```

**Skip header:**
```bash
awk 'NR > 1 {print}' file.csv
```

awk is your data processing Swiss Army knife! 🔧
