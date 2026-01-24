---
render_with_liquid: false
---
# 06 - Arithmetic & Brace Expansion

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Arithmetic operations in bash
- Different arithmetic syntaxes ($(( )), let, expr)
- Brace expansion for sequences and patterns
- Floating-point arithmetic with bc
- Practical math operations for scripts
- Generating file sequences and ranges

---

## 🔢 Arithmetic Expansion $(( ))

**Real-World Analogy:**
```
Regular Calculator:
Press 5, press +, press 3, press =
→ Get 8

Bash Arithmetic:
$(( 5 + 3 ))
→ Get 8

Same operations, different interface!
```

### **Basic Arithmetic:**

```bash
# Simple operations
echo $(( 5 + 3 ))          # 8
echo $(( 10 - 4 ))         # 6
echo $(( 6 * 7 ))          # 42
echo $(( 20 / 4 ))         # 5
echo $(( 17 % 5 ))         # 2 (remainder/modulo)

# With variables
NUM1=10
NUM2=3

echo $(( NUM1 + NUM2 ))    # 13
echo $(( NUM1 - NUM2 ))    # 7
echo $(( NUM1 * NUM2 ))    # 30
echo $(( NUM1 / NUM2 ))    # 3 (integer division!)
echo $(( NUM1 % NUM2 ))    # 1 (remainder)

# Store result
RESULT=$(( NUM1 + NUM2 ))
echo $RESULT               # 13
```

### **Assignment Operators:**

```bash
COUNT=10

# Increment
((COUNT++))                # COUNT becomes 11
((COUNT+=5))               # COUNT becomes 16
((COUNT*=2))               # COUNT becomes 32
((COUNT-=10))              # COUNT becomes 22
((COUNT/=2))               # COUNT becomes 11
((COUNT%=3))               # COUNT becomes 2

echo $COUNT                # 2

# Pre vs Post increment
COUNT=10
echo $((COUNT++))          # 10 (returns old value, then increments)
echo $COUNT                # 11

COUNT=10
echo $((++COUNT))          # 11 (increments, then returns new value)
echo $COUNT                # 11
```

### **Comparison Operations:**

```bash
NUM1=10
NUM2=5

# Comparisons (returns 1 for true, 0 for false)
echo $(( NUM1 > NUM2 ))    # 1 (true)
echo $(( NUM1 < NUM2 ))    # 0 (false)
echo $(( NUM1 >= 10 ))     # 1 (true)
echo $(( NUM1 <= 5 ))      # 0 (false)
echo $(( NUM1 == 10 ))     # 1 (true)
echo $(( NUM1 != 5 ))      # 1 (true)

# Logical operators
echo $(( NUM1 > 5 && NUM2 < 10 ))    # 1 (both true)
echo $(( NUM1 < 5 || NUM2 < 10 ))    # 1 (one true)
echo $(( ! (NUM1 < 5) ))             # 1 (not false = true)
```

### **Bitwise Operations:**

```bash
NUM=12  # Binary: 1100

echo $(( NUM << 1 ))       # 24 (left shift: 11000)
echo $(( NUM >> 1 ))       # 6  (right shift: 0110)
echo $(( NUM & 5 ))        # 4  (AND: 1100 & 0101 = 0100)
echo $(( NUM | 5 ))        # 13 (OR: 1100 | 0101 = 1101)
echo $(( NUM ^ 5 ))        # 9  (XOR: 1100 ^ 0101 = 1001)
echo $(( ~NUM ))           # -13 (NOT)
```

---

## 🎲 Other Arithmetic Methods

### **let Command:**

```bash
# let performs arithmetic and assignment
let "NUM = 5 + 3"
echo $NUM                  # 8

let "NUM += 10"
echo $NUM                  # 18

let "NUM *= 2"
echo $NUM                  # 36

# Multiple operations
let "A = 5" "B = 10" "C = A + B"
echo $C                    # 15

# In conditions
let "RESULT = 10 > 5"
echo $RESULT               # 1 (true)
```

### **expr Command (deprecated but still used):**

```bash
# expr evaluates expressions
expr 5 + 3                 # 8
expr 10 - 4                # 6
expr 6 \* 7                # 42 (note: * must be escaped!)
expr 20 / 4                # 5
expr 17 % 5                # 2

# With variables
NUM1=10
NUM2=3
expr $NUM1 + $NUM2         # 13

# Store result
RESULT=$(expr $NUM1 \* $NUM2)
echo $RESULT               # 30

# Note: expr is old-style, prefer $(( ))
```

---

## 💰 Floating-Point Arithmetic with bc

**Bash Only Does Integers!**
```bash
# This gives wrong result:
echo $(( 10 / 3 ))         # 3 (not 3.333...)

# Use bc for decimals:
echo "10 / 3" | bc         # 3 (still integer)
echo "scale=2; 10 / 3" | bc    # 3.33 ✅
```

### **bc Examples:**

```bash
# Basic operations
echo "5.5 + 3.2" | bc      # 8.7
echo "10.5 - 3.2" | bc     # 7.3
echo "4.5 * 2.1" | bc      # 9.45

# Set decimal precision
echo "scale=4; 10 / 3" | bc          # 3.3333
echo "scale=2; 22 / 7" | bc          # 3.14

# Advanced math
echo "sqrt(25)" | bc       # 5
echo "scale=2; sqrt(2)" | bc         # 1.41
echo "2^10" | bc           # 1024 (power)

# With variables
PRICE=19.99
QUANTITY=3
TOTAL=$(echo "scale=2; $PRICE * $QUANTITY" | bc)
echo "Total: \$$TOTAL"     # Total: $59.97

# Here document style
RESULT=$(bc << EOF
scale=2
price = 19.99
quantity = 3
tax_rate = 0.08
subtotal = price * quantity
tax = subtotal * tax_rate
total = subtotal + tax
total
EOF
)
echo "Final total: \$$RESULT"
```

---

## 🌟 Brace Expansion

**Real-World Analogy:**
```
Manual way:
Create: file1.txt, file2.txt, file3.txt, file4.txt, file5.txt
→ Type 5 times

Brace expansion:
file{1..5}.txt
→ Bash expands to all 5 files!

Template expansion, automatic!
```

### **Numeric Sequences:**

```bash
# Simple sequence
echo {1..5}                # 1 2 3 4 5
echo {0..10}               # 0 1 2 3 4 5 6 7 8 9 10
echo {10..1}               # 10 9 8 7 6 5 4 3 2 1

# With step (bash 4.0+)
echo {0..20..2}            # 0 2 4 6 8 10 12 14 16 18 20
echo {0..100..10}          # 0 10 20 30 40 50 60 70 80 90 100
echo {10..0..2}            # 10 8 6 4 2 0

# Zero-padded
echo {01..10}              # 01 02 03 04 05 06 07 08 09 10
echo {001..005}            # 001 002 003 004 005

# Practical: Create files
touch file{1..5}.txt       # Creates file1.txt, file2.txt, ..., file5.txt
```

### **Character Sequences:**

```bash
# Letters
echo {a..z}                # a b c d ... x y z
echo {A..Z}                # A B C D ... X Y Z
echo {z..a}                # z y x ... c b a

# Subrange
echo {a..e}                # a b c d e
echo {A..E}                # A B C D E

# Practical: Create directories
mkdir dir_{A..D}           # Creates dir_A, dir_B, dir_C, dir_D
```

### **String Lists:**

```bash
# List of strings
echo {red,green,blue}      # red green blue
echo {cat,dog,bird}        # cat dog bird

# With prefix/suffix
echo file_{a,b,c}.txt      # file_a.txt file_b.txt file_c.txt
echo {test,prod,dev}_server    # test_server prod_server dev_server

# Practical: Backup files
cp file.txt{,.backup}      # Copies file.txt to file.txt.backup
mv config.yaml{.old,}      # Renames config.yaml.old to config.yaml
```

### **Nested Brace Expansion:**

```bash
# Nested sequences
echo {A..C}{1..3}          # A1 A2 A3 B1 B2 B3 C1 C2 C3

# Multiple braces
echo {a,b}_{1,2}           # a_1 a_2 b_1 b_2

# Complex nesting
echo {{A..C},{1..3}}       # A B C 1 2 3

# Practical: Create directory structure
mkdir -p project/{src,bin,docs}/{main,test}
# Creates:
# project/src/main
# project/src/test
# project/bin/main
# project/bin/test
# project/docs/main
# project/docs/test
```

### **Combining with Commands:**

```bash
# Create dated backups
cp database.sql database_$(date +%Y%m%d).sql.bak

# Create multiple backups
cp important.txt{,.bak1,.bak2,.bak3}
# Creates: important.txt.bak1, important.txt.bak2, important.txt.bak3

# Batch renaming (careful!)
mv file.{old,new}          # Renames file.old to file.new

# Create test files
touch test_{001..100}.txt  # 100 test files

# Year range
echo report_{2020..2026}.pdf    # All yearly reports
```

---

## 🎯 Practical Examples

### **Example 1: Simple Calculator**

```bash
#!/bin/bash

echo "=== Simple Calculator ==="
read -p "Enter first number: " num1
read -p "Enter second number: " num2

echo ""
echo "Addition:       $(( num1 + num2 ))"
echo "Subtraction:    $(( num1 - num2 ))"
echo "Multiplication: $(( num1 * num2 ))"
echo "Division:       $(( num1 / num2 ))"
echo "Remainder:      $(( num1 % num2 ))"
echo ""
echo "With Decimals:"
echo "Division:       $(echo "scale=2; $num1 / $num2" | bc)"
```

### **Example 2: Progress Counter**

```bash
#!/bin/bash

TOTAL=50
COUNT=0

while [[ $COUNT -lt $TOTAL ]]; do
    ((COUNT++))
    PERCENT=$(( COUNT * 100 / TOTAL ))
    
    echo -ne "Progress: $COUNT/$TOTAL ($PERCENT%)\r"
    sleep 0.1
done

echo ""
echo "Complete!"
```

### **Example 3: Disk Usage Percentage**

```bash
#!/bin/bash

# Get disk usage
USED=$(df / | tail -1 | awk '{print $3}')
TOTAL=$(df / | tail -1 | awk '{print $2}')

# Calculate percentage
PERCENT=$(( USED * 100 / TOTAL ))

echo "Disk Usage: $PERCENT%"

if [[ $PERCENT -gt 80 ]]; then
    echo "⚠️  Warning: Disk usage high!"
elif [[ $PERCENT -gt 90 ]]; then
    echo "🔴 Critical: Disk almost full!"
fi
```

### **Example 4: Bulk File Creation**

```bash
#!/bin/bash

# Create project structure
echo "Creating project structure..."

# Create directories
mkdir -p project/{src,tests,docs,config}/{main,backup}

# Create files
touch project/src/main/app_{1..5}.sh
touch project/tests/test_{001..010}.sh
touch project/docs/readme_{intro,usage,api}.md

echo "✅ Project structure created!"
tree project
```

### **Example 5: Date Range Generator**

```bash
#!/bin/bash

# Generate backup file names for each month
YEAR=2026

for MONTH in {01..12}; do
    FILENAME="backup_${YEAR}_${MONTH}.tar.gz"
    echo "Creating: $FILENAME"
    # touch "$FILENAME"  # Uncomment to actually create
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does $(( 10 / 3 )) return?**
   <details>
   <summary>Answer</summary>
   3 (integer division, decimals truncated)
   </details>

2. **How do you get 3.33 instead?**
   <details>
   <summary>Answer</summary>
   echo "scale=2; 10 / 3" | bc
   </details>

3. **What does {1..5} expand to?**
   <details>
   <summary>Answer</summary>
   1 2 3 4 5
   </details>

4. **What does ((COUNT++)) do?**
   <details>
   <summary>Answer</summary>
   Increments COUNT by 1
   </details>

5. **How do you create file{1..3}.txt?**
   <details>
   <summary>Answer</summary>
   touch file{1..3}.txt (creates file1.txt, file2.txt, file3.txt)
   </details>

---

## 🏋️ Hands-On Exercise

### **Exercise 1: Basic Math**

```bash
#!/bin/bash

A=15
B=4

echo "A = $A, B = $B"
echo ""
echo "Addition: $(( A + B ))"
echo "Subtraction: $(( A - B ))"
echo "Multiplication: $(( A * B ))"
echo "Division: $(( A / B ))"
echo "Modulo: $(( A % B ))"
echo ""
echo "A squared: $(( A * A ))"
echo "A cubed: $(( A * A * A ))"
```

### **Exercise 2: Temperature Converter**

```bash
#!/bin/bash

read -p "Enter temperature in Celsius: " celsius

# Fahrenheit = (Celsius * 9/5) + 32
fahrenheit=$(echo "scale=2; ($celsius * 9 / 5) + 32" | bc)

echo "${celsius}°C = ${fahrenheit}°F"
```

### **Exercise 3: Loop with Arithmetic**

```bash
#!/bin/bash

SUM=0

for NUM in {1..100}; do
    ((SUM += NUM))
done

echo "Sum of 1 to 100: $SUM"
# Should be 5050
```

### **Exercise 4: Create Backup Structure**

```bash
#!/bin/bash

# Create backup directories for each day of the week
mkdir -p backups/{monday,tuesday,wednesday,thursday,friday}

# Create hourly backup files
for HOUR in {00..23}; do
    touch "backups/monday/backup_${HOUR}00.tar.gz"
done

echo "Backup structure created!"
ls -R backups/
```

### **Exercise 5: Financial Calculator**

```bash
#!/bin/bash

read -p "Enter principal amount: " principal
read -p "Enter annual interest rate (e.g., 5 for 5%): " rate
read -p "Enter time in years: " years

# Simple interest: (P * R * T) / 100
interest=$(echo "scale=2; ($principal * $rate * $years) / 100" | bc)
total=$(echo "scale=2; $principal + $interest" | bc)

echo ""
echo "Principal: \$$principal"
echo "Interest:  \$$interest"
echo "Total:     \$$total"
```

---

## 📝 Key Takeaways

✅ **$(( ))** for integer arithmetic  
✅ **bc** for floating-point math  
✅ **((var++))** to increment  
✅ **{1..10}** for sequences  
✅ **{a,b,c}** for lists  
✅ **Bash only does integers** natively  
✅ **Brace expansion** saves typing  
✅ **Zero-padded** with {01..10}  
✅ **Nested braces** for complex patterns  
✅ **Use scale=N** in bc for decimals  

---

## 🚀 Next Steps

You can now perform arithmetic and generate sequences!

**Next lesson:** [07-command-substitution.md](07-command-substitution.md) - Capturing command output

---

## 💡 Pro Tips

**Increment Shortcuts:**
```bash
# These are equivalent:
COUNT=$((COUNT + 1))
((COUNT++))
((COUNT+=1))
let COUNT++
let COUNT+=1

# Prefer ((COUNT++)) - clearest and fastest
```

**Floating-Point Tricks:**
```bash
# Round to nearest integer
echo "scale=0; (10.7 + 0.5) / 1" | bc     # 11

# Calculate percentage
PERCENT=$(echo "scale=2; $PART * 100 / $TOTAL" | bc)

# Format currency
printf "\$%.2f\n" $(echo "19.99 * 3" | bc)
```

**Brace Expansion Tricks:**
```bash
# Quick backup
cp important.conf{,.backup}

# Restore backup
mv config.yaml{.backup,}

# Create dated backup
cp file.txt file_$(date +%Y%m%d).txt

# Mass file operations
rm test_{1..100}.tmp
```

**Common Mistakes:**
```bash
# ❌ Wrong: Spaces in $(( ))
echo $((5 + 3))            # ✅ Works
echo $(( 5 + 3 ))          # ✅ Also works (recommended)

# ❌ Wrong: Forgot bc for decimals
echo $(( 10 / 3 ))         # 3 (wrong!)
echo "10 / 3" | bc         # 3.33 ✅

# ❌ Wrong: No quotes in bc
VAR=$(echo scale=2; 10/3 | bc)        # ❌ Error!
VAR=$(echo "scale=2; 10/3" | bc)      # ✅ Correct
```

Math and sequences are now in your toolkit! 🔢
