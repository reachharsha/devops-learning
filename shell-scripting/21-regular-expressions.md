---
render_with_liquid: false
---
# 21 - Regular Expressions Mastery

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Basic, Extended, and Perl-Compatible Regular Expressions
- Regex syntax and special characters
- Character classes and quantifiers
- Groups and backreferences
- Lookahead and lookbehind
- Using regex in bash, grep, sed, and awk

---

## 🎨 What are Regular Expressions?

**Regular expressions (regex) are patterns used to match text.**

**Three main types:**
1. **BRE** - Basic Regular Expressions (grep, sed)
2. **ERE** - Extended Regular Expressions (grep -E, awk)
3. **PCRE** - Perl-Compatible Regular Expressions (grep -P, if available)

---

## 🔤 Basic Regex Syntax

### **Literals and Metacharacters:**

```bash
#!/bin/bash

# Literal characters (match themselves)
echo "cat" | grep "cat"              # Match: cat

# Metacharacters (special meaning)
# . ^ $ * + ? { } [ ] \ | ( )

# . - Any single character
echo "cat" | grep "c.t"              # Match: cat, cot, cut
echo "cat" | grep "c..t"             # No match (need 4 chars)

# ^ - Start of line
echo "cat" | grep "^cat"             # Match: cat at start
echo "the cat" | grep "^cat"         # No match: cat not at start

# $ - End of line
echo "cat" | grep "cat$"             # Match: cat at end
echo "cat is" | grep "cat$"          # No match: cat not at end

# * - Zero or more
echo "ct" | grep "ca*t"              # Match: ct, cat, caat
echo "caaaat" | grep "ca*t"          # Match

# Escape metacharacters with \
echo "2.5" | grep "2\.5"             # Match: 2.5 (literal dot)
```

---

## 📋 Character Classes

### **Predefined Character Classes:**

```bash
#!/bin/bash

# [abc] - Any one of a, b, or c
echo "cat" | grep "[cb]at"           # Match: cat or bat

# [^abc] - NOT a, b, or c
echo "cat" | grep "[^cb]at"          # No match

# [a-z] - Range
echo "cat" | grep "[a-z]"            # Match: any lowercase
echo "CAT" | grep "[A-Z]"            # Match: any uppercase
echo "123" | grep "[0-9]"            # Match: any digit

# Combine ranges
echo "Cat5" | grep "[a-zA-Z0-9]"     # Match: alphanumeric

# POSIX character classes
echo "cat123" | grep "[[:alpha:]]"   # Letters
echo "cat123" | grep "[[:digit:]]"   # Digits
echo "cat123" | grep "[[:alnum:]]"   # Letters + digits
echo "Hello!" | grep "[[:punct:]]"   # Punctuation
echo "a b c" | grep "[[:space:]]"    # Whitespace
```

### **Common POSIX Classes:**

| Class | Meaning | Equivalent |
|-------|---------|------------|
| `[:alpha:]` | Letters | `[a-zA-Z]` |
| `[:digit:]` | Digits | `[0-9]` |
| `[:alnum:]` | Alphanumeric | `[a-zA-Z0-9]` |
| `[:lower:]` | Lowercase | `[a-z]` |
| `[:upper:]` | Uppercase | `[A-Z]` |
| `[:space:]` | Whitespace | `[ \t\n\r\f\v]` |
| `[:punct:]` | Punctuation | `[!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~]` |
| `[:xdigit:]` | Hex digits | `[0-9A-Fa-f]` |

---

## 🔢 Quantifiers

### **Basic Quantifiers:**

```bash
#!/bin/bash

# * - Zero or more
grep "ca*t" file.txt                 # ct, cat, caat, caaat

# + - One or more (ERE)
grep -E "ca+t" file.txt              # cat, caat, caaat (not ct)

# ? - Zero or one (ERE)
grep -E "colou?r" file.txt           # color, colour

# {n} - Exactly n times
grep -E "a{3}" file.txt              # aaa

# {n,} - n or more times
grep -E "a{3,}" file.txt             # aaa, aaaa, aaaaa

# {n,m} - Between n and m times
grep -E "a{2,4}" file.txt            # aa, aaa, aaaa

# Examples
echo "abc" | grep -E "ab*c"          # Match: ac, abc, abbc
echo "abbbc" | grep -E "ab+c"        # Match: abc, abbc, abbbc
echo "ac" | grep -E "ab+c"           # No match (need at least one b)
echo "abc" | grep -E "ab?c"          # Match: ac, abc
echo "abbc" | grep -E "ab?c"         # No match (too many b's)
```

---

## 🎯 Groups and Alternation

### **Grouping with Parentheses:**

```bash
#!/bin/bash

# | - Alternation (OR)
grep -E "cat|dog" file.txt           # Match: cat OR dog

# () - Grouping
grep -E "(cat|dog)s" file.txt        # Match: cats OR dogs

# Quantifiers on groups
grep -E "(ab)+" file.txt             # Match: ab, abab, ababab

# Multiple groups
grep -E "(red|blue) (car|truck)" file.txt
# Match: red car, red truck, blue car, blue truck
```

### **Backreferences:**

```bash
#!/bin/bash

# \1, \2, etc. refer to captured groups

# Match repeated words
echo "hello hello" | grep -E "(\w+) \1"
# Match: hello hello

# Swap numbers
echo "123-456" | sed -E 's/([0-9]+)-([0-9]+)/\2-\1/'
# Output: 456-123

# Find duplicate lines
grep -E "^(.+)$.*^\1$" file.txt

# Match HTML tags
echo "<b>text</b>" | grep -E "<([a-z]+)>.*</\1>"
# Match: <b>text</b>
```

---

## 🔍 Anchors and Boundaries

### **Position Anchors:**

```bash
#!/bin/bash

# ^ - Start of line
grep "^Error" log.txt                # Lines starting with Error

# $ - End of line
grep "failed$" log.txt               # Lines ending with failed

# ^$ - Empty line
grep "^$" file.txt                   # Match empty lines

# \b - Word boundary (ERE)
grep -E "\bcat\b" file.txt           # Match: cat (not category)
echo "category" | grep -E "\bcat\b"  # No match

# \B - Not a word boundary
grep -E "\Bcat\B" file.txt           # Match: scatter (not cat)

# Examples
grep "^#" file.txt                   # Comment lines
grep "^[[:space:]]*$" file.txt       # Empty or whitespace-only lines
grep -E "^[0-9]+$" file.txt          # Lines with only numbers
```

---

## 🎨 Regex in Different Tools

### **In bash with =~:**

```bash
#!/bin/bash

# Bash regex matching
EMAIL="user@example.com"

if [[ $EMAIL =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]; then
    echo "Valid email"
fi

# Extract with BASH_REMATCH
URL="https://example.com:8080/path"

if [[ $URL =~ ^(https?)://([^:/]+)(:([0-9]+))?(/.*)?$ ]]; then
    PROTOCOL="${BASH_REMATCH[1]}"
    DOMAIN="${BASH_REMATCH[2]}"
    PORT="${BASH_REMATCH[4]}"
    PATH="${BASH_REMATCH[5]}"
    
    echo "Protocol: $PROTOCOL"
    echo "Domain: $DOMAIN"
    echo "Port: $PORT"
    echo "Path: $PATH"
fi
```

### **In grep:**

```bash
#!/bin/bash

# Basic regex (default)
grep "pattern" file.txt

# Extended regex
grep -E "pattern+" file.txt
grep -E "(cat|dog)" file.txt

# Perl regex (if available)
grep -P "\d{3}-\d{3}-\d{4}" file.txt    # Phone numbers
grep -P "(?<=@)\w+" emails.txt           # Domain names (lookbehind)

# Case-insensitive
grep -i "pattern" file.txt

# Invert match
grep -v "pattern" file.txt

# Count matches
grep -c "pattern" file.txt

# Show only matching part
grep -o "pattern" file.txt
```

### **In sed:**

```bash
#!/bin/bash

# Basic regex (default)
sed 's/old/new/' file.txt

# Extended regex
sed -E 's/(cat|dog)/animal/g' file.txt

# Backreferences
sed -E 's/([0-9]+)-([0-9]+)/\2-\1/' file.txt

# Multiple patterns
sed -E 's/cat/dog/; s/red/blue/' file.txt

# Delete lines matching pattern
sed '/pattern/d' file.txt

# Print only matching lines
sed -n '/pattern/p' file.txt
```

### **In awk:**

```bash
#!/bin/bash

# Pattern matching
awk '/pattern/ {print}' file.txt

# Regex in conditionals
awk '$1 ~ /^[0-9]+$/ {print}' file.txt

# Negation
awk '$1 !~ /pattern/ {print}' file.txt

# Case-insensitive
awk 'tolower($0) ~ /pattern/ {print}' file.txt

# Extract with match()
awk 'match($0, /[0-9]+/) {print substr($0, RSTART, RLENGTH)}' file.txt
```

---

## 🎯 Practical Regex Patterns

### **Common Validation Patterns:**

```bash
#!/bin/bash

# Email
EMAIL_REGEX="^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"

# Phone (xxx-xxx-xxxx)
PHONE_REGEX="^[0-9]{3}-[0-9]{3}-[0-9]{4}$"

# URL
URL_REGEX="^https?://[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(/.*)?$"

# IPv4 address
IP_REGEX="^([0-9]{1,3}\.){3}[0-9]{1,3}$"

# Date (YYYY-MM-DD)
DATE_REGEX="^[0-9]{4}-[0-9]{2}-[0-9]{2}$"

# Time (HH:MM:SS)
TIME_REGEX="^[0-9]{2}:[0-9]{2}:[0-9]{2}$"

# Hexadecimal color
COLOR_REGEX="^#[0-9A-Fa-f]{6}$"

# Username (alphanumeric, underscore, 3-16 chars)
USERNAME_REGEX="^[a-zA-Z0-9_]{3,16}$"

# Strong password (8+ chars, letter, number, special)
PASSWORD_REGEX="^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#$%]).{8,}$"

# Validation function
validate() {
    local value=$1
    local pattern=$2
    
    if [[ $value =~ $pattern ]]; then
        return 0
    else
        return 1
    fi
}

# Usage
if validate "user@example.com" "$EMAIL_REGEX"; then
    echo "Valid email"
fi
```

---

## 🔍 Advanced Patterns

### **Lookahead and Lookbehind (PCRE):**

```bash
#!/bin/bash

# Positive lookahead (?=pattern)
# Match X followed by Y, but don't include Y
grep -P "foo(?=bar)" file.txt        # Match: foo in "foobar"

# Negative lookahead (?!pattern)
# Match X NOT followed by Y
grep -P "foo(?!bar)" file.txt        # Match: foo not in "foobar"

# Positive lookbehind (?<=pattern)
# Match Y preceded by X, but don't include X
grep -P "(?<=foo)bar" file.txt       # Match: bar in "foobar"

# Negative lookbehind (?<!pattern)
# Match Y NOT preceded by X
grep -P "(?<!foo)bar" file.txt       # Match: bar not in "foobar"

# Extract domain from email
grep -oP "(?<=@)[a-zA-Z0-9.-]+" emails.txt

# Extract numbers after dollar sign
grep -oP "(?<=\$)[0-9.]+" prices.txt
```

### **Non-Greedy Matching:**

```bash
#!/bin/bash

# Greedy (default) - matches as much as possible
echo "<b>text</b> more <b>text</b>" | grep -oP "<b>.*</b>"
# Output: <b>text</b> more <b>text</b>

# Non-greedy - matches as little as possible
echo "<b>text</b> more <b>text</b>" | grep -oP "<b>.*?</b>"
# Output: <b>text</b>
```

---

## 🎯 Practical Examples

### **Example 1: Log Parser**

```bash
#!/bin/bash

LOGFILE="access.log"

# Extract IP addresses
grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" "$LOGFILE" | sort | uniq

# Extract URLs
grep -oE "GET [^ ]+" "$LOGFILE" | sed 's/GET //'

# Extract dates (YYYY-MM-DD)
grep -oE "[0-9]{4}-[0-9]{2}-[0-9]{2}" "$LOGFILE"

# Extract errors (lines with ERROR or CRITICAL)
grep -E "(ERROR|CRITICAL)" "$LOGFILE"

# Extract timestamps (HH:MM:SS)
grep -oE "[0-9]{2}:[0-9]{2}:[0-9]{2}" "$LOGFILE"
```

### **Example 2: Data Validator**

```bash
#!/bin/bash

validate_email() {
    [[ $1 =~ ^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$ ]]
}

validate_phone() {
    [[ $1 =~ ^[0-9]{3}-[0-9]{3}-[0-9]{4}$ ]]
}

validate_ip() {
    if [[ $1 =~ ^([0-9]{1,3}\.){3}[0-9]{1,3}$ ]]; then
        # Check each octet is 0-255
        IFS='.' read -ra OCTETS <<< "$1"
        for octet in "${OCTETS[@]}"; do
            if [ "$octet" -gt 255 ]; then
                return 1
            fi
        done
        return 0
    fi
    return 1
}

validate_date() {
    [[ $1 =~ ^[0-9]{4}-[0-9]{2}-[0-9]{2}$ ]]
}

# Test
validate_email "user@example.com" && echo "Valid email"
validate_phone "123-456-7890" && echo "Valid phone"
validate_ip "192.168.1.1" && echo "Valid IP"
validate_date "2024-01-15" && echo "Valid date"
```

### **Example 3: Text Sanitizer**

```bash
#!/bin/bash

# Remove HTML tags
sanitize_html() {
    echo "$1" | sed 's/<[^>]*>//g'
}

# Remove special characters
sanitize_alphanumeric() {
    echo "$1" | sed 's/[^a-zA-Z0-9]//g'
}

# Normalize whitespace
normalize_whitespace() {
    echo "$1" | sed 's/[[:space:]]+/ /g' | sed 's/^[[:space:]]*//; s/[[:space:]]*$//'
}

# Extract numbers only
extract_numbers() {
    echo "$1" | grep -oE "[0-9]+" | tr '\n' ' '
}

# Usage
HTML="<p>Hello <b>World</b>!</p>"
sanitize_html "$HTML"
# Output: Hello World!

TEXT="abc123!@#xyz789"
sanitize_alphanumeric "$TEXT"
# Output: abc123xyz789

MESSY="  too    much   space  "
normalize_whitespace "$MESSY"
# Output: too much space
```

### **Example 4: URL Parser**

```bash
#!/bin/bash

parse_url() {
    local url=$1
    
    # Pattern: protocol://domain:port/path?query#fragment
    local pattern="^(([^:]+)://)?([^:/]+)(:([0-9]+))?(/([^?#]*))?(\?([^#]*))?(#(.*))?$"
    
    if [[ $url =~ $pattern ]]; then
        echo "Protocol: ${BASH_REMATCH[2]}"
        echo "Domain: ${BASH_REMATCH[3]}"
        echo "Port: ${BASH_REMATCH[5]}"
        echo "Path: ${BASH_REMATCH[7]}"
        echo "Query: ${BASH_REMATCH[9]}"
        echo "Fragment: ${BASH_REMATCH[11]}"
    else
        echo "Invalid URL"
        return 1
    fi
}

# Test
parse_url "https://example.com:8080/path/to/page?id=123&name=test#section"
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does the ^ anchor match?**
   <details>
   <summary>Answer</summary>
   Start of line
   </details>

2. **What's the difference between * and +?**
   <details>
   <summary>Answer</summary>
   * = zero or more, + = one or more
   </details>

3. **How do you match a literal dot?**
   <details>
   <summary>Answer</summary>
   \. (escape with backslash)
   </details>

4. **What does [^0-9] match?**
   <details>
   <summary>Answer</summary>
   Any character that is NOT a digit
   </details>

5. **How do you use regex in bash conditionals?**
   <details>
   <summary>Answer</summary>
   [[ $var =~ pattern ]]
   </details>

---

## 📝 Key Takeaways

✅ **^ and $ anchor to start/end of line**  
✅ **. matches any single character**  
✅ **\* = zero or more, + = one or more, ? = zero or one**  
✅ **[abc] = character class, [^abc] = negated class**  
✅ **() for grouping, \1 for backreferences**  
✅ **Use grep -E for extended regex**  
✅ **[[ $var =~ pattern ]] for bash regex**  

---

## 🚀 Next Steps

You're now a regex master!

**Next lesson:** [22 - Networking Scripts](22-networking-scripts.md) - APIs, curl, wget, SSH automation

---

## 💡 Pro Tips

**Test regex online:**
- regex101.com
- regexr.com

**Common mistake:**
```bash
# ❌ Wrong
if [ $var =~ pattern ]; then

# ✅ Correct
if [[ $var =~ pattern ]]; then
```

**Verbose regex (readable):**
```bash
grep -P "(?x)  # Enable verbose mode
    ^          # Start of line
    [0-9]+     # One or more digits
    $          # End of line
" file.txt
```

Regex = text processing superpower! 🔥
