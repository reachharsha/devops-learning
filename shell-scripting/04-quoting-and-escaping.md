---
render_with_liquid: false
---
# 04 - Quoting & Escaping

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Single quotes vs double quotes vs no quotes
- When and how to escape special characters
- Word splitting and globbing
- The IFS (Internal Field Separator) variable
- Best practices for handling spaces and special characters

---

## 🎭 The Three Types of Quoting

### **No Quotes (Dangerous!):**

```bash
#!/bin/bash

FILE="my document.txt"

# ❌ Without quotes (WRONG!)
ls $FILE
# Error: ls tries to find "my" and "document.txt" (two files)

# ✅ With quotes (CORRECT!)
ls "$FILE"
# Works correctly: treats as single filename
```

### **Double Quotes (" "):**

**Allow variable expansion and command substitution:**

```bash
#!/bin/bash

NAME="Alice"
COUNT=5

# Variables are expanded
echo "Hello, $NAME"
# Output: Hello, Alice

echo "You have $COUNT messages"
# Output: You have 5 messages

# Command substitution works
echo "Current directory: $(pwd)"
# Output: Current directory: /home/user

# Preserves spaces
MESSAGE="Hello     World"
echo "$MESSAGE"
# Output: Hello     World (preserves multiple spaces)
```

### **Single Quotes (' '):**

**Treat everything literally (no expansion):**

```bash
#!/bin/bash

NAME="Alice"
COUNT=5

# Variables NOT expanded
echo 'Hello, $NAME'
# Output: Hello, $NAME (literal)

echo 'You have $COUNT messages'
# Output: You have $COUNT messages (literal)

# Command substitution NOT works
echo 'Current directory: $(pwd)'
# Output: Current directory: $(pwd) (literal)

# Use for literal strings
echo 'Cost: $100'
# Output: Cost: $100
```

---

## 📊 Comparison Table

| Feature | No Quotes | Double Quotes `" "` | Single Quotes `' '` |
|---------|-----------|-------------------|-------------------|
| **Variable expansion** | ✅ Yes | ✅ Yes | ❌ No (literal) |
| **Command substitution** | ✅ Yes | ✅ Yes | ❌ No (literal) |
| **Word splitting** | ✅ Yes | ❌ No | ❌ No |
| **Glob expansion** | ✅ Yes | ❌ No | ❌ No |
| **Preserves spaces** | ❌ No | ✅ Yes | ✅ Yes |
| **Special chars** | Active | Most escaped | All escaped |
| **Use case** | ⚠️ Rarely | 🎯 Most common | 📝 Literal strings |

---

## 🔍 Word Splitting Explained

**Word splitting breaks strings on spaces/tabs/newlines:**

```bash
#!/bin/bash

FILES="file1.txt file2.txt file3.txt"

# Without quotes: each word becomes separate argument
for file in $FILES; do
    echo "Processing: $file"
done
# Output:
# Processing: file1.txt
# Processing: file2.txt
# Processing: file3.txt

# With quotes: treated as single string
for file in "$FILES"; do
    echo "Processing: $file"
done
# Output:
# Processing: file1.txt file2.txt file3.txt
```

### **The Space Problem:**

```bash
#!/bin/bash

FILENAME="my document.txt"

# ❌ WRONG: word splitting breaks filename
cat $FILENAME
# Error: cat tries to find "my" and "document.txt"

# ✅ CORRECT: quotes preserve the space
cat "$FILENAME"
# Works: treats as single filename "my document.txt"

# Real example: deleting files
rm $FILENAME          # ❌ Tries to delete "my" and "document.txt"
rm "$FILENAME"        # ✅ Deletes "my document.txt"
```

---

## 🌟 Glob Expansion (Wildcards)

**Wildcards expand to matching files:**

```bash
#!/bin/bash

# Without quotes: * expands to filenames
echo *.txt
# Output: file1.txt file2.txt notes.txt

# With double quotes: * is literal
echo "*.txt"
# Output: *.txt (not expanded)

# With single quotes: * is literal
echo '*.txt'
# Output: *.txt (not expanded)

# Practical example
FILES=*.log

# Unquoted: expands
for file in $FILES; do
    echo "$file"    # Each matching file
done

# Quoted: literal
for file in "$FILES"; do
    echo "$file"    # Just the string "*.log"
done
```

---

## 🛡️ Escaping Special Characters

### **The Backslash \\:**

```bash
#!/bin/bash

# Escape special characters
echo "Price: \$100"
# Output: Price: $100

echo "Path: C:\\Users\\Documents"
# Output: Path: C:\Users\Documents

echo "Quote: \"Hello\""
# Output: Quote: "Hello"

# Escape newline (continue line)
echo "This is a very long \
line that continues"
# Output: This is a very long line that continues

# Special characters to escape
echo "\$ \\ \" \` \! \* \?"
# Output: $ \ " ` ! * ?
```

### **Special Characters Reference:**

| Character | Meaning | Escape |
|-----------|---------|--------|
| `$` | Variable | `\$` |
| `\` | Escape char | `\\` |
| `"` | Quote | `\"` |
| `` ` `` | Command sub | ``\` `` |
| `!` | History expansion | `\!` |
| `*` | Wildcard | `\*` |
| `?` | Single char wildcard | `\?` |
| `[` `]` | Character class | `\[` `\]` |
| `~` | Home directory | `\~` |
| `&` | Background | `\&` |
| `;` | Command separator | `\;` |
| `|` | Pipe | `\|` |
| `<` `>` | Redirection | `\<` `\>` |

---

## 🎯 Practical Examples

### **Example 1: File Operations with Spaces:**

```bash
#!/bin/bash

# Create file with spaces
touch "my document.txt"

# ❌ WRONG WAYS:
cat my document.txt        # Error: looks for "my" and "document.txt"
cat $FILE                  # Error: same issue

# ✅ CORRECT WAYS:
FILE="my document.txt"
cat "$FILE"                # ✅ Correct
cat "my document.txt"      # ✅ Correct
cat my\ document.txt       # ✅ Correct (escaped space)

# Delete safely
rm "$FILE"                 # ✅ Correct
```

### **Example 2: Working with User Input:**

```bash
#!/bin/bash

echo "Enter filename:"
read FILENAME

# ❌ DANGEROUS:
cat $FILENAME              # Breaks with spaces

# ✅ SAFE:
cat "$FILENAME"            # Always quote user input!

# ❌ DANGEROUS:
rm $FILENAME               # Could delete wrong files!

# ✅ SAFE:
rm "$FILENAME"             # Safe deletion
```

### **Example 3: Command Substitution:**

```bash
#!/bin/bash

# Double quotes: variable expansion works
CURRENT_USER=$(whoami)
echo "Logged in as: $CURRENT_USER"
# Output: Logged in as: john

# Single quotes: no expansion
echo 'Logged in as: $CURRENT_USER'
# Output: Logged in as: $CURRENT_USER

# Nested quotes
echo "User: $(whoami), Home: $HOME"
# Output: User: john, Home: /home/john
```

---

## 🔧 The IFS Variable

**IFS (Internal Field Separator) controls word splitting:**

```bash
#!/bin/bash

# Default IFS: space, tab, newline
DATA="apple banana cherry"

for item in $DATA; do
    echo "Item: $item"
done
# Output:
# Item: apple
# Item: banana
# Item: cherry

# Change IFS to comma
IFS=','
DATA="apple,banana,cherry"

for item in $DATA; do
    echo "Item: $item"
done
# Output:
# Item: apple
# Item: banana
# Item: cherry

# Always restore IFS!
IFS=' '    # Or: IFS=$' \t\n'
```

### **Practical IFS Usage:**

```bash
#!/bin/bash

# Parse CSV file
while IFS=',' read -r name age city; do
    echo "Name: $name, Age: $age, City: $city"
done < users.csv

# Parse /etc/passwd
while IFS=':' read -r username password uid gid comment home shell; do
    echo "User: $username, Home: $home, Shell: $shell"
done < /etc/passwd

# Split path
OLD_IFS="$IFS"
IFS=':'
for dir in $PATH; do
    echo "Directory: $dir"
done
IFS="$OLD_IFS"
```

---

## 💡 ANSI-C Quoting ($'...')

**Support escape sequences like \n, \t:**

```bash
#!/bin/bash

# Regular double quotes: \n is literal
echo "Line 1\nLine 2"
# Output: Line 1\nLine 2

# ANSI-C quoting: \n becomes newline
echo $'Line 1\nLine 2'
# Output:
# Line 1
# Line 2

# Useful escape sequences
echo $'Tab:\there'           # Tab character
echo $'Quote:\''             # Single quote inside $'...'
echo $'Newline:\n'           # Newline
echo $'Backslash:\\'         # Backslash
echo $'Alert:\a'             # Bell/alert sound
echo $'Carriage return:\r'   # Carriage return

# Practical use
MESSAGE=$'Error:\n  File not found\n  Please check path'
echo "$MESSAGE"
# Output:
# Error:
#   File not found
#   Please check path
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between single and double quotes?**
   <details>
   <summary>Answer</summary>
   Single quotes: everything is literal (no expansion)
   Double quotes: variables and commands expand
   </details>

2. **How do you include a literal $ in a double-quoted string?**
   <details>
   <summary>Answer</summary>
   Escape it with backslash: \$
   echo "Price: \$100"
   </details>

3. **Why should you quote variables?**
   <details>
   <summary>Answer</summary>
   To prevent word splitting and glob expansion, especially with filenames containing spaces
   </details>

4. **What does IFS control?**
   <details>
   <summary>Answer</summary>
   Internal Field Separator - controls how bash splits words (default: space, tab, newline)
   </details>

5. **How do you include a newline in a string?**
   <details>
   <summary>Answer</summary>
   Use ANSI-C quoting: echo $'Line 1\nLine 2'
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Quote Practice**

```bash
#!/bin/bash

NAME="John"
PRICE=100

# Test different quoting
echo "1. Double quotes: Hello $NAME, price is \$$PRICE"
echo '2. Single quotes: Hello $NAME, price is $PRICE'
echo 3. No quotes: Hello $NAME, price is $PRICE

# Output:
# 1. Double quotes: Hello John, price is $100
# 2. Single quotes: Hello $NAME, price is $PRICE
# 3. No quotes: Hello John, price is  (100 is treated as command)
```

### **Exercise 2: Filename Handler**

```bash
#!/bin/bash
# safe_file_handler.sh

# Create test file with spaces
TEST_FILE="my test file.txt"
echo "Sample content" > "$TEST_FILE"

echo "Filename: $TEST_FILE"
echo "---"

# Safe operations
echo "Contents:"
cat "$TEST_FILE"

echo "---"
echo "File size:"
ls -lh "$TEST_FILE" | awk '{print $5}'

# Clean up
rm "$TEST_FILE"
echo "File deleted safely"
```

### **Exercise 3: CSV Parser**

```bash
#!/bin/bash
# parse_csv.sh

# Create sample CSV
cat > users.csv << 'EOF'
John Doe,30,New York
Jane Smith,25,Los Angeles
Bob Johnson,35,Chicago
EOF

# Parse CSV
echo "============================"
echo "User Report"
echo "============================"

while IFS=',' read -r name age city; do
    printf "%-20s Age: %-3s City: %s\n" "$name" "$age" "$city"
done < users.csv

# Clean up
rm users.csv
```

### **Exercise 4: Special Characters**

```bash
#!/bin/bash
# special_chars.sh

# Test escaping
echo "Regular: Price is 100 dollars"
echo "Escaped: Price is \$100"
echo "Path: C:\\Users\\Documents"
echo "Quote: She said \"Hello\""

# Newlines with ANSI-C quoting
MENU=$'Select option:\n1. Start\n2. Stop\n3. Exit'
echo "$MENU"

# Tabs
HEADER=$'Name\tAge\tCity'
DATA=$'John\t30\tNY'
echo "$HEADER"
echo "$DATA"
```

---

## 📝 Key Takeaways

✅ **Always quote variables: "$VAR" not $VAR**  
✅ **Double quotes for variables: "Hello $NAME"**  
✅ **Single quotes for literal strings: 'Price: $100'**  
✅ **Escape special chars with \: \$, \\, \", \`**  
✅ **Use $'...' for escape sequences: $'Line\nBreak'**  
✅ **IFS controls word splitting (default: space/tab/newline)**  
✅ **Quote command substitution: "$(command)"**  
✅ **Unquoted variables undergo word splitting and globbing**  

---

## 🚀 Next Steps

You now understand quoting and escaping!

**Next lesson:** [05 - Parameter Expansion](05-parameter-expansion.md) - Advanced variable manipulation

---

## 💡 Pro Tips

**Golden rule: Quote everything:**
```bash
# ✅ Always safe
cat "$FILE"
rm "$DIRECTORY"/*
echo "$MESSAGE"

# ❌ Dangerous
cat $FILE
rm $DIRECTORY/*
echo $MESSAGE
```

**When NOT to quote:**
```bash
# Array expansion (need word splitting)
FILES=(file1.txt file2.txt)
cat ${FILES[@]}    # Unquoted to pass as separate arguments

# Intentional word splitting
OPTIONS="-l -a -h"
ls $OPTIONS        # Want each option separate
```

**Detect problems with shellcheck:**
```bash
# Install shellcheck
sudo apt install shellcheck    # Ubuntu/Debian
brew install shellcheck         # macOS

# Check script
shellcheck myscript.sh

# It catches:
# - Unquoted variables
# - Word splitting issues
# - Glob problems
```

**Safe pattern for user input:**
```bash
read -r -p "Enter filename: " FILENAME
# Always quote when using:
if [ -f "$FILENAME" ]; then
    cat "$FILENAME"
fi
```

Master quoting = Master bash! 🎯
