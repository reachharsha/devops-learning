---
render_with_liquid: false
---
# 03 - Command Line Mastery

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Why the command line is essential for DevOps
- Command structure and syntax
- How to use command options and arguments
- Essential navigation and file listing commands
- Command history and shortcuts
- Getting help with man pages

---

## 🖥️ Why Command Line?

**Real-World Analogy:**
```
GUI (Graphical Interface) = Ordering at restaurant counter
- Point at menu items
- Visual, easy
- Slow for bulk orders
- Can't automate

CLI (Command Line) = Calling order by phone
- Speak exactly what you want
- Fast once you know it
- Can place 100 orders instantly
- Easily automated
```

### Why DevOps Uses CLI:

```
✅ **Speed**: Type faster than clicking
✅ **Automation**: Scripts run commands automatically
✅ **Remote Access**: SSH = command line only
✅ **Power**: More control, more options
✅ **Efficiency**: Chain commands together
✅ **Reproducible**: Same command = same result
✅ **Server Management**: Most servers have NO GUI
```

**Reality Check:**
```
Desktop Users:    Click buttons (GUI)
DevOps Engineers: Type commands (CLI) 90% of time

You MUST be comfortable with command line!
```

---

## 🏗️ Command Structure

### **Basic Anatomy:**
```
command  [options]  [arguments]
│        │          │
│        │          └─ What to act on
│        └─ How to behave
└─ What to do

Examples:
ls                    # Command only
ls -l                 # Command + option
ls -l /etc            # Command + option + argument
ls -la /home/user     # Command + multiple options + argument
```

### **Components Explained:**

**Command:**
```bash
The program/action you want to run

ls       # List files
cd       # Change directory
cat      # Display file contents
pwd      # Print working directory
```

**Options (Flags):**
```bash
Modify command behavior
Usually start with - or --

Short form (single dash):
-a    -l    -h    -v

Can combine:
-la   Same as: -l -a

Long form (double dash):
--all  --help  --version

Examples:
ls -a          # Show all (including hidden)
ls --all       # Same as above
ls -la         # Combine: long format + all
```

**Arguments:**
```bash
What the command acts on
Files, directories, text, etc.

Examples:
cat file.txt              # file.txt is argument
ls /etc                   # /etc is argument
cp source.txt dest.txt    # Two arguments
```

---

## 📋 Essential Commands Deep Dive

### **pwd (Print Working Directory)**

**Shows your current location**

```bash
# Basic usage
pwd

# Output examples:
/home/yourname
/var/log
/etc/nginx

# Always absolute path (starts with /)
```

**When to use:**
- Lost in directory structure
- Before running destructive commands
- In scripts to verify location

---

### **ls (List Directory Contents)**

**Shows files and directories**

**Basic usage:**
```bash
# List current directory
ls

# Output:
Desktop    Documents    Downloads    Pictures
```

**Common options:**
```bash
# Long format (detailed)
ls -l
# Shows: permissions, owner, size, date, name

# Show all files (including hidden)
ls -a
# Hidden files start with .
# Examples: .bashrc  .ssh  .config

# Human-readable sizes
ls -lh
# Instead of: 1234567 bytes
# Shows:      1.2M

# Sort by time (newest first)
ls -lt

# Reverse sort (oldest first)
ls -ltr

# Show directories only
ls -d */

# Recursive (show subdirectories)
ls -R

# Combinations (most useful)
ls -lah       # Long, all, human-readable
ls -lth       # Long, time-sorted, human-readable
ls -lthr      # Same + reverse (oldest first)
```

**Practical examples:**
```bash
# List files larger than 1MB
ls -lh | grep M

# Count files in directory
ls | wc -l

# List newest file
ls -lt | head -n 2

# List specific type
ls *.txt
ls *.log
ls *.sh

# List with full path
ls -d $PWD/*
```

---

### **cd (Change Directory)**

**Navigate to different directories**

```bash
# Go to specific directory (absolute path)
cd /var/log
cd /home/user/Documents

# Go to specific directory (relative path)
cd Documents
cd ../Downloads
cd ../../etc

# Go to home directory (3 ways)
cd
cd ~
cd $HOME

# Go to previous directory
cd -
# Example:
cd /var/log
cd /etc
cd -        # Back to /var/log

# Go to root directory
cd /

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to parent's sibling directory
cd ../other-directory
```

**Pro tips:**
```bash
# Auto-complete with Tab
cd Doc[Tab]        # Completes to: cd Documents/

# Directory stack
cd ~               # Start at home
pushd /var/log     # Go to /var/log (save home)
pushd /etc         # Go to /etc (save /var/log)
popd               # Back to /var/log
popd               # Back to home

# CDPATH (advanced)
export CDPATH=/var/log:/etc
cd nginx           # Looks in /etc/nginx first
```

---

### **clear**

**Clear terminal screen**

```bash
# Clear screen
clear

# Keyboard shortcut (faster)
Ctrl + L
```

---

### **man (Manual Pages)**

**Get help on commands**

```bash
# View manual for command
man ls
man cd
man grep

# Search man pages
man -k keyword
man -k network
man -k copy

# Specific section
man 5 passwd      # passwd file format
man 1 passwd      # passwd command

# Navigation in man pages:
Space        Next page
b            Previous page
/search      Search forward
?search      Search backward
n            Next search result
N            Previous search result
q            Quit
```

**Man page sections:**
```
1: User commands (ls, cd, cat)
2: System calls (programming)
3: Library functions (programming)
4: Device files (/dev)
5: File formats (/etc/passwd)
6: Games
7: Miscellaneous
8: System admin commands (reboot, ifconfig)
```

---

### **help and --help**

**Quick command help**

```bash
# Built-in command help
help cd
help pwd

# Command's help option
ls --help
grep --help
cat --help

# Show version
ls --version
grep --version
```

---

## 🔑 Command Line Shortcuts

### **Keyboard Shortcuts:**

**Navigation:**
```bash
Ctrl + A    Move to beginning of line
Ctrl + E    Move to end of line
Ctrl + →    Move forward one word
Ctrl + ←    Move backward one word
Alt + F     Move forward one word
Alt + B     Move backward one word
```

**Editing:**
```bash
Ctrl + K    Cut from cursor to end of line
Ctrl + U    Cut from cursor to beginning
Ctrl + W    Cut word before cursor
Ctrl + Y    Paste (yank) what was cut
Ctrl + L    Clear screen (like 'clear')
```

**History:**
```bash
↑           Previous command
↓           Next command
Ctrl + R    Search command history (then type)
Ctrl + G    Cancel search
!!          Run last command
!$          Last argument of previous command
!^          First argument of previous command
```

**Process Control:**
```bash
Ctrl + C    Cancel current command
Ctrl + Z    Suspend current command
Ctrl + D    Exit/logout (or EOF)
```

---

## 📜 Command History

**Every command you type is saved!**

```bash
# Show command history
history

# Output:
  1  ls
  2  cd Documents
  3  pwd
  4  cat file.txt
  ...

# Run command by number
!2              # Run command #2 (cd Documents)

# Run last command
!!

# Run last command with sudo
sudo !!

# Search history
Ctrl + R        # Then type to search
# Example: Press Ctrl+R, type "grep"

# Clear history
history -c

# Delete specific entry
history -d 123

# Execute previous command matching string
!ls             # Run last command starting with 'ls'
!?docker        # Run last command containing 'docker'
```

**History file:**
```bash
# View history file
cat ~/.bash_history

# Or
cat ~/.zsh_history   # If using zsh
```

---

## 🎨 Tab Completion

**Save typing with Tab!**

```bash
# Complete command
gre[Tab]        # Completes to: grep

# Complete file/directory
ls Doc[Tab]     # Completes to: ls Documents/

# Show options if multiple matches
cat f[Tab][Tab]  # Shows: file1.txt  file2.txt

# Complete with path
cd /var/lo[Tab]  # Completes to: cd /var/log/

# Complete command options
ls --[Tab][Tab]  # Shows all long options
```

---

## 🔧 Combining Commands

**Chain commands together:**

```bash
# Run sequentially (always run both)
command1 ; command2

# Run if previous succeeds (AND)
command1 && command2

# Run if previous fails (OR)
command1 || command2

# Examples:
mkdir test && cd test
# Create directory, if successful, enter it

cd /nonexistent || echo "Failed to change directory"
# Try to cd, if fails, print message

sudo apt update && sudo apt upgrade
# Update package list, if successful, upgrade
```

---

## 🎯 Practical Examples

### **Daily Navigation:**

```bash
# Start at home
cd ~
pwd
# /home/yourname

# List files with details
ls -lah

# Go to Documents
cd Documents

# What's here?
ls

# Go to system configs
cd /etc

# Find nginx config
ls | grep nginx

# Go to that directory
cd nginx

# View main config
ls -lh

# Go back home quickly
cd

# Where were we? Go back
cd -

# Check we're in nginx directory
pwd
```

### **Finding Information:**

```bash
# What does ls do?
man ls

# Quick help for grep
grep --help

# Search for network-related commands
man -k network

# What version of bash?
bash --version

# What version of Python?
python3 --version
```

### **Using History:**

```bash
# Run last command
!!

# Run last command with sudo
sudo !!

# Search for previous grep command
Ctrl + R
(then type): grep

# Use last argument
ls /var/log/nginx/error.log
cat !$
# Runs: cat /var/log/nginx/error.log
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between -a and --all?**
   <details>
   <summary>Answer</summary>
   Short form (-a) vs long form (--all). They do the same thing, long form is more readable.
   </details>

2. **How do you see the manual for the ls command?**
   <details>
   <summary>Answer</summary>
   man ls
   </details>

3. **What does !! do?**
   <details>
   <summary>Answer</summary>
   Runs the last command you typed
   </details>

4. **How do you clear the terminal screen?**
   <details>
   <summary>Answer</summary>
   clear command or Ctrl+L
   </details>

5. **What's the difference between ls -l -a and ls -la?**
   <details>
   <summary>Answer</summary>
   No difference! Both combine -l and -a options. -la is just shorter.
   </details>

---

## 🏋️ Hands-On Exercise

**Practice these commands:**

```bash
# 1. Where am I?
pwd

# 2. What's here?
ls -lah

# 3. Go to /tmp
cd /tmp

# 4. Confirm location
pwd

# 5. Create test file
touch test.txt

# 6. List to verify
ls -l test.txt

# 7. Go back home
cd

# 8. Now where?
pwd

# 9. View command history
history | tail -n 10

# 10. Clear screen
clear

# 11. Run previous pwd command
!!

# 12. Get help on cat command
man cat

# 13. Quick help for cp
cp --help

# 14. Go to /tmp again
cd /tmp

# 15. Go back to previous location (home)
cd -

# 16. Verify
pwd
```

---

## 📝 Key Takeaways

✅ Command line is **essential** for DevOps (90% of work)  
✅ Basic syntax: `command [options] [arguments]`  
✅ **pwd** - where am I?  
✅ **ls** - what's here?  
✅ **cd** - go somewhere  
✅ **man** - get help on any command  
✅ **Tab** completes commands and paths (use it!)  
✅ **↑ arrow** recalls previous commands  
✅ **Ctrl+R** searches command history  
✅ **!!** runs last command  
✅ **&&** chains commands (run if previous succeeds)  

---

## 🚀 Next Steps

You've mastered command line basics!

**Next lesson:** [04-file-operations.md](04-file-operations.md) - Creating, copying, moving, and deleting files

---

## 💡 Pro Tips

**Speed up your workflow:**
```bash
# Most used command combo
ls -lah         # List all with details

# Quick directory jump
cd -            # Toggle between two locations

# Run with sudo if forgot
sudo !!

# Auto-complete everything
Use Tab CONSTANTLY!

# Search history instead of retyping
Ctrl + R

# Learn these 4, you're 80% there:
ls -lah         # See everything
cd              # Navigate
pwd             # Know where you are
man command     # Get help
```

**Create muscle memory:**
```
Week 1: Force yourself to use CLI only
Week 2: Notice you're faster
Week 3: Can't imagine going back to GUI
Week 4: CLI master 🚀
```

The command line is your superpower! 💪
