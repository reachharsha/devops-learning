---
render_with_liquid: false
---
# 05 - Parameter Expansion

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What parameter expansion is and why it's powerful
- Basic variable expansion syntax
- Default values and error handling
- String manipulation techniques
- Length, substring, and pattern matching
- Case conversion and replacement
- Array expansion

---

## 🔧 What is Parameter Expansion?

**Real-World Analogy:**
```
Regular Variable:
$NAME
→ Get value

Parameter Expansion:
${NAME}
${NAME:-default}
${NAME#pattern}
${NAME/search/replace}
→ Get value AND manipulate it

Think of it as:
Variable = Toolbox
Parameter Expansion = Swiss Army Knife
→ Get, modify, check, all in one!
```

**Why Use ${} Instead of $?**
```bash
# Without braces (simple)
echo $NAME

# With braces (required for manipulation)
echo ${NAME}

# When braces are required:
echo "$NAMETest"     # ❌ Looks for variable NAMETest
echo "${NAME}Test"   # ✅ Appends "Test" to NAME

# Always safer to use ${NAME}
```

---

## 📦 Basic Parameter Expansion

### **Simple Expansion:**

```bash
NAME="Alice"

# Basic expansion
echo $NAME           # Alice
echo ${NAME}         # Alice (same, but clearer)

# With concatenation
echo "$NAMEsmith"    # Empty! (looks for NAMEsmith variable)
echo "${NAME}smith"  # Alicesmith ✅

# In strings
echo "Hello, $NAME!"        # Hello, Alice!
echo "Hello, ${NAME}!"      # Hello, Alice!

# Multiple variables
FIRST="John"
LAST="Doe"
echo "${FIRST} ${LAST}"     # John Doe
```

---

## 🎯 Default Values

### **Use Default if Unset (:-)**

```bash
# ${var:-default}
# If var is unset or empty, use default

NAME=""
echo "${NAME:-Anonymous}"    # Anonymous (NAME is empty)

NAME="Alice"
echo "${NAME:-Anonymous}"    # Alice (NAME is set)

unset NAME
echo "${NAME:-Anonymous}"    # Anonymous (NAME is unset)

# Practical example
#!/bin/bash
ENVIRONMENT=${ENV:-development}
echo "Running in: $ENVIRONMENT"

# Running without ENV set:
# Output: Running in: development

# Running with ENV=production:
# Output: Running in: production
```

### **Assign Default if Unset (:=)**

```bash
# ${var:=default}
# If var is unset or empty, assign default AND return it

unset NAME
echo "${NAME:=Alice}"        # Alice
echo "$NAME"                 # Alice (variable was assigned!)

# Practical example
#!/bin/bash
LOG_DIR=${LOG_DIR:=/var/log/myapp}
echo "Logging to: $LOG_DIR"

# Creates LOG_DIR variable if it doesn't exist
```

### **Use Alternate Value if Set (:+)**

```bash
# ${var:+alternate}
# If var is set, use alternate, otherwise use nothing

NAME="Alice"
echo "${NAME:+User is set}"      # User is set

unset NAME
echo "${NAME:+User is set}"      # (empty)

# Practical example
#!/bin/bash
DEBUG=1
echo "Running${DEBUG:+ in DEBUG mode}..."

# With DEBUG set: Running in DEBUG mode...
# Without DEBUG: Running...
```

### **Error if Unset (:?)**

```bash
# ${var:?error message}
# If var is unset, print error and exit

#!/bin/bash
set -u  # Exit on undefined variable

DATABASE=${DATABASE:?Error: DATABASE not set}

# If DATABASE is not set:
# Output: script.sh: line 4: DATABASE: Error: DATABASE not set
# Script exits!

# Practical example
#!/bin/bash
API_KEY=${API_KEY:?Error: API_KEY environment variable required}
echo "Using API key: $API_KEY"
```

---

## 📏 String Length

```bash
NAME="Alice"

# Get length
echo ${#NAME}               # 5

# Empty string
EMPTY=""
echo ${#EMPTY}              # 0

# Practical example
#!/bin/bash
PASSWORD="secret123"

if [[ ${#PASSWORD} -lt 8 ]]; then
    echo "Password too short! Must be at least 8 characters."
    exit 1
fi

echo "Password length: ${#PASSWORD} characters ✅"
```

---

## ✂️ Substring Extraction

### **${var:offset:length}**

```bash
TEXT="Hello, World!"

# From position 0, length 5
echo ${TEXT:0:5}            # Hello

# From position 7, length 5
echo ${TEXT:7:5}            # World

# From position 7 to end
echo ${TEXT:7}              # World!

# Negative offset (from end)
echo ${TEXT: -6}            # World! (note the space after :)
echo ${TEXT: -6:5}          # World

# Practical example
DATE="2026-01-23"
YEAR=${DATE:0:4}            # 2026
MONTH=${DATE:5:2}           # 01
DAY=${DATE:8:2}             # 23

echo "Year: $YEAR, Month: $MONTH, Day: $DAY"
```

---

## 🎨 Pattern Matching & Removal

### **Remove from Beginning (#)**

```bash
# ${var#pattern}   - Remove shortest match
# ${var##pattern}  - Remove longest match

FILEPATH="/home/user/documents/file.txt"

# Remove shortest match from beginning
echo ${FILEPATH#*/}         # home/user/documents/file.txt

# Remove longest match from beginning
echo ${FILEPATH##*/}        # file.txt (filename only!)

# Extract extension
FILENAME="document.tar.gz"
echo ${FILENAME#*.}         # tar.gz (first extension)
echo ${FILENAME##*.}        # gz (last extension)

# Remove prefix
URL="https://www.example.com/page"
echo ${URL#https://}        # www.example.com/page
```

### **Remove from End (%)**

```bash
# ${var%pattern}   - Remove shortest match from end
# ${var%%pattern}  - Remove longest match from end

FILEPATH="/home/user/documents/file.txt"

# Remove shortest match from end
echo ${FILEPATH%/*}         # /home/user/documents (directory only!)

# Remove longest match from end
echo ${FILEPATH%%/*}        # (empty - everything removed)

# Remove extension
FILENAME="document.tar.gz"
echo ${FILENAME%.*}         # document.tar
echo ${FILENAME%%.*}        # document (all extensions removed!)

# Practical example
#!/bin/bash
LOGFILE="/var/log/app.log"

# Get directory
DIR=${LOGFILE%/*}           # /var/log

# Get filename
FILE=${LOGFILE##*/}         # app.log

# Get filename without extension
BASENAME=${FILE%.*}         # app

echo "Dir: $DIR, File: $FILE, Base: $BASENAME"
```

---

## 🔄 Search and Replace

### **Replace First Match (/)**

```bash
# ${var/pattern/replacement}

TEXT="Hello World World"

# Replace first occurrence
echo ${TEXT/World/Universe}     # Hello Universe World

# Replace with nothing (delete first)
echo ${TEXT/World/}             # Hello  World

# Practical example
PATH_STRING="/usr/local/bin:/usr/bin:/bin"
echo ${PATH_STRING/:/,}         # /usr/local/bin,/usr/bin:/bin
```

### **Replace All Matches (//)**

```bash
# ${var//pattern/replacement}

TEXT="Hello World World World"

# Replace all occurrences
echo ${TEXT//World/Universe}    # Hello Universe Universe Universe

# Remove all spaces
TEXT="H e l l o"
echo ${TEXT// /}                # Hello

# Practical example
#!/bin/bash
CSV="apple,banana,cherry,date"

# Convert CSV to space-separated
SPACE_SEP=${CSV//,/ }
echo $SPACE_SEP                 # apple banana cherry date

# Convert to newline-separated
NEWLINE_SEP=${CSV//,/$'\n'}
echo "$NEWLINE_SEP"
# apple
# banana
# cherry
# date
```

### **Anchored Replacement**

```bash
# ${var/#pattern/replacement}  - Replace at beginning
# ${var/%pattern/replacement}  - Replace at end

TEXT="Hello World"

# Replace at beginning
echo ${TEXT/#Hello/Hi}          # Hi World
echo ${TEXT/#World/Hi}          # Hello World (no match at beginning)

# Replace at end
echo ${TEXT/%World/Universe}    # Hello Universe
echo ${TEXT/%Hello/Hi}          # Hello World (no match at end)

# Practical example
#!/bin/bash
URL="http://example.com"

# Ensure HTTPS
URL=${URL/#http:/https:}
echo $URL                       # https://example.com
```

---

## 🔡 Case Conversion

```bash
# ${var^}   - Uppercase first letter
# ${var^^}  - Uppercase all
# ${var,}   - Lowercase first letter
# ${var,,}  - Lowercase all

NAME="alice"

# Uppercase first letter
echo ${NAME^}               # Alice

# Uppercase all
echo ${NAME^^}              # ALICE

# Lowercase conversions
NAME="ALICE"
echo ${NAME,}               # aLICE
echo ${NAME,,}              # alice

# Practical example
#!/bin/bash
read -p "Enter your name: " name

# Capitalize first letter
name_proper=${name,,}       # all lowercase first
name_proper=${name_proper^} # capitalize first

echo "Hello, $name_proper!"

# Input: alice
# Output: Hello, Alice!
```

---

## 📚 Array Expansion

### **Basic Array Expansion:**

```bash
# Define array
COLORS=("red" "green" "blue")

# Expand all elements
echo ${COLORS[@]}           # red green blue
echo ${COLORS[*]}           # red green blue

# Expand specific element
echo ${COLORS[0]}           # red
echo ${COLORS[1]}           # green

# Number of elements
echo ${#COLORS[@]}          # 3

# Length of element
echo ${#COLORS[0]}          # 3 (length of "red")

# Get indices
echo ${!COLORS[@]}          # 0 1 2
```

### **Array Slicing:**

```bash
NUMBERS=(10 20 30 40 50)

# Slice: ${array[@]:offset:length}
echo ${NUMBERS[@]:1:3}      # 20 30 40

# From position 2 to end
echo ${NUMBERS[@]:2}        # 30 40 50

# Last 2 elements
echo ${NUMBERS[@]: -2}      # 40 50
```

### **Pattern Matching on Arrays:**

```bash
FILES=("file.txt" "document.pdf" "image.jpg" "script.sh")

# Remove pattern from all elements
echo ${FILES[@]%.* }        # file document image script

# Replace in all elements
echo ${FILES[@]//file/doc}  # doc.txt document.pdf image.jpg script.sh
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does ${NAME:-default} do?**
   <details>
   <summary>Answer</summary>
   Returns default if NAME is unset or empty, doesn't assign it
   </details>

2. **What's the difference between # and ##?**
   <details>
   <summary>Answer</summary>
   # removes shortest match from beginning, ## removes longest match
   </details>

3. **How do you get string length?**
   <details>
   <summary>Answer</summary>
   ${#var}
   </details>

4. **How do you replace all occurrences?**
   <details>
   <summary>Answer</summary>
   ${var//pattern/replacement}
   </details>

5. **How do you uppercase all letters?**
   <details>
   <summary>Answer</summary>
   ${var^^}
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Default Values**

```bash
#!/bin/bash

# Test default values
unset USERNAME
echo "User: ${USERNAME:-guest}"

USERNAME="alice"
echo "User: ${USERNAME:-guest}"

# Assign default
unset LOG_LEVEL
echo "Level: ${LOG_LEVEL:=INFO}"
echo "LOG_LEVEL is now: $LOG_LEVEL"
```

### **Exercise 2: File Path Manipulation**

```bash
#!/bin/bash

FILEPATH="/home/user/documents/report.pdf"

# Extract parts
DIR=${FILEPATH%/*}
FILE=${FILEPATH##*/}
NAME=${FILE%.*}
EXT=${FILE##*.}

echo "Full Path: $FILEPATH"
echo "Directory: $DIR"
echo "Filename:  $FILE"
echo "Basename:  $NAME"
echo "Extension: $EXT"
```

### **Exercise 3: String Processing**

```bash
#!/bin/bash

TEXT="  Hello, World!  "

# Remove leading/trailing spaces using patterns
TEXT=${TEXT#"${TEXT%%[![:space:]]*}"}  # Remove leading
TEXT=${TEXT%"${TEXT##*[![:space:]]}"}  # Remove trailing

echo "Trimmed: '$TEXT'"

# Replace and convert
TEXT=${TEXT//,/}                       # Remove comma
TEXT=${TEXT^^}                         # Uppercase all

echo "Processed: '$TEXT'"              # HELLO WORLD!
```

### **Exercise 4: URL Parser**

```bash
#!/bin/bash

URL="https://www.example.com:8080/path/to/page?query=123"

# Extract protocol
PROTOCOL=${URL%%:*}                    # https

# Remove protocol
TEMP=${URL#*://}                       # www.example.com:8080/path/to/page?query=123

# Extract domain and port
DOMAIN_PORT=${TEMP%%/*}                # www.example.com:8080
DOMAIN=${DOMAIN_PORT%%:*}              # www.example.com
PORT=${DOMAIN_PORT##*:}                # 8080

# Extract path and query
PATH_QUERY=${TEMP#*/}                  # path/to/page?query=123
PATH=${PATH_QUERY%%\?*}                # path/to/page
QUERY=${PATH_QUERY#*\?}                # query=123

echo "Protocol: $PROTOCOL"
echo "Domain:   $DOMAIN"
echo "Port:     $PORT"
echo "Path:     $PATH"
echo "Query:    $QUERY"
```

---

## 📝 Key Takeaways

✅ **${var}** is safer than $var  
✅ **${var:-default}** provides fallback values  
✅ **${#var}** gets string length  
✅ **${var:offset:length}** extracts substring  
✅ **${var#pattern}** removes from beginning  
✅ **${var%pattern}** removes from end  
✅ **${var//old/new}** replaces all occurrences  
✅ **${var^^}** converts to uppercase  
✅ **${var,,}** converts to lowercase  
✅ **Works on arrays** with [@] or [*]  

---

## 🚀 Next Steps

You now understand powerful parameter expansion!

**Next lesson:** [06-arithmetic-and-brace-expansion.md](06-arithmetic-and-brace-expansion.md) - Math and sequences

---

## 💡 Pro Tips

**Combining Techniques:**
```bash
# Clean and validate input
read -p "Enter username: " USERNAME

# Remove spaces, lowercase, check length
USERNAME=${USERNAME// /}               # Remove spaces
USERNAME=${USERNAME,,}                 # Lowercase
USERNAME=${USERNAME:-default_user}     # Default if empty

if [[ ${#USERNAME} -lt 3 ]]; then
    echo "Username too short!"
    exit 1
fi

echo "Valid username: $USERNAME"
```

**Common Patterns:**
```bash
# Remove file extension
FILE="document.txt"
${FILE%.*}                             # document

# Get file extension
${FILE##*.}                            # txt

# Get filename from path
PATH="/home/user/file.txt"
${PATH##*/}                            # file.txt

# Get directory from path
${PATH%/*}                             # /home/user

# Ensure trailing slash
DIR="/home/user"
${DIR%/}/                              # /home/user/
```

**Debugging:**
```bash
# See exactly what expansion produces
set -x
NAME="Alice"
echo "${NAME:-Default}"
set +x
```

Parameter expansion is your Swiss Army knife for string manipulation! 🔧
