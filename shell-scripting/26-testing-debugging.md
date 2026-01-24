# 26 - Testing and Debugging

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Shell script testing frameworks (BATS, shunit2)
- Unit testing best practices
- Integration testing
- Debugging tools and techniques
- Static analysis with shellcheck
- Code coverage and quality metrics

---

## 🧪 Testing Frameworks

### **BATS (Bash Automated Testing System):**

**Installation:**
```bash
# macOS
brew install bats-core

# Linux
git clone https://github.com/bats-core/bats-core.git
cd bats-core
sudo ./install.sh /usr/local
```

**Basic BATS Test:**

```bash
#!/usr/bin/env bats

# test_script.bats

# Setup - runs before each test
setup() {
    TEST_DIR="$(mktemp -d)"
    export TEST_DIR
}

# Teardown - runs after each test
teardown() {
    rm -rf "$TEST_DIR"
}

# Test: Check if command exists
@test "script exists and is executable" {
    [ -x "./myscript.sh" ]
}

# Test: Script runs successfully
@test "script runs without errors" {
    run ./myscript.sh
    [ "$status" -eq 0 ]
}

# Test: Script output
@test "script outputs expected message" {
    run ./myscript.sh
    [ "$output" = "Hello, World!" ]
}

# Test: Script with arguments
@test "script accepts arguments" {
    run ./myscript.sh arg1 arg2
    [ "$status" -eq 0 ]
    [[ "$output" =~ "arg1" ]]
}

# Test: Error handling
@test "script fails with invalid input" {
    run ./myscript.sh --invalid
    [ "$status" -ne 0 ]
}
```

**Run BATS Tests:**
```bash
# Run all tests
bats test_script.bats

# Run with verbose output
bats -t test_script.bats

# Run specific test
bats -f "script outputs" test_script.bats
```

### **Advanced BATS Examples:**

```bash
#!/usr/bin/env bats

load test_helper  # Load helper functions

@test "function returns correct value" {
    source ./script.sh
    
    result=$(my_function "input")
    [ "$result" = "expected" ]
}

@test "file is created" {
    run ./script.sh create
    
    [ -f "$TEST_DIR/output.txt" ]
}

@test "output contains specific text" {
    run ./script.sh
    
    [[ "$output" =~ "Success" ]]
}

@test "script handles missing file" {
    run ./script.sh /nonexistent/file
    
    [ "$status" -eq 1 ]
    [[ "$output" =~ "not found" ]]
}

@test "environment variable is set" {
    export MY_VAR="test"
    run ./script.sh
    
    [ "$status" -eq 0 ]
}

@test "multiple assertions" {
    run ./script.sh
    
    [ "$status" -eq 0 ]
    [ "${lines[0]}" = "Line 1" ]
    [ "${lines[1]}" = "Line 2" ]
    [ "${#lines[@]}" -eq 2 ]
}
```

---

## 🔧 shunit2 Testing

### **Basic shunit2 Test:**

```bash
#!/bin/bash

# test_functions.sh

# Source the script to test
. ./functions.sh

# Test: Addition function
testAddition() {
    result=$(add 2 3)
    assertEquals "Addition failed" 5 "$result"
}

# Test: String function
testStringLength() {
    result=$(string_length "hello")
    assertEquals "String length wrong" 5 "$result"
}

# Test: File exists
testFileCreation() {
    create_file "test.txt"
    assertTrue "File not created" "[ -f test.txt ]"
    rm -f test.txt
}

# Test: Error handling
testInvalidInput() {
    result=$(validate_email "invalid")
    assertFalse "Validation should fail" "$?"
}

# Setup - runs before each test
setUp() {
    TEST_DIR=$(mktemp -d)
    cd "$TEST_DIR"
}

# Teardown - runs after each test
tearDown() {
    cd - >/dev/null
    rm -rf "$TEST_DIR"
}

# Load shunit2
. shunit2
```

---

## 🐛 Debugging Techniques

### **Debug Mode:**

```bash
#!/bin/bash

# Enable debugging
set -x          # Print commands as executed
set -v          # Print lines as read

# Your code here
echo "Hello"
NAME="John"
echo "Hello, $NAME"

# Disable debugging
set +x
set +v
```

### **Custom Debug Output:**

```bash
#!/bin/bash

DEBUG=${DEBUG:-false}

debug() {
    if $DEBUG; then
        echo "[DEBUG] $*" >&2
    fi
}

debug "Script started"
debug "Variable: $MY_VAR"

# Usage: DEBUG=true ./script.sh
```

### **Enhanced PS4 for Better Traces:**

```bash
#!/bin/bash

# Show file, line, and function in traces
export PS4='+(${BASH_SOURCE}:${LINENO}): ${FUNCNAME[0]:+${FUNCNAME[0]}(): }'

set -x

function my_function() {
    local var="value"
    echo "$var"
}

my_function
```

### **Trap for Debugging:**

```bash
#!/bin/bash

# Debug trap - execute after every command
trap 'echo "Line $LINENO: $BASH_COMMAND"' DEBUG

echo "First command"
VAR="value"
echo "Variable is $VAR"
```

### **bashdb Debugger:**

```bash
# Install bashdb
# Ubuntu: sudo apt-get install bashdb
# macOS: brew install bashdb

# Debug a script
bashdb ./script.sh

# Common bashdb commands:
# s - step (into function)
# n - next (over function)
# c - continue
# l - list code
# p variable - print variable
# b line - set breakpoint
# q - quit
```

---

## 🔍 Static Analysis with shellcheck

### **Install shellcheck:**

```bash
# macOS
brew install shellcheck

# Ubuntu/Debian
sudo apt-get install shellcheck

# Using Docker
docker run --rm -v "$PWD:/mnt" koalaman/shellcheck script.sh
```

### **Basic Usage:**

```bash
# Check a script
shellcheck script.sh

# Check with specific shell
shellcheck -s bash script.sh

# Exclude specific warnings
shellcheck -e SC2034,SC2086 script.sh

# Output in different formats
shellcheck -f json script.sh
shellcheck -f gcc script.sh
```

### **Common shellcheck Warnings:**

```bash
#!/bin/bash

# SC2086: Quote to prevent word splitting
FILES="file1.txt file2.txt"
rm $FILES           # ❌ Wrong
rm "$FILES"         # ✅ Correct

# SC2034: Variable appears unused
UNUSED_VAR="value"  # ❌ Warning

# SC2046: Quote to prevent globbing
for file in $(ls); do  # ❌ Wrong
    echo "$file"
done

for file in *; do      # ✅ Correct
    echo "$file"
done

# SC2164: Use cd ... || exit
cd /some/path       # ❌ Wrong
cd /some/path || exit  # ✅ Correct

# SC2181: Check exit code directly
command
if [ $? -eq 0 ]; then  # ❌ Wrong
    echo "Success"
fi

if command; then       # ✅ Correct
    echo "Success"
fi
```

### **Disable shellcheck Warnings:**

```bash
#!/bin/bash

# Disable specific warning for one line
# shellcheck disable=SC2086
rm $FILES

# Disable for entire file
# shellcheck disable=SC2086,SC2034

# Disable with explanation
# shellcheck disable=SC2086  # Files is a list, word splitting is intentional
rm $FILES
```

---

## 📊 Test Coverage

### **Track Test Coverage:**

```bash
#!/bin/bash

# test_with_coverage.sh

SCRIPT="./myscript.sh"
COVERAGE_DIR="coverage"

# Enable profiling
export BASH_ENV="$PWD/profile.sh"

# Profile script
cat > profile.sh << 'EOF'
shopt -s extdebug
trap 'echo "LINE:$LINENO"' DEBUG
EOF

# Run tests
./test_script.sh

# Analyze coverage
# (Manual analysis of which lines were executed)

# Clean up
rm -rf "$COVERAGE_DIR"
```

---

## 🧪 Integration Testing

### **Test Complete Workflow:**

```bash
#!/usr/bin/env bats

@test "complete deployment workflow" {
    # Setup
    export ENV="test"
    
    # Build
    run ./build.sh
    [ "$status" -eq 0 ]
    
    # Test
    run ./test.sh
    [ "$status" -eq 0 ]
    
    # Deploy
    run ./deploy.sh
    [ "$status" -eq 0 ]
    
    # Verify
    run curl -f http://localhost:8080/health
    [ "$status" -eq 0 ]
    
    # Cleanup
    run ./cleanup.sh
    [ "$status" -eq 0 ]
}
```

---

## 🎯 Practical Testing Examples

### **Example 1: Test a Backup Script**

```bash
#!/usr/bin/env bats

# test_backup.bats

setup() {
    export BACKUP_DIR="$(mktemp -d)"
    export SOURCE_DIR="$(mktemp -d)"
    
    # Create test files
    echo "data" > "$SOURCE_DIR/file1.txt"
    echo "data" > "$SOURCE_DIR/file2.txt"
}

teardown() {
    rm -rf "$BACKUP_DIR" "$SOURCE_DIR"
}

@test "backup creates archive" {
    run ./backup.sh "$SOURCE_DIR" "$BACKUP_DIR"
    
    [ "$status" -eq 0 ]
    [ -f "$BACKUP_DIR/backup_"*.tar.gz ]
}

@test "backup includes all files" {
    run ./backup.sh "$SOURCE_DIR" "$BACKUP_DIR"
    
    # Extract and verify
    ARCHIVE=$(ls "$BACKUP_DIR"/backup_*.tar.gz)
    tar -tzf "$ARCHIVE" | grep -q "file1.txt"
    tar -tzf "$ARCHIVE" | grep -q "file2.txt"
}

@test "backup fails with invalid source" {
    run ./backup.sh "/nonexistent" "$BACKUP_DIR"
    
    [ "$status" -ne 0 ]
}
```

### **Example 2: Mock External Commands**

```bash
#!/usr/bin/env bats

# Mock curl for testing
setup() {
    export PATH="$BATS_TEST_DIRNAME/mocks:$PATH"
    
    mkdir -p "$BATS_TEST_DIRNAME/mocks"
    
    # Create mock curl
    cat > "$BATS_TEST_DIRNAME/mocks/curl" << 'EOF'
#!/bin/bash
echo '{"status":"ok"}'
exit 0
EOF
    
    chmod +x "$BATS_TEST_DIRNAME/mocks/curl"
}

teardown() {
    rm -rf "$BATS_TEST_DIRNAME/mocks"
}

@test "script handles API response" {
    run ./api_client.sh
    
    [ "$status" -eq 0 ]
    [[ "$output" =~ "ok" ]]
}
```

### **Example 3: Test with Docker**

```bash
#!/usr/bin/env bats

@test "script works in Docker container" {
    # Build test image
    docker build -t test-image -f Dockerfile.test .
    
    # Run test in container
    run docker run --rm test-image ./myscript.sh
    
    [ "$status" -eq 0 ]
}

@test "script creates correct output in container" {
    run docker run --rm test-image ./myscript.sh
    
    [[ "$output" =~ "Expected output" ]]
}
```

---

## 🔧 Debugging Tools

### **Debugging Checklist:**

```bash
#!/bin/bash

debugging_checklist() {
    cat << 'EOF'
Debugging Checklist:
===================

1. Enable strict mode:
   set -euo pipefail

2. Add debugging output:
   set -x (or bash -x script.sh)

3. Check syntax:
   bash -n script.sh

4. Use shellcheck:
   shellcheck script.sh

5. Add debug logging:
   debug() { echo "[DEBUG] $*" >&2; }

6. Check exit codes:
   echo $?

7. Verify variable values:
   declare -p variable

8. Check function definitions:
   declare -F

9. Trace execution:
   export PS4='+(${BASH_SOURCE}:${LINENO}): '

10. Use bashdb for interactive debugging
EOF
}
```

### **Debug Helper Functions:**

```bash
#!/bin/bash

# Print variable with name
debug_var() {
    local var_name=$1
    local var_value="${!var_name}"
    echo "[DEBUG] $var_name = '$var_value'" >&2
}

# Print all variables
debug_all_vars() {
    echo "[DEBUG] All variables:" >&2
    declare -p | grep -v '^declare -[^ ]*r' >&2
}

# Print function names
debug_functions() {
    echo "[DEBUG] Defined functions:" >&2
    declare -F >&2
}

# Print stack trace
debug_stack() {
    echo "[DEBUG] Stack trace:" >&2
    local frame=0
    while caller $frame; do
        ((frame++))
    done | awk '{print "  " $0}' >&2
}

# Conditional breakpoint
debug_break() {
    local condition=$1
    shift
    local message=$*
    
    if eval "$condition"; then
        echo "[BREAK] $message" >&2
        echo "[BREAK] Condition: $condition" >&2
        debug_stack
    fi
}

# Usage
NAME="John"
debug_var NAME

COUNT=10
debug_break '[ $COUNT -gt 5 ]' "Count exceeded threshold"
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you run BATS tests?**
   <details>
   <summary>Answer</summary>
   bats test_script.bats
   </details>

2. **How do you enable debug mode in bash?**
   <details>
   <summary>Answer</summary>
   set -x or bash -x script.sh
   </details>

3. **What tool checks shell script syntax?**
   <details>
   <summary>Answer</summary>
   shellcheck script.sh
   </details>

4. **How do you check script syntax without running?**
   <details>
   <summary>Answer</summary>
   bash -n script.sh
   </details>

5. **What's the variable for last command's exit code?**
   <details>
   <summary>Answer</summary>
   $?
   </details>

---

## 📝 Key Takeaways

✅ **Use BATS or shunit2 for testing**  
✅ **Always run shellcheck**  
✅ **set -x for debugging**  
✅ **Write tests for critical functions**  
✅ **Test edge cases and error handling**  
✅ **Mock external dependencies**  
✅ **Use setup/teardown for test isolation**  
✅ **Check exit codes and output**  

---

## 🚀 Next Steps

Your scripts are now tested and debugged!

**Next lesson:** [27 - Real-World Projects](27-real-world-projects.md) - Complete practical projects

---

## 💡 Pro Tips

**Test-Driven Development:**
```bash
# 1. Write test first
@test "feature works" {
    run ./script.sh
    [ "$status" -eq 0 ]
}

# 2. Write code to pass test
# 3. Refactor
```

**Continuous Testing:**
```bash
# Watch for changes and re-run tests
while inotifywait -e modify script.sh; do
    bats test_script.bats
done
```

**Coverage goal:**
Aim for >80% coverage of critical functions

Test everything, trust nothing! 🧪
