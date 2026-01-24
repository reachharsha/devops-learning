# 12 - Arrays

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Indexed arrays (0-based lists)
- Associative arrays (key-value pairs / hash maps)
- Array operations (add, remove, update)
- Iterating over arrays
- Array slicing and manipulation
- Practical array use cases

---

## 📚 What are Arrays?

**Real-World Analogy:**  
Arrays are like numbered boxes (indexed) or labeled boxes (associative) where you store multiple items. Instead of creating separate variables for each item, you use one array to hold them all.

```bash
#!/bin/bash

# Individual variables (tedious)
SERVER1="web1.example.com"
SERVER2="web2.example.com"
SERVER3="web3.example.com"

# Array (better!)
SERVERS=("web1.example.com" "web2.example.com" "web3.example.com")
```

---

## 📦 Indexed Arrays

### **Creating Arrays:**

```bash
#!/bin/bash

# Method 1: Direct assignment
FRUITS=("apple" "banana" "cherry")

# Method 2: Individual elements
COLORS[0]="red"
COLORS[1]="green"
COLORS[2]="blue"

# Method 3: Empty array
EMPTY=()

# Method 4: From command output
FILES=($(ls *.txt))

# Method 5: Declare
declare -a NUMBERS
NUMBERS=(1 2 3 4 5)
```

### **Accessing Array Elements:**

```bash
#!/bin/bash

FRUITS=("apple" "banana" "cherry" "date")

# Access by index (0-based)
echo ${FRUITS[0]}      # apple
echo ${FRUITS[1]}      # banana
echo ${FRUITS[3]}      # date

# Last element
echo ${FRUITS[-1]}     # date
echo ${FRUITS[-2]}     # cherry

# All elements
echo ${FRUITS[@]}      # apple banana cherry date
echo ${FRUITS[*]}      # apple banana cherry date

# Number of elements
echo ${#FRUITS[@]}     # 4

# Length of element
echo ${#FRUITS[0]}     # 5 (length of "apple")
```

### **Adding Elements:**

```bash
#!/bin/bash

NUMBERS=(1 2 3)

# Append to end
NUMBERS+=(4)
echo ${NUMBERS[@]}     # 1 2 3 4

# Append multiple
NUMBERS+=(5 6 7)
echo ${NUMBERS[@]}     # 1 2 3 4 5 6 7

# Add at specific index
NUMBERS[10]=100
echo ${NUMBERS[@]}     # 1 2 3 4 5 6 7 100 (gaps allowed)

# Prepend (add to beginning)
NUMBERS=(0 "${NUMBERS[@]}")
echo ${NUMBERS[@]}     # 0 1 2 3 4 5 6 7 100
```

### **Removing Elements:**

```bash
#!/bin/bash

FRUITS=("apple" "banana" "cherry" "date")

# Remove element at index 1
unset FRUITS[1]
echo ${FRUITS[@]}      # apple cherry date (banana removed)

# Note: indices don't shift
echo ${FRUITS[0]}      # apple
echo ${FRUITS[1]}      # (empty)
echo ${FRUITS[2]}      # cherry

# Remove all elements
unset FRUITS
```

### **Updating Elements:**

```bash
#!/bin/bash

COLORS=("red" "green" "blue")

# Update specific element
COLORS[1]="yellow"
echo ${COLORS[@]}      # red yellow blue

# Replace all
COLORS=("orange" "purple" "pink")
echo ${COLORS[@]}      # orange purple pink
```

---

## 🔄 Iterating Over Arrays

### **Method 1: For Loop with [@]:**

```bash
#!/bin/bash

FRUITS=("apple" "banana" "cherry")

for fruit in "${FRUITS[@]}"; do
    echo "Fruit: $fruit"
done

# Output:
# Fruit: apple
# Fruit: banana
# Fruit: cherry
```

### **Method 2: For Loop with Indices:**

```bash
#!/bin/bash

COLORS=("red" "green" "blue")

for i in "${!COLORS[@]}"; do
    echo "Index $i: ${COLORS[$i]}"
done

# Output:
# Index 0: red
# Index 1: green
# Index 2: blue
```

### **Method 3: C-Style For Loop:**

```bash
#!/bin/bash

NUMBERS=(10 20 30 40 50)

for ((i=0; i<${#NUMBERS[@]}; i++)); do
    echo "Element $i: ${NUMBERS[$i]}"
done
```

---

## 🗺️ Associative Arrays (Hash Maps)

**Key-value pairs (Bash 4.0+):**

### **Creating Associative Arrays:**

```bash
#!/bin/bash

# Must declare with -A
declare -A USER

# Assign values
USER[name]="John Doe"
USER[email]="john@example.com"
USER[age]=30

# Or in one line
declare -A SCORES=(
    [alice]=95
    [bob]=87
    [charlie]=92
)
```

### **Accessing Associative Arrays:**

```bash
#!/bin/bash

declare -A CONFIG=(
    [host]="localhost"
    [port]=8080
    [database]="mydb"
)

# Access by key
echo ${CONFIG[host]}       # localhost
echo ${CONFIG[port]}       # 8080

# All values
echo ${CONFIG[@]}          # localhost 8080 mydb

# All keys
echo ${!CONFIG[@]}         # host port database

# Number of elements
echo ${#CONFIG[@]}         # 3

# Check if key exists
if [[ -v CONFIG[host] ]]; then
    echo "Host is configured"
fi
```

### **Iterating Over Associative Arrays:**

```bash
#!/bin/bash

declare -A GRADES=(
    [Alice]=95
    [Bob]=87
    [Charlie]=92
)

# Iterate over keys and values
for student in "${!GRADES[@]}"; do
    echo "$student scored ${GRADES[$student]}"
done

# Output:
# Alice scored 95
# Bob scored 87
# Charlie scored 92
```

---

## ✂️ Array Slicing

```bash
#!/bin/bash

NUMBERS=(0 1 2 3 4 5 6 7 8 9)

# Slice: ${array[@]:start:length}
echo ${NUMBERS[@]:2:3}     # 2 3 4 (start at index 2, take 3 elements)
echo ${NUMBERS[@]:5}       # 5 6 7 8 9 (from index 5 to end)
echo ${NUMBERS[@]: -3}     # 7 8 9 (last 3 elements)

# Copy array
COPY=("${NUMBERS[@]}")
echo ${COPY[@]}            # 0 1 2 3 4 5 6 7 8 9
```

---

## 🔍 Searching and Filtering

```bash
#!/bin/bash

FRUITS=("apple" "banana" "cherry" "apricot" "blueberry")

# Check if element exists
if [[ " ${FRUITS[@]} " =~ " apple " ]]; then
    echo "Apple found!"
fi

# Find elements starting with 'a'
for fruit in "${FRUITS[@]}"; do
    if [[ $fruit == a* ]]; then
        echo "Found: $fruit"
    fi
done

# Filter to new array
A_FRUITS=()
for fruit in "${FRUITS[@]}"; do
    if [[ $fruit == a* ]]; then
        A_FRUITS+=("$fruit")
    fi
done
echo "A fruits: ${A_FRUITS[@]}"    # apple apricot
```

---

## 🔢 Sorting Arrays

```bash
#!/bin/bash

NUMBERS=(5 2 8 1 9 3)

# Sort (creates new sorted array)
IFS=$'\n' SORTED=($(sort -n <<<"${NUMBERS[*]}"))
unset IFS

echo "Original: ${NUMBERS[@]}"    # 5 2 8 1 9 3
echo "Sorted: ${SORTED[@]}"       # 1 2 3 5 8 9

# Sort strings alphabetically
NAMES=("Charlie" "Alice" "Bob")
IFS=$'\n' SORTED_NAMES=($(sort <<<"${NAMES[*]}"))
unset IFS

echo ${SORTED_NAMES[@]}           # Alice Bob Charlie

# Reverse sort
IFS=$'\n' REVERSE=($(sort -r <<<"${NAMES[*]}"))
unset IFS
echo ${REVERSE[@]}                # Charlie Bob Alice
```

---

## 🎯 Practical Examples

### **Example 1: Server Monitor**

```bash
#!/bin/bash

# Array of servers
SERVERS=(
    "web1.example.com"
    "web2.example.com"
    "db1.example.com"
    "cache1.example.com"
)

echo "Checking ${#SERVERS[@]} servers..."

for server in "${SERVERS[@]}"; do
    if ping -c 1 -W 1 "$server" &>/dev/null; then
        echo "✅ $server is UP"
    else
        echo "❌ $server is DOWN"
    fi
done
```

### **Example 2: Configuration with Associative Array**

```bash
#!/bin/bash

# Configuration as associative array
declare -A CONFIG=(
    [app_name]="MyApp"
    [version]="1.0.0"
    [port]=8080
    [host]="0.0.0.0"
    [debug]=true
    [max_connections]=100
)

# Display configuration
echo "Application Configuration"
echo "========================="
for key in "${!CONFIG[@]}"; do
    printf "%-20s : %s\n" "$key" "${CONFIG[$key]}"
done
```

### **Example 3: Process Multiple Files**

```bash
#!/bin/bash

# Get all .txt files
FILES=(*.txt)

if [ ${#FILES[@]} -eq 0 ]; then
    echo "No .txt files found"
    exit 1
fi

echo "Found ${#FILES[@]} text files"

for file in "${FILES[@]}"; do
    echo "Processing: $file"
    
    # Count lines
    lines=$(wc -l < "$file")
    echo "  Lines: $lines"
    
    # Create backup
    cp "$file" "${file}.backup"
    echo "  Backed up to: ${file}.backup"
done
```

### **Example 4: User Database**

```bash
#!/bin/bash

# Array of associative arrays (simulate database)
declare -a USERS

# Add users
declare -A user1=(
    [name]="Alice"
    [email]="alice@example.com"
    [role]="admin"
)

declare -A user2=(
    [name]="Bob"
    [email]="bob@example.com"
    [role]="user"
)

# Alternative: Use indexed array with formatted strings
USERS=(
    "Alice|alice@example.com|admin"
    "Bob|bob@example.com|user"
    "Charlie|charlie@example.com|user"
)

# Query users
echo "User List:"
echo "=========="

for user_data in "${USERS[@]}"; do
    IFS='|' read -r name email role <<< "$user_data"
    printf "Name: %-10s Email: %-25s Role: %s\n" "$name" "$email" "$role"
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you create an array?**
   <details>
   <summary>Answer</summary>
   ARRAY=("item1" "item2" "item3") or ARRAY[0]="item1"
   </details>

2. **How do you get all array elements?**
   <details>
   <summary>Answer</summary>
   ${ARRAY[@]} or ${ARRAY[*]}
   </details>

3. **How do you get the number of elements in an array?**
   <details>
   <summary>Answer</summary>
   ${#ARRAY[@]}
   </details>

4. **What's the difference between indexed and associative arrays?**
   <details>
   <summary>Answer</summary>
   Indexed arrays use numeric indices (0, 1, 2...).
   Associative arrays use string keys (like hash maps/dictionaries).
   </details>

5. **How do you iterate over an array?**
   <details>
   <summary>Answer</summary>
   for item in "${ARRAY[@]}"; do ... done
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Shopping List**

```bash
#!/bin/bash
# shopping_list.sh

ITEMS=()

while true; do
    echo ""
    echo "Shopping List (${#ITEMS[@]} items)"
    echo "===================="
    
    if [ ${#ITEMS[@]} -gt 0 ]; then
        for i in "${!ITEMS[@]}"; do
            echo "$((i+1)). ${ITEMS[$i]}"
        done
    else
        echo "(empty)"
    fi
    
    echo "===================="
    echo "1. Add item"
    echo "2. Remove item"
    echo "3. Clear list"
    echo "4. Exit"
    read -p "Choice: " choice
    
    case $choice in
        1)
            read -p "Item name: " item
            ITEMS+=("$item")
            echo "Added: $item"
            ;;
        2)
            read -p "Item number to remove: " num
            idx=$((num-1))
            if [ $idx -ge 0 ] && [ $idx -lt ${#ITEMS[@]} ]; then
                unset ITEMS[$idx]
                ITEMS=("${ITEMS[@]}")  # Reindex
                echo "Item removed"
            else
                echo "Invalid item number"
            fi
            ;;
        3)
            ITEMS=()
            echo "List cleared"
            ;;
        4)
            echo "Goodbye!"
            exit 0
            ;;
    esac
done
```

### **Exercise 2: Grade Calculator**

```bash
#!/bin/bash
# grade_calculator.sh

declare -A STUDENTS=(
    [Alice]=95
    [Bob]=87
    [Charlie]=92
    [David]=78
    [Eve]=88
)

echo "Grade Report"
echo "============"

TOTAL=0
COUNT=0

for student in "${!STUDENTS[@]}"; do
    grade=${STUDENTS[$student]}
    
    # Determine letter grade
    if [ $grade -ge 90 ]; then
        letter="A"
    elif [ $grade -ge 80 ]; then
        letter="B"
    elif [ $grade -ge 70 ]; then
        letter="C"
    else
        letter="F"
    fi
    
    printf "%-10s : %3d (%s)\n" "$student" "$grade" "$letter"
    
    TOTAL=$((TOTAL + grade))
    COUNT=$((COUNT + 1))
done

echo "============"
AVERAGE=$((TOTAL / COUNT))
echo "Class Average: $AVERAGE"
```

### **Exercise 3: Log File Analyzer**

```bash
#!/bin/bash
# log_analyzer.sh

# Create sample log
cat > app.log << 'EOF'
2024-01-15 10:00:00 INFO Application started
2024-01-15 10:01:23 ERROR Database connection failed
2024-01-15 10:02:45 WARN Low memory
2024-01-15 10:03:12 INFO User logged in
2024-01-15 10:04:33 ERROR File not found
2024-01-15 10:05:21 INFO Processing complete
EOF

# Count log levels
declare -A LOG_COUNTS=(
    [INFO]=0
    [WARN]=0
    [ERROR]=0
)

# Parse log file
while read -r line; do
    if [[ $line =~ INFO ]]; then
        ((LOG_COUNTS[INFO]++))
    elif [[ $line =~ WARN ]]; then
        ((LOG_COUNTS[WARN]++))
    elif [[ $line =~ ERROR ]]; then
        ((LOG_COUNTS[ERROR]++))
    fi
done < app.log

# Display results
echo "Log Analysis"
echo "============"
for level in "${!LOG_COUNTS[@]}"; do
    printf "%-6s : %d\n" "$level" "${LOG_COUNTS[$level]}"
done

# Clean up
rm app.log
```

---

## 📝 Key Takeaways

✅ **Indexed arrays: ARRAY=(item1 item2 item3)**  
✅ **Associative arrays: declare -A ARRAY=([key]=value)**  
✅ **Access elements: ${ARRAY[index]} or ${ARRAY[key]}**  
✅ **All elements: ${ARRAY[@]}**  
✅ **Array length: ${#ARRAY[@]}**  
✅ **Add elements: ARRAY+=(new_item)**  
✅ **Remove elements: unset ARRAY[index]**  
✅ **Iterate: for item in "${ARRAY[@]}"; do ... done**  

---

## 🚀 Next Steps

You now know how to work with collections of data!

**Next lesson:** [13 - String Manipulation](13-string-manipulation.md) - Advanced text processing

---

## 💡 Pro Tips

**Always quote array expansions:**
```bash
for item in "${ARRAY[@]}"; do    # ✅ Correct
for item in ${ARRAY[@]}; do      # ❌ Breaks with spaces
```

**Check if array is empty:**
```bash
if [ ${#ARRAY[@]} -eq 0 ]; then
    echo "Array is empty"
fi
```

**Merge arrays:**
```bash
COMBINED=("${ARRAY1[@]}" "${ARRAY2[@]}")
```

**Read file into array:**
```bash
mapfile -t LINES < file.txt
# or
readarray -t LINES < file.txt
```

Arrays power your data processing! 📊
