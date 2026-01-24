# 13 - String Manipulation

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- String length and substring extraction
- Pattern matching and replacement
- Case conversion (upper/lower)
- String trimming and padding
- Splitting and joining strings
- Regular expressions in bash
- Advanced text transformations

---

## 📏 String Length

```bash
#!/bin/bash

TEXT="Hello, World!"

# Get length
echo ${#TEXT}              # 13

# Practical use
PASSWORD="secret123"
if [ ${#PASSWORD} -lt 8 ]; then
    echo "Password too short (minimum 8 characters)"
fi

# Compare lengths
STRING1="hello"
STRING2="world"
if [ ${#STRING1} -eq ${#STRING2} ]; then
    echo "Strings have same length"
fi
```

---

## ✂️ Substring Extraction

### **Basic Extraction:**

```bash
#!/bin/bash

TEXT="Hello, World!"

# ${string:position:length}
echo ${TEXT:0:5}           # Hello (start at 0, length 5)
echo ${TEXT:7}             # World! (start at 7 to end)
echo ${TEXT:7:5}           # World (start at 7, length 5)

# Negative indices (from end)
echo ${TEXT: -6}           # World! (last 6 chars)
echo ${TEXT: -6:5}         # World (6 from end, length 5)

# Practical examples
DATE="2024-01-15"
YEAR=${DATE:0:4}           # 2024
MONTH=${DATE:5:2}          # 01
DAY=${DATE:8:2}            # 15

PHONE="123-456-7890"
AREA=${PHONE:0:3}          # 123
EXCHANGE=${PHONE:4:3}      # 456
NUMBER=${PHONE:8:4}        # 7890
```

---

## 🔍 Pattern Matching

### **Remove from Beginning (# and ##):**

```bash
#!/bin/bash

# # removes shortest match from start
# ## removes longest match from start

FILENAME="document.tar.gz"

echo ${FILENAME#*.}        # tar.gz (removes "document.")
echo ${FILENAME##*.}       # gz (removes "document.tar.")

# Remove path
FULLPATH="/home/user/docs/file.txt"
echo ${FULLPATH##*/}       # file.txt (removes path)

# Remove prefix
URL="https://www.example.com"
echo ${URL#https://}       # www.example.com
echo ${URL#http*://}       # www.example.com (works for http/https)
```

### **Remove from End (% and %%):**

```bash
#!/bin/bash

# % removes shortest match from end
# %% removes longest match from end

FILENAME="document.tar.gz"

echo ${FILENAME%.*}        # document.tar (removes ".gz")
echo ${FILENAME%%.*}       # document (removes ".tar.gz")

# Remove extension
FILE="report.pdf"
BASENAME=${FILE%.*}        # report

# Get directory
FULLPATH="/home/user/docs/file.txt"
echo ${FULLPATH%/*}        # /home/user/docs (removes filename)

# Remove suffix
URL="https://example.com/"
echo ${URL%/}              # https://example.com (removes trailing slash)
```

---

## 🔄 String Replacement

### **Replace First Occurrence:**

```bash
#!/bin/bash

TEXT="hello world hello"

# ${string/pattern/replacement}
echo ${TEXT/hello/hi}      # hi world hello (first only)

# Remove first occurrence (empty replacement)
echo ${TEXT/hello}         # world hello
```

### **Replace All Occurrences:**

```bash
#!/bin/bash

TEXT="hello world hello"

# ${string//pattern/replacement}
echo ${TEXT//hello/hi}     # hi world hi (all occurrences)

# Remove all occurrences
echo ${TEXT//hello}        # world  (removes all "hello")

# Replace spaces with underscores
FILENAME="my document file.txt"
SAFE=${FILENAME// /_}      # my_document_file.txt

# Replace multiple characters
PATH_WIN="C:\Users\John\Documents"
PATH_UNIX=${PATH_WIN//\\//}    # C:/Users/John/Documents
```

### **Replace at Beginning or End:**

```bash
#!/bin/bash

TEXT="Hello, World!"

# Replace at beginning: ${string/#pattern/replacement}
echo ${TEXT/#Hello/Hi}     # Hi, World!
echo ${TEXT/#World/Hi}     # Hello, World! (no match at start)

# Replace at end: ${string/%pattern/replacement}
echo ${TEXT/%World!/Earth!}    # Hello, Earth!
echo ${TEXT/%Hello/Hi}     # Hello, World! (no match at end)

# Practical: Fix URL protocol
URL="http://example.com"
SECURE=${URL/#http:/https:}    # https://example.com
```

---

## 🔤 Case Conversion

### **To Uppercase:**

```bash
#!/bin/bash

TEXT="hello world"

# First character
echo ${TEXT^}              # Hello world

# All characters
echo ${TEXT^^}             # HELLO WORLD

# Specific characters
echo ${TEXT^^[hw]}         # Hello World (only h and w)

# Practical use
read -p "Enter yes/no: " ANSWER
if [[ ${ANSWER^^} == "YES" ]]; then
    echo "You said yes"
fi
```

### **To Lowercase:**

```bash
#!/bin/bash

TEXT="HELLO WORLD"

# First character
echo ${TEXT,}              # hELLO WORLD

# All characters
echo ${TEXT,,}             # hello world

# Practical: Case-insensitive comparison
read -p "Continue? (Y/N): " INPUT
INPUT_LOWER=${INPUT,,}
if [[ $INPUT_LOWER == "y" || $INPUT_LOWER == "yes" ]]; then
    echo "Continuing..."
fi
```

### **Toggle Case:**

```bash
#!/bin/bash

TEXT="Hello World"

# Toggle case (requires bash 4+)
echo ${TEXT~~}             # hELLO wORLD
echo ${TEXT~}              # hello World (first char only)
```

---

## ✨ String Trimming

### **Trim Whitespace:**

```bash
#!/bin/bash

# Leading whitespace
trim_left() {
    local var="$1"
    var="${var#"${var%%[![:space:]]*}"}"
    echo "$var"
}

# Trailing whitespace
trim_right() {
    local var="$1"
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

# Both sides
trim() {
    local var="$1"
    var="${var#"${var%%[![:space:]]*}"}"
    var="${var%"${var##*[![:space:]]}"}"
    echo "$var"
}

# Usage
MESSY="   hello world   "
echo "'$(trim "$MESSY")'"    # 'hello world'
```

### **Using sed/awk:**

```bash
#!/bin/bash

TEXT="   hello world   "

# Using sed
echo "$TEXT" | sed 's/^[[:space:]]*//;s/[[:space:]]*$//'

# Using awk
echo "$TEXT" | awk '{$1=$1};1'

# Using xargs
echo "$TEXT" | xargs
```

---

## 📐 String Padding

```bash
#!/bin/bash

# Pad with spaces (printf)
printf "%-20s\n" "hello"       # hello               (left-aligned)
printf "%20s\n" "hello"        # hello (right-aligned)

# Pad with zeros
printf "%05d\n" 42             # 00042

# Custom padding
pad_right() {
    local string="$1"
    local width="$2"
    local pad_char="${3:- }"   # Default: space
    
    local padding=$((width - ${#string}))
    if [ $padding -gt 0 ]; then
        printf "%s%*s\n" "$string" $padding | tr ' ' "$pad_char"
    else
        echo "$string"
    fi
}

pad_right "hello" 10 "."       # hello.....
```

---

## ✂️ Splitting Strings

### **Split by Delimiter:**

```bash
#!/bin/bash

# Using IFS
DATA="apple,banana,cherry"

IFS=',' read -ra FRUITS <<< "$DATA"
for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# Split on newlines
TEXT=$'line1\nline2\nline3'
IFS=$'\n' read -ra LINES <<< "$TEXT"

# CSV parsing
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done << 'EOF'
John,30,NYC
Alice,25,LA
Bob,35,Chicago
EOF
```

### **Split into Array:**

```bash
#!/bin/bash

# Split string to array
string_to_array() {
    local string="$1"
    local delimiter="$2"
    local -n result=$3
    
    IFS="$delimiter" read -ra result <<< "$string"
}

PATH_STR="/usr/local/bin:/usr/bin:/bin"
declare -a PATHS
string_to_array "$PATH_STR" ":" PATHS

for path in "${PATHS[@]}"; do
    echo "Path: $path"
done
```

---

## 🔗 Joining Strings

```bash
#!/bin/bash

# Join array with delimiter
join_array() {
    local delimiter="$1"
    shift
    local items=("$@")
    
    local result="${items[0]}"
    for ((i=1; i<${#items[@]}; i++)); do
        result="${result}${delimiter}${items[$i]}"
    done
    
    echo "$result"
}

ITEMS=("apple" "banana" "cherry")
JOINED=$(join_array ", " "${ITEMS[@]}")
echo $JOINED              # apple, banana, cherry

# Simple concatenation
STRING1="Hello"
STRING2="World"
COMBINED="${STRING1} ${STRING2}"    # Hello World
```

---

## 🎯 Regular Expressions

### **Pattern Matching with =~:**

```bash
#!/bin/bash

# Email validation
validate_email() {
    local email=$1
    if [[ $email =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
        return 0
    else
        return 1
    fi
}

if validate_email "user@example.com"; then
    echo "Valid email"
fi

# Extract matched groups
TEXT="My phone is 123-456-7890"
if [[ $TEXT =~ ([0-9]{3})-([0-9]{3})-([0-9]{4}) ]]; then
    echo "Full match: ${BASH_REMATCH[0]}"    # 123-456-7890
    echo "Area code: ${BASH_REMATCH[1]}"     # 123
    echo "Exchange: ${BASH_REMATCH[2]}"      # 456
    echo "Number: ${BASH_REMATCH[3]}"        # 7890
fi

# IP address validation
is_valid_ip() {
    local ip=$1
    if [[ $ip =~ ^[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}$ ]]; then
        return 0
    else
        return 1
    fi
}
```

---

## 🎯 Practical Examples

### **Example 1: URL Parser**

```bash
#!/bin/bash

parse_url() {
    local url=$1
    
    # Extract protocol
    local protocol=${url%%://*}
    
    # Remove protocol
    local temp=${url#*://}
    
    # Extract domain and path
    local domain=${temp%%/*}
    local path=${temp#*/}
    
    # Extract port if present
    local port=""
    if [[ $domain == *:* ]]; then
        port=${domain##*:}
        domain=${domain%:*}
    fi
    
    echo "Protocol: $protocol"
    echo "Domain: $domain"
    echo "Port: ${port:-default}"
    echo "Path: /$path"
}

parse_url "https://example.com:8080/path/to/page"
```

### **Example 2: Filename Sanitizer**

```bash
#!/bin/bash

sanitize_filename() {
    local filename="$1"
    
    # Remove leading/trailing spaces
    filename=$(echo "$filename" | xargs)
    
    # Replace spaces with underscores
    filename=${filename// /_}
    
    # Remove special characters
    filename=${filename//[^a-zA-Z0-9._-]/}
    
    # Convert to lowercase
    filename=${filename,,}
    
    # Limit length
    if [ ${#filename} -gt 50 ]; then
        filename=${filename:0:50}
    fi
    
    echo "$filename"
}

UNSAFE="My Document (Final) [2024].txt"
SAFE=$(sanitize_filename "$UNSAFE")
echo "Safe filename: $SAFE"    # my_document_final_2024.txt
```

### **Example 3: Template Engine**

```bash
#!/bin/bash

# Simple template replacement
render_template() {
    local template="$1"
    declare -n vars=$2
    
    local result="$template"
    
    for key in "${!vars[@]}"; do
        local value="${vars[$key]}"
        result=${result//\{\{$key\}\}/$value}
    done
    
    echo "$result"
}

# Usage
TEMPLATE="Hello {{name}}, your order #{{order_id}} is {{status}}"

declare -A DATA=(
    [name]="John"
    [order_id]="12345"
    [status]="shipped"
)

render_template "$TEMPLATE" DATA
# Output: Hello John, your order #12345 is shipped
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you get string length?**
   <details>
   <summary>Answer</summary>
   ${#STRING}
   </details>

2. **How do you extract substring from position 5, length 3?**
   <details>
   <summary>Answer</summary>
   ${STRING:5:3}
   </details>

3. **How do you replace all occurrences of "old" with "new"?**
   <details>
   <summary>Answer</summary>
   ${STRING//old/new}
   </details>

4. **How do you convert string to uppercase?**
   <details>
   <summary>Answer</summary>
   ${STRING^^}
   </details>

5. **How do you remove file extension?**
   <details>
   <summary>Answer</summary>
   ${FILENAME%.*}
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: String Processor**

```bash
#!/bin/bash

read -p "Enter text: " INPUT

echo "===================="
echo "String Analysis"
echo "===================="
echo "Original: $INPUT"
echo "Length: ${#INPUT}"
echo "Uppercase: ${INPUT^^}"
echo "Lowercase: ${INPUT,,}"
echo "First 10 chars: ${INPUT:0:10}"
echo "Last 5 chars: ${INPUT: -5}"
echo "Reversed: $(echo "$INPUT" | rev)"
echo "Word count: $(echo "$INPUT" | wc -w)"
```

### **Exercise 2: CSV Parser**

```bash
#!/bin/bash

# Create sample CSV
cat > data.csv << 'EOF'
name,age,city,salary
John Doe,30,New York,75000
Jane Smith,25,Los Angeles,65000
Bob Johnson,35,Chicago,80000
EOF

echo "Employee Report"
echo "==============="

# Skip header, process data
tail -n +2 data.csv | while IFS=',' read -r name age city salary; do
    # Format salary
    formatted_salary=$(printf "%'d" $salary)
    
    printf "%-20s | Age: %2d | City: %-15s | Salary: \$%s\n" \
        "$name" "$age" "$city" "$formatted_salary"
done

rm data.csv
```

---

## 📝 Key Takeaways

✅ **String length: ${#STRING}**  
✅ **Substring: ${STRING:pos:len}**  
✅ **Remove prefix: ${STRING#pattern} or ${STRING##pattern}**  
✅ **Remove suffix: ${STRING%pattern} or ${STRING%%pattern}**  
✅ **Replace: ${STRING/old/new} or ${STRING//old/new}**  
✅ **Uppercase: ${STRING^^}, Lowercase: ${STRING,,}**  
✅ **Regex match: [[ $STRING =~ pattern ]]**  

---

## 🚀 Next Steps

You're now a string manipulation expert!

**Next lesson:** [14 - File Operations](14-file-operations.md) - Reading and writing files

---

## 💡 Pro Tips

**Remove leading zeros:**
```bash
NUM="00042"
echo $((10#$NUM))    # 42
```

**Extract filename and extension:**
```bash
FILE="document.tar.gz"
NAME=${FILE%%.*}     # document
EXT=${FILE#*.}       # tar.gz
```

Master strings, master scripting! 🎯
