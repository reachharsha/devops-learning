# 07 - Process Management

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What processes are and how they work
- Viewing running processes (ps, top, htop)
- Managing processes (kill, jobs, bg, fg)
- System monitoring (uptime, free, df, du)
- Understanding system resources

---

## 🔄 What is a Process?

**Real-World Analogy:**
```
Program = Recipe in a cookbook (stored instructions)
Process = Actually cooking the dish (running the recipe)

One recipe (program) can be used multiple times
Each time = separate process
```

**Technical Definition:**
```
Process = Running instance of a program

Program on disk:  /usr/bin/firefox  (file, not running)
Process in RAM:   firefox (PID 1234) (running, using CPU/RAM)
```

### **Every Process Has:**

```
PID (Process ID):     Unique number (like employee ID)
PPID (Parent PID):    Who started it (parent process)
User:                 Who owns it
State:                Running, Sleeping, Stopped, Zombie
Priority:             How important it is
Resources:            CPU%, Memory%, Open files
```

### **Process States:**

```
R  Running       Currently executing or ready to run
S  Sleeping      Waiting for an event (most common)
D  Disk Sleep    Waiting for I/O (uninterruptible)
T  Stopped       Paused (Ctrl+Z)
Z  Zombie        Finished but parent hasn't cleaned up
```

---

## 📊 Viewing Processes

### **ps - Process Status**

**Basic usage:**
```bash
# Show processes in current terminal
ps

# All processes, full format
ps aux

# All processes, tree view
ps auxf

# Specific user's processes
ps -u username

# Search for specific process
ps aux | grep firefox
```

**Understanding ps aux output:**
```bash
ps aux

# Output:
USER    PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
john   1234   2.5  1.3 123456  8192 pts/0  S+   10:30   0:05 firefox
│      │     │    │     │      │    │     │     │      │     │
│      │     │    │     │      │    │     │     │      │     └─ Command name
│      │     │    │     │      │    │     │     │      └─ CPU time used
│      │     │    │     │      │    │     │     └─ Start time
│      │     │    │     │      │    │     └─ State (S=Sleeping, R=Running)
│      │     │    │     │      │    └─ Terminal
│      │     │    │     │      └─ Physical RAM (KB)
│      │     │    │     └─ Virtual memory size (KB)
│      │     │    └─ Memory percentage
│      │     └─ CPU percentage
│      └─ Process ID
└─ Owner
```

**Useful ps commands:**
```bash
# Custom format
ps -eo pid,ppid,cmd,%mem,%cpu --sort=-%cpu | head

# Find specific processes
ps aux | grep nginx
ps aux | grep python
ps aux | grep docker

# Process tree (shows parent-child relationships)
ps auxf
pstree

# Specific process by PID
ps -p 1234

# All processes for current user
ps -u $USER

# Processes using most CPU
ps aux --sort=-%cpu | head -10

# Processes using most memory
ps aux --sort=-%mem | head -10
```

---

### **top - Interactive Process Viewer**

**Real-time system monitoring**

```bash
# Start top
top

# Top with better defaults
top -o %CPU        # Sort by CPU
top -o %MEM        # Sort by memory
```

**Keyboard shortcuts in top:**
```
P     Sort by CPU% (default)
M     Sort by Memory%
k     Kill a process (prompts for PID)
r     Renice (change priority)
u     Filter by user
1     Toggle individual CPU cores
q     Quit
h     Help
Space Update display now
```

**Understanding top output:**
```
top - 10:30:15 up 5 days,  2:15,  3 users,  load average: 0.15, 0.20, 0.18
      │        │                  │         │
      │        │                  │         └─ Load: 1min, 5min, 15min avg
      │        │                  └─ Number of logged-in users
      │        └─ System uptime (5 days, 2 hours, 15 minutes)
      └─ Current time

Tasks: 245 total,   1 running, 244 sleeping,   0 stopped,   0 zombie
%Cpu(s):  5.2 us,  1.3 sy,  0.0 ni, 93.2 id,  0.3 wa,  0.0 hi,  0.0 si
          │        │        │       │        │       │       │
          │        │        │       │        │       │       └─ Software interrupts
          │        │        │       │        │       └─ Hardware interrupts
          │        │        │       │        └─ I/O wait
          │        │        │       └─ Idle
          │        │        └─ Nice (low priority)
          │        └─ System (kernel)
          └─ User processes

MiB Mem :   7932.5 total,   1234.2 free,   3456.1 used,   3242.2 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   4123.3 avail Mem
```

**Load Average explained:**
```
Load average: 0.15, 0.20, 0.18
              │     │     │
              │     │     └─ 15-minute average
              │     └─ 5-minute average
              └─ 1-minute average

Interpretation:
< 1.0    System not busy
1.0-2.0  System moderately busy
> 2.0    System very busy (on single-core CPU)

For multi-core: Divide by number of cores
4-core CPU with load 2.0 = 50% utilized
```

---

### **htop - Enhanced top**

**Better alternative to top (more user-friendly)**

```bash
# Install htop
sudo apt install htop     # Ubuntu/Debian
sudo yum install htop     # CentOS/RHEL

# Run htop
htop
```

**htop features:**
- ✅ Color-coded display
- ✅ Mouse support (click to select)
- ✅ Easier process management
- ✅ Tree view of processes
- ✅ Search functionality
- ✅ Filter by user

**htop keyboard shortcuts:**
```
F1    Help
F2    Setup
F3    Search process
F4    Filter by text
F5    Tree view
F6    Sort by column
F9    Kill process
F10   Quit

/     Search
u     Filter by user
k     Kill selected process
Space Tag process
```

---

## 🎮 Managing Processes

### **kill - Terminate Processes**

**Sending signals to processes**

```bash
# Graceful termination (default)
kill PID
kill 1234

# Force kill (immediate termination)
kill -9 PID
kill -9 1234

# Reload configuration (common for services)
kill -1 PID
kill -HUP PID
```

**Common signals:**
```bash
# List all signals
kill -l

# Most used signals:
SIGTERM (15)  Graceful shutdown (default)
              Process can clean up before exiting

SIGKILL (9)   Force kill (immediate)
              Cannot be caught or ignored

SIGHUP (1)    Hangup signal
              Often used to reload config

SIGINT (2)    Interrupt (Ctrl+C)
              Graceful interrupt

# Examples:
kill -15 1234      # Graceful (same as: kill 1234)
kill -SIGTERM 1234 # Same as above
kill -9 1234       # Force kill
kill -SIGKILL 1234 # Same as above
kill -1 1234       # Reload config
kill -HUP 1234     # Same as above
```

### **killall - Kill by Name**

```bash
# Kill all processes with name
killall firefox

# Kill with specific signal
killall -9 chrome

# Interactive (ask before killing)
killall -i nginx

# Case-insensitive
killall -I Firefox
```

### **pkill - Pattern-based Kill**

```bash
# Kill processes matching pattern
pkill firefox

# Kill by user
pkill -u john

# Kill with specific signal
pkill -9 python

# Combine criteria
pkill -9 -u john firefox
```

---

## 🎯 Background & Foreground Jobs

**Running processes in background**

```bash
# Run command in background
command &

# Examples:
sleep 100 &
firefox &
./long-running-script.sh &

# View background jobs
jobs

# Output:
[1]+ Running    sleep 100 &
[2]  Running    firefox &

# Bring job to foreground
fg %1              # Job number 1
fg %2              # Job number 2

# Send job to background
bg %1

# Suspend current job (Ctrl+Z)
# Then resume in background
Ctrl+Z             # Suspend
bg                 # Resume in background
```

**Workflow example:**
```bash
# Start process
firefox

# Oops, need terminal back
Ctrl+Z             # Suspend (pauses firefox)
# [1]+  Stopped    firefox

# Resume in background
bg
# [1]+ firefox &

# Continue working in terminal
ls
cd /etc

# Bring firefox back to foreground
fg
```

### **nohup - Run After Logout**

```bash
# Keep process running after logout
nohup command &

# Example:
nohup ./long-script.sh &

# Output goes to nohup.out
cat nohup.out

# Redirect output
nohup ./script.sh > output.log 2>&1 &
```

---

## 📈 System Monitoring

### **uptime - System Load**

```bash
# Show system uptime and load
uptime

# Output:
10:30:15 up 5 days, 2:15, 3 users, load average: 0.15, 0.20, 0.18
│        │                 │         │
│        │                 │         └─ Load averages (1min, 5min, 15min)
│        │                 └─ Logged-in users
│        └─ How long system has been running
└─ Current time
```

---

### **free - Memory Usage**

```bash
# Memory usage in human-readable format
free -h

# Output:
              total     used     free   shared  buff/cache available
Mem:           7.7G     3.4G     1.2G     200M        3.2G      4.0G
Swap:          2.0G       0B     2.0G

# Update every 2 seconds
free -h -s 2

# Memory in MB
free -m

# Memory in GB
free -g

# Show total (sum)
free -h -t
```

**Understanding memory:**
```
total      Total installed RAM
used       Used by applications
free       Completely unused
shared     Shared memory (tmpfs)
buff/cache Kernel buffers and cache
available  Available for applications (free + reclaimable cache)

Important: "available" is what you should look at!
Linux uses free RAM for cache (makes system faster)
This cache is automatically freed when apps need it
```

---

### **df - Disk Space**

```bash
# Disk usage, human-readable
df -h

# Output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G   15G  4.2G  78% /
/dev/sdb1       100G   50G   46G  52% /data
tmpfs           3.9G     0  3.9G   0% /dev/shm

# Specific filesystem
df -h /

# Show inodes (file count)
df -i

# Only show local filesystems
df -h -l

# Exclude filesystem type
df -h -x tmpfs
```

---

### **du - Directory Usage**

```bash
# Size of current directory
du -sh .

# Size of specific directory
du -sh /var/log

# All subdirectories
du -h /var/log

# Top 10 largest directories
du -h /var | sort -rh | head -10

# Exclude subdirectories
du -h --max-depth=1

# Exclude patterns
du -h --exclude="*.log"
du -h --exclude="node_modules"

# Find large files
du -ah /home | sort -rh | head -20
```

**Practical example:**
```bash
# Find what's using disk space
cd /
du -h --max-depth=1 | sort -rh

# Check home directory usage
du -sh ~/*  | sort -rh

# Find large log files
find /var/log -type f -exec du -h {} \; | sort -rh | head -10
```

---

## 🚀 Practical Scenarios

### **Find and Kill High CPU Process:**

```bash
# Method 1: Using top
top
# Press 'k', enter PID, press Enter

# Method 2: Command line
ps aux --sort=-%cpu | head -5
kill -15 PID

# Method 3: One-liner
kill $(ps aux --sort=-%cpu | awk 'NR==2 {print $2}')
```

### **Find Memory Hogs:**

```bash
# Top 10 memory consumers
ps aux --sort=-%mem | head -11

# Kill process using most memory
pkill -9 -f process-name
```

### **Monitor Specific Process:**

```bash
# Watch process continuously
watch -n 1 'ps aux | grep nginx'

# Monitor with top
top -p PID

# Example:
top -p 1234
```

### **Clean Up Zombie Processes:**

```bash
# Find zombies
ps aux | grep Z

# Kill parent process (zombies will disappear)
kill -15 PPID

# If many zombies, might need reboot
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you see all running processes?**
   <details>
   <summary>Answer</summary>
   ps aux
   </details>

2. **How do you force kill process with PID 1234?**
   <details>
   <summary>Answer</summary>
   kill -9 1234
   </details>

3. **How do you run a command in the background?**
   <details>
   <summary>Answer</summary>
   command &
   </details>

4. **What's the difference between kill and kill -9?**
   <details>
   <summary>Answer</summary>
   kill (or kill -15) allows graceful shutdown. kill -9 forces immediate termination.
   </details>

5. **How do you check memory usage?**
   <details>
   <summary>Answer</summary>
   free -h
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. View all processes
ps aux

# 2. Find your processes
ps -u $USER

# 3. Start background process
sleep 300 &

# 4. View jobs
jobs

# 5. Start top
top
# Press 'q' to quit

# 6. Check system uptime
uptime

# 7. Check memory
free -h

# 8. Check disk space
df -h

# 9. Check current directory size
du -sh .

# 10. Find the background sleep job
jobs

# 11. Kill it
kill %1

# 12. Verify it's gone
jobs
```

---

## 📝 Key Takeaways

✅ **ps aux** - see all processes  
✅ **top/htop** - real-time monitoring (htop is better!)  
✅ **kill PID** - graceful termination  
✅ **kill -9 PID** - force kill (last resort)  
✅ **command &** - run in background  
✅ **jobs, bg, fg** - manage background jobs  
✅ **Ctrl+Z** - suspend current process  
✅ **uptime** - system load averages  
✅ **free -h** - memory usage  
✅ **df -h** - disk space  
✅ **du -sh** - directory size  

---

## 🚀 Next Steps

You can now monitor and manage processes like a pro!

**Next lesson:** [08-package-management.md](08-package-management.md) - Installing and managing software

---

## 💡 Pro Tips

**Quick process management:**
```bash
# Kill all Python processes
pkill python

# Kill all processes for user
pkill -u username

# Find what's using a file
lsof /path/to/file

# Find what's using a port
lsof -i :8080
netstat -tlnp | grep 8080
```

**Monitoring tips:**
```bash
# Watch command output
watch -n 1 'df -h'
watch -n 1 'free -h'

# Monitor specific process
watch -n 1 'ps aux | grep nginx'

# Real-time log + process monitoring
# Terminal 1:
tail -f /var/log/app.log
# Terminal 2:
htop
```

**Remember:**
```
Always try kill (SIGTERM) before kill -9 (SIGKILL)
SIGTERM allows cleanup, SIGKILL does not!

Use htop instead of top - it's much better!
Install it: sudo apt install htop
```

Master process management = control your system! 🚀
