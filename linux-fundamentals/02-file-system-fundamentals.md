---
render_with_liquid: false
---
# 02 - File System Fundamentals

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Linux directory structure and hierarchy
- The "everything is a file" philosophy
- Absolute vs relative paths
- How to navigate the file system
- Where important files and directories are located

---

## 🏗️ The Linux File System Hierarchy

**Real-World Analogy:**
```
File System = City Layout
/ (root)            = City Center (everything starts here)
/home              = Residential neighborhoods (user homes)
/etc               = City Hall (configuration/rules)
/var               = Warehouse district (variable data)
/bin               = Tool shop (essential commands)
/usr               = Shopping district (user programs)
/tmp               = Public park (temporary space)
```

### The Directory Tree:
```
/  (root - the top of everything)
│
├── bin/     Essential command binaries (ls, cp, mv)
├── boot/    Boot loader files (kernel, grub)
├── dev/     Device files (hard drives, USB, etc.)
├── etc/     System configuration files
├── home/    User home directories
│   ├── user1/
│   ├── user2/
│   └── yourname/
├── lib/     Shared libraries (like Windows DLLs)
├── media/   Mount points for removable media
├── mnt/     Temporary mount points
├── opt/     Optional software packages
├── proc/    Process and system information
├── root/    Root user's home directory
├── run/     Runtime data
├── sbin/    System binaries (admin commands)
├── srv/     Service data (web servers, FTP)
├── sys/     System and hardware information
├── tmp/     Temporary files (cleared on reboot)
├── usr/     User programs and data
│   ├── bin/     User commands
│   ├── lib/     Libraries
│   ├── local/   Locally installed software
│   └── share/   Shared data
└── var/     Variable data (logs, databases)
    ├── log/     Log files
    ├── mail/    Mail files
    └── tmp/     Temp files (not cleared)
```

---

## 📂 Important Directories Explained

### **/ (Root Directory)**
```
The very top of the file system
Everything starts here
NOT the same as /root (root user's home)

Windows equivalent: C:\
```

### **/home (User Home Directories)**
```
Each user gets their own directory here
/home/john  = John's home
/home/mary  = Mary's home

Your personal files, documents, configs go here
Like: C:\Users\YourName on Windows
```

**Your home shortcuts:**
```bash
~        Same as /home/yourname
~/       Your home directory
~/Desktop   Your desktop folder

Examples:
cd ~           # Go to your home
cd ~/Documents # Go to your documents
ls ~           # List your home directory
```

### **/etc (Configuration Files)**
```
System-wide configuration files
"et cetera" or "editable text configuration"

Important files:
/etc/passwd      User accounts
/etc/group       User groups
/etc/hostname    System name
/etc/hosts       DNS entries
/etc/ssh/        SSH configuration
/etc/nginx/      Nginx web server config
/etc/fstab       File system mount table
```

**Example:**
```bash
# View your system hostname
cat /etc/hostname

# View user accounts
cat /etc/passwd

# Check SSH config
cat /etc/ssh/sshd_config
```

### **/var (Variable Data)**
```
Data that changes frequently
Logs, databases, websites, email

Important subdirectories:
/var/log/        System logs
/var/www/        Web server files
/var/lib/        Application state data
/var/tmp/        Temp files (persist across reboots)
```

**Example:**
```bash
# View system log
sudo tail -f /var/log/syslog

# Check web server logs
sudo tail -f /var/log/nginx/access.log

# See all log files
ls -la /var/log/
```

### **/bin and /usr/bin (Binaries/Commands)**
```
/bin          Essential commands (ls, cp, mv, cat)
              Needed even in single-user mode

/usr/bin      User commands (most programs)
              Non-essential for basic system

/sbin         System admin commands (reboot, ifconfig)
/usr/sbin     Non-essential system commands
```

**Example:**
```bash
# Where is the ls command?
which ls
# Output: /usr/bin/ls

# Where is the cat command?
which cat
# Output: /usr/bin/cat
```

### **/tmp (Temporary Files)**
```
Temporary files
Cleared on reboot
Anyone can write here
```

**Example:**
```bash
# Create temp file
echo "test" > /tmp/mytest.txt

# Use /tmp for testing
cd /tmp
touch testfile
```

### **/dev (Devices)**
```
Device files (everything is a file!)
Hard drives, USB, terminals, null device

Important devices:
/dev/sda        First hard drive
/dev/sda1       First partition
/dev/sdb        Second hard drive
/dev/null       "Black hole" (discard output)
/dev/zero       Infinite zeros
/dev/random     Random data
```

**Example:**
```bash
# List block devices
lsblk

# Discard output (hide it)
command > /dev/null

# Generate random data
head -c 10 /dev/random | base64
```

### **/proc (Process Information)**
```
Virtual filesystem
Process and system info
Not real files on disk!
```

**Example:**
```bash
# CPU information
cat /proc/cpuinfo

# Memory information
cat /proc/meminfo

# System uptime
cat /proc/uptime

# Process info (PID 1)
ls -la /proc/1/
```

### **/root (Root User's Home)**
```
Home directory for root user
NOT the same as / (root directory)
Regular users can't access
```

---

## 🌳 The "Everything is a File" Philosophy

**In Linux, almost EVERYTHING is represented as a file:**

```
Regular file:     document.txt
Directory:        /home/user/  (a special file listing contents)
Hard drive:       /dev/sda     (file representing disk)
Terminal:         /dev/tty     (file for terminal)
Process info:     /proc/1234   (file for process 1234)
Network socket:   /var/run/socket
```

**Why this matters:**
```
Same commands work on everything!
cat, cp, mv work on devices, processes, etc.

Example:
cat document.txt       # Read file
cat /dev/sda           # Read disk (raw data)
cat /proc/cpuinfo      # Read CPU info
```

---

## 🗺️ Paths: Absolute vs Relative

### **Absolute Path (Full Address)**
```
Starts with /
Complete path from root
Works from anywhere

Examples:
/home/john/documents/report.txt
/etc/nginx/nginx.conf
/var/log/syslog
```

**Analogy:**
```
"123 Main Street, New York, NY 10001"
Complete address - works from anywhere
```

### **Relative Path (From Current Location)**
```
Doesn't start with /
Relative to current directory
Shorter, but context-dependent

Examples:
documents/report.txt    (from /home/john)
../other-user/file.txt  (go up one level, then down)
./script.sh             (in current directory)
```

**Analogy:**
```
"Next door"
"Down the street"
"Two blocks north"
Depends on where you are!
```

### **Special Path Symbols:**
```
.    = Current directory
..   = Parent directory (one level up)
~    = Your home directory
/    = Root directory
-    = Previous directory
```

**Examples:**
```bash
# Current directory
ls .           # List current directory (same as: ls)
./script.sh    # Run script in current directory

# Parent directory
ls ..          # List parent directory
cd ..          # Go up one level
cd ../..       # Go up two levels

# Home directory
cd ~           # Go to your home
ls ~/Documents # List your Documents folder

# Previous directory
cd /var/log    # Go to /var/log
cd /etc        # Go to /etc
cd -           # Go back to /var/log
```

---

## 🧭 Navigating the File System

### **Basic Navigation Commands:**

**pwd (Print Working Directory)**
```bash
# Where am I?
pwd

# Output: /home/yourname
```

**cd (Change Directory)**
```bash
# Go to home
cd
cd ~
cd $HOME

# Go to specific directory (absolute path)
cd /var/log
cd /etc/nginx

# Go to directory (relative path)
cd Documents
cd ../other-folder

# Go up one level
cd ..

# Go back to previous directory
cd -

# Go to root
cd /
```

**ls (List Directory Contents)**
```bash
# List current directory
ls

# Long format (detailed)
ls -l

# Show hidden files (start with .)
ls -a

# Human-readable sizes
ls -lh

# Sort by time (newest first)
ls -lt

# Reverse sort
ls -lr

# Combination (detailed, all files, human-readable)
ls -lah

# List specific directory
ls /etc
ls ~/Documents

# Recursive (show subdirectories)
ls -R
```

---

## 📊 Understanding ls -l Output

```bash
ls -l /home

# Output:
drwxr-xr-x  2 john  users  4096 Jan 23 10:30 john
-rw-r--r--  1 mary  users  1234 Jan 22 14:15 file.txt
lrwxrwxrwx  1 root  root     11 Jan 21 09:00 link -> /etc/config

Explained:
drwxr-xr-x  2 john  users  4096 Jan 23 10:30 john
│││││││││  │  │     │      │    │              │
││││││││└──┴──┴─────┴──────┴────┴──────────────┴─ Name
││││││││└─ Permissions (more in lesson 05)
││││││└─ Type: d=directory, -=file, l=link
│││││└─ Links count
││││└─ Owner
│││└─ Group
││└─ Size (bytes)
│└─ Last modified
└─ File type and permissions
```

---

## 🎯 Practical Examples

### **Finding Your Way Around:**

```bash
# Where am I?
pwd
# /home/yourname

# What's here?
ls
# Desktop  Documents  Downloads  Pictures

# Go to Documents
cd Documents
pwd
# /home/yourname/Documents

# List with details
ls -lh
# Shows size, permissions, date

# Go up one level
cd ..
pwd
# /home/yourname

# Go to system logs
cd /var/log
ls
# auth.log  syslog  kern.log

# Go back home (3 ways)
cd
# or
cd ~
# or
cd /home/yourname
```

### **Using Relative Paths:**

```bash
# Current location: /home/john
pwd
# /home/john

# Go to Documents (relative)
cd Documents
pwd
# /home/john/Documents

# Go to parent's Downloads (relative)
cd ../Downloads
pwd
# /home/john/Downloads

# Go to another user's folder (absolute needed)
cd /home/mary/Documents
pwd
# /home/mary/Documents
```

---

## 🔍 Finding Things

**Simple search in current directory:**
```bash
# Find file by name
ls -la | grep filename

# Count files in directory
ls | wc -l

# Show directory size
du -sh /var/log

# Show disk usage
df -h
```

**Tree view (if installed):**
```bash
# Install tree
sudo apt install tree

# Show directory structure
tree

# Show 2 levels deep
tree -L 2

# Show hidden files
tree -a
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between / and /root?**
   <details>
   <summary>Answer</summary>
   / is the root directory (top of filesystem). /root is the home directory for the root user.
   </details>

2. **What does ~ mean?**
   <details>
   <summary>Answer</summary>
   Your home directory (/home/username). Shortcut instead of typing full path.
   </details>

3. **What's the difference between /bin and /home?**
   <details>
   <summary>Answer</summary>
   /bin contains executable programs (commands). /home contains user personal files and directories.
   </details>

4. **Where would you find system log files?**
   <details>
   <summary>Answer</summary>
   /var/log/
   </details>

5. **What's the difference between ./file and /file?**
   <details>
   <summary>Answer</summary>
   ./file is in current directory (relative). /file is in root directory (absolute).
   </details>

---

## 🏋️ Hands-On Exercise

**Try these commands:**
```bash
# 1. Where are you?
pwd

# 2. Go to root directory
cd /

# 3. List everything
ls -la

# 4. Go to /etc
cd /etc

# 5. Find hostname file
ls | grep hostname

# 6. View it
cat hostname

# 7. Go to your home
cd ~

# 8. Create test directory
mkdir test-filesystem

# 9. Go into it
cd test-filesystem

# 10. Create file
touch myfile.txt

# 11. List with details
ls -lh

# 12. Go back home
cd ..

# 13. Remove test directory
rmdir test-filesystem

# 14. Check it's gone
ls
```

---

## 📝 Key Takeaways

✅ **/ (root)** is the top of the filesystem  
✅ **/home** contains user directories  
✅ **/etc** has configuration files  
✅ **/var** has variable data (logs, databases)  
✅ **~** is shortcut for your home directory  
✅ **Absolute paths** start with / (full address)  
✅ **Relative paths** don't start with / (from current location)  
✅ **Everything is a file** in Linux (devices, processes, etc.)  
✅ **pwd** shows where you are  
✅ **cd** changes directory  
✅ **ls** lists directory contents  

---

## 🚀 Next Steps

You understand the Linux filesystem structure!

**Next lesson:** [03-command-line-mastery.md](03-command-line-mastery.md) - Master the Linux command line

---

## 💡 Pro Tip

**Create a mental map:**
```
Root (/)
├── Home (~)         ← Your stuff
├── Config (/etc)    ← Settings
├── Logs (/var/log)  ← Troubleshooting
└── Programs (/bin)  ← Commands

Remember these 4, you'll use them 90% of the time!
```

**Bookmark this in your brain:**
```bash
cd              # Go home
cd -            # Go back
ls -lah         # List everything with details
pwd             # Where am I?
```

These 4 commands = navigation mastery! 🚀
