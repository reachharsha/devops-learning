# 01 - What is Shell Scripting?

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What shell scripting is and why it's essential for DevOps
- The difference between shell and scripting languages
- Different shells and when to use each
- How scripts are executed
- The role of the shebang line

---

## 🐚 What is a Shell?

**Real-World Analogy:**
```
Shell = Translator between you and the OS

You:        "Please open my file"
Shell:      "Got it! Let me translate to computer language"
Computer:   *opens file*

It's like having a bilingual assistant who speaks:
- Human (your commands)
- Computer (system calls)
```

**Technical Definition:**
```
Shell = Command-line interpreter

Takes your commands → Interprets them → Executes them

Examples:
- bash (Bourne Again SHell) - Most common
- sh (Bourne Shell) - Original shell
- zsh (Z Shell) - Modern, feature-rich
- fish (Friendly Interactive SHell)
- ksh (Korn Shell)
```

---

## 📜 What is Shell Scripting?

**Simple Definition:**
```
Shell Script = List of shell commands in a file

Instead of typing commands one by one:
$ ls
$ cd /var/log
$ grep ERROR app.log
$ mail admin@example.com < errors.txt

Write them once in a file:
#!/bin/bash
ls
cd /var/log
grep ERROR app.log
mail admin@example.com < errors.txt

Run anytime: ./script.sh
```

**Professional Definition:**
```
Shell scripting = Writing programs using shell commands

Benefits:
✅ Automation (run tasks without manual intervention)
✅ Repeatability (same results every time)
✅ Scheduling (via cron jobs)
✅ Efficiency (save hours of manual work)
✅ Documentation (script IS the documentation)
```

---

## 🤔 Why Shell Scripting for DevOps?

### **DevOps Reality:**
```
A DevOps engineer's day:

08:00 - Deploy app to 50 servers       🔧 Script it!
10:00 - Backup 20 databases            🔧 Script it!
12:00 - Rotate log files               🔧 Script it!
14:00 - Monitor system health          🔧 Script it!
16:00 - Generate compliance reports    🔧 Script it!
18:00 - Cleanup old Docker images      🔧 Script it!

Shell scripts = Your automation army! 🚀
```

### **Why Shell Scripts vs Manual Commands:**

| Manual Commands | Shell Scripts |
|----------------|---------------|
| ❌ Type every time | ✅ Write once, run forever |
| ❌ Error-prone (typos) | ✅ Consistent execution |
| ❌ Hard to audit | ✅ Version controlled |
| ❌ Doesn't scale | ✅ Run on 1 or 1000 servers |
| ❌ Undocumented | ✅ Self-documenting |

### **Real-World Use Cases:**

```bash
✅ CI/CD Pipelines
   - Build code
   - Run tests
   - Deploy applications
   - Rollback on failure

✅ Infrastructure Management
   - Provision servers
   - Configure software
   - Apply security patches
   - Manage users/permissions

✅ Monitoring & Alerting
   - Check disk space
   - Monitor CPU/memory
   - Scan logs for errors
   - Send notifications

✅ Backup & Recovery
   - Automated backups
   - Database dumps
   - File synchronization
   - Disaster recovery

✅ Data Processing
   - ETL operations
   - Log parsing
   - Report generation
   - Data transformation
```

---

## 🆚 Shell Scripts vs Programming Languages

### **Shell (bash) vs Python vs Ruby:**

```python
# Task: Rename all .txt files to .bak

# Shell Script (2 lines)
#!/bin/bash
for file in *.txt; do mv "$file" "${file%.txt}.bak"; done

# Python (8 lines)
#!/usr/bin/env python3
import os
import glob

for file in glob.glob("*.txt"):
    base = os.path.splitext(file)[0]
    os.rename(file, f"{base}.bak")

# Ruby (6 lines)
#!/usr/bin/env ruby
Dir.glob("*.txt").each do |file|
  base = File.basename(file, ".txt")
  File.rename(file, "#{base}.bak")
end
```

### **When to Use Shell vs Other Languages:**

**Use Shell Scripts when:**
```
✅ Gluing together Linux commands
✅ System administration tasks
✅ Simple automation
✅ File/directory operations
✅ Text processing (grep, sed, awk)
✅ Quick prototypes
✅ Startup scripts
✅ CI/CD pipelines

Example: Deploying application, log rotation
```

**Use Python/Ruby/etc. when:**
```
✅ Complex logic needed
✅ Cross-platform compatibility
✅ Advanced data structures
✅ API integrations
✅ Machine learning
✅ Large-scale applications
✅ Need extensive libraries

Example: Web scraping, data analysis
```

**Why Shell Wins for DevOps:**
```
1. Native to Linux/Unix (no installation needed)
2. Direct access to all system commands
3. Fast for system tasks
4. Standard in production servers
5. Most DevOps tools have shell interfaces
6. Industry standard for automation
```

---

## 🐚 Different Shells Explained

### **Common Shells:**

```bash
# Bourne Shell (sh)
/bin/sh
- Original Unix shell (1977)
- POSIX compliant
- Basic features
- Still used for portability

# Bourne Again Shell (bash)
/bin/bash
- Most popular (default on most Linux)
- Rich features (arrays, functions, etc.)
- Excellent documentation
- Industry standard

# Z Shell (zsh)
/bin/zsh
- Modern features
- Better autocomplete
- Themes and plugins
- Default on macOS Catalina+

# Korn Shell (ksh)
/bin/ksh
- Commercial Unix systems
- Compatible with bash
- Fast execution

# Fish (Friendly Interactive SHell)
/usr/bin/fish
- User-friendly
- Syntax highlighting
- Not POSIX compatible
```

### **Which Shell to Use?**

```bash
For Scripts:
✅ bash - Best choice (universal, well-documented)
✅ sh - For maximum portability
❌ zsh - Not on all systems
❌ fish - Different syntax

For Interactive Use:
✅ zsh - Great features, productivity
✅ fish - Very user-friendly
✅ bash - Works everywhere

Our Course Focus: bash
Why? Most common, industry standard, best for DevOps
```

### **Check Your Shell:**

```bash
# Current shell
echo $SHELL
# Output: /bin/bash

# List available shells
cat /etc/shells
# Output:
# /bin/sh
# /bin/bash
# /bin/zsh
# /bin/fish

# Check bash version
bash --version
# Output: GNU bash, version 5.0.17

# Switch shell temporarily
bash          # Switch to bash
zsh           # Switch to zsh
exit          # Return to previous shell
```

---

## 🔧 How Scripts are Executed

### **Script Execution Flow:**

```
1. You type: ./script.sh

2. Kernel reads shebang: #!/bin/bash

3. Kernel executes: /bin/bash script.sh

4. Bash reads script line by line

5. Each command is executed

6. Script exits with status code
```

### **Ways to Run a Script:**

```bash
# Method 1: Direct execution (needs execute permission)
./script.sh

# Method 2: Explicit interpreter
bash script.sh
sh script.sh

# Method 3: Source/dot (runs in current shell)
source script.sh
. script.sh

# Method 4: Full path
/home/user/scripts/script.sh

# Method 5: Add to PATH
script.sh  # (if script is in PATH)
```

### **Execution Permissions:**

```bash
# View permissions
ls -l script.sh
# -rw-r--r--  1 user user  123 Jan 15 10:00 script.sh
#  ^^^ ^^^ ^^^
#  |   |   └─ Others: read only
#  |   └─ Group: read only
#  └─ Owner: read, write (no execute!)

# Add execute permission
chmod +x script.sh
# -rwxr-xr-x  1 user user  123 Jan 15 10:00 script.sh
#  ^^^
#  Now executable!

# Now can run
./script.sh
```

---

## #️⃣ The Shebang Line (#!/bin/bash)

### **What is the Shebang?**

```bash
#!/bin/bash
# ^^ Shebang (hash-bang)

Tells the system: "Use this interpreter to run this script"

#!         Hash-bang characters
/bin/bash  Path to bash interpreter

The kernel looks at first 2 bytes:
- If they're #! → reads rest of line for interpreter
- Executes: /bin/bash scriptname
```

### **Common Shebangs:**

```bash
#!/bin/bash
# Use bash (most common for scripts)

#!/bin/sh
# Use sh (portable, POSIX)

#!/usr/bin/env bash
# Find bash in PATH (more portable)
# Works even if bash is at /usr/local/bin/bash

#!/usr/bin/env python3
# Python script

#!/usr/bin/env node
# Node.js script

#!/bin/zsh
# Z shell script
```

### **Why #!/usr/bin/env bash is Better:**

```bash
# Problem with #!/bin/bash:
# - Assumes bash is at /bin/bash
# - Might be at /usr/local/bin/bash (macOS, custom installs)
# - Script fails if bash is elsewhere

# Solution: #!/usr/bin/env bash
# - env finds bash wherever it is in PATH
# - More portable
# - Recommended for modern scripts

#!/usr/bin/env bash  # ✅ Recommended
#!/bin/bash           # ✅ Also fine (more common)
```

### **What Happens Without Shebang?**

```bash
# script.sh (no shebang)
echo "Hello"

# Execute:
./script.sh
# Uses your current shell (might work, might not)

# Better to be explicit!
#!/bin/bash
echo "Hello"
```

---

## 🎯 Your First Look at a Script

```bash
#!/bin/bash
# File: hello.sh
# Description: My first shell script!
# Author: Your Name
# Date: 2024-01-15

# This is a comment (ignored by bash)

# Print message to screen
echo "Hello, DevOps World!"

# Print current directory
echo "Current directory: $(pwd)"

# Print current user
echo "Current user: $USER"

# Print date
echo "Today is: $(date +%Y-%m-%d)"

# Exit successfully
exit 0
```

**Breakdown:**
```
Line 1:  Shebang - tells system to use bash
Line 2-5: Comments - documentation
Line 9:  echo command - prints text
Line 12: Command substitution - $(pwd) runs pwd
Line 15: Environment variable - $USER
Line 18: Date formatting
Line 21: Exit with success code (0)
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is the purpose of the shebang line?**
   <details>
   <summary>Answer</summary>
   Tells the system which interpreter to use to execute the script (e.g., #!/bin/bash uses bash)
   </details>

2. **What's the difference between ./script.sh and bash script.sh?**
   <details>
   <summary>Answer</summary>
   ./script.sh requires execute permission and uses the shebang interpreter
   bash script.sh explicitly uses bash, no execute permission needed
   </details>

3. **Which shell is most common for DevOps scripting?**
   <details>
   <summary>Answer</summary>
   bash (Bourne Again Shell) - most widely available and documented
   </details>

4. **Why use shell scripts instead of Python for system tasks?**
   <details>
   <summary>Answer</summary>
   Native to Linux, direct access to system commands, no installation needed, faster for system operations
   </details>

5. **What command makes a script executable?**
   <details>
   <summary>Answer</summary>
   chmod +x script.sh
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Check your current shell
echo $SHELL

# 2. Check bash version
bash --version

# 3. List available shells
cat /etc/shells

# 4. Create your first script
nano hello.sh

# 5. Add this content:
#!/bin/bash
echo "My first script!"
echo "Running on: $(hostname)"
echo "As user: $USER"
date

# 6. Save and exit (Ctrl+O, Enter, Ctrl+X)

# 7. Try to run (should fail - no permission)
./hello.sh

# 8. Check permissions
ls -l hello.sh

# 9. Add execute permission
chmod +x hello.sh

# 10. Check permissions again
ls -l hello.sh

# 11. Run the script
./hello.sh

# 12. Alternative ways to run
bash hello.sh
sh hello.sh

# 13. See the difference
which bash
which sh
```

---

## 📝 Key Takeaways

✅ **Shell = Command interpreter** (bash, zsh, sh)  
✅ **Shell script = Sequence of commands in a file**  
✅ **Shebang (#!/bin/bash) = Tells system which interpreter to use**  
✅ **bash = Most common shell for scripting (DevOps standard)**  
✅ **chmod +x = Make script executable**  
✅ **Shell scripts perfect for system automation**  
✅ **Use shell for DevOps, Python for complex logic**  
✅ **#!/usr/bin/env bash = More portable shebang**  

---

## 🚀 Next Steps

You now understand what shell scripting is and why it matters!

**Next lesson:** [02 - Your First Script](02-your-first-script.md) - Create, run, and structure your first real script

---

## 💡 Pro Tips

**Choosing shell for scripts:**
```bash
# For maximum compatibility (works on any Unix)
#!/bin/sh

# For modern Linux (best features, documentation)
#!/bin/bash

# For portability (finds bash anywhere)
#!/usr/bin/env bash
```

**Check what a shebang does:**
```bash
# Script has #!/bin/bash
# To see what runs:
head -1 script.sh
# Output: #!/bin/bash

# Verify interpreter exists
which bash
# Output: /bin/bash
```

**Shell script file extensions:**
```bash
script.sh     # Common convention
script        # No extension (also common)
script.bash   # Explicit bash script

Extension doesn't matter to execution!
Shebang determines interpreter, not filename.
```

**Why DevOps loves shell scripts:**
```
1. Every Linux server has bash
2. No dependencies to install
3. Direct system access
4. Fast execution
5. Easy to version control
6. Industry standard for automation
7. Glue language for DevOps tools
```

Welcome to the world of shell scripting! 🚀
