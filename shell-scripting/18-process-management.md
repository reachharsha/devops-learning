# 18 - Process Management

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Foreground vs background processes
- Job control (jobs, fg, bg)
- Process signals and the kill command
- The trap command for signal handling
- Parallel execution
- Process monitoring
- Timeout and retry patterns

---

## 🔄 Foreground vs Background Processes

### **Foreground Processes:**

```bash
#!/bin/bash

# Runs in foreground (blocks terminal)
sleep 30

# While it's running, you can't use the terminal
# Press Ctrl+C to terminate
# Press Ctrl+Z to suspend
```

### **Background Processes:**

```bash
#!/bin/bash

# Run in background with &
sleep 30 &

# Terminal is free to use
echo "Sleep is running in background"

# Get process ID
BGPID=$!
echo "Background PID: $BGPID"

# Check if still running
if ps -p $BGPID > /dev/null; then
    echo "Process is running"
fi
```

---

## 🎮 Job Control

### **Managing Jobs:**

```bash
#!/bin/bash

# Start multiple background jobs
sleep 100 &
sleep 200 &
sleep 300 &

# List jobs
jobs
# Output:
# [1]   Running    sleep 100 &
# [2]   Running    sleep 200 &
# [3]-  Running    sleep 300 &

# Bring job to foreground
# fg %1          # Job 1
# fg %2          # Job 2
# fg             # Most recent job

# Send foreground job to background
# 1. Press Ctrl+Z (suspends)
# 2. bg %1       (resumes in background)

# Kill job
# kill %1        # Kill job 1
# kill %2        # Kill job 2
```

### **Job Commands:**

| Command | Description | Example |
|---------|-------------|---------|
| `jobs` | List jobs | `jobs -l` (with PIDs) |
| `fg` | Foreground job | `fg %1` |
| `bg` | Background job | `bg %1` |
| `kill %n` | Kill job n | `kill %1` |
| `disown` | Remove from job table | `disown %1` |
| `wait` | Wait for job to finish | `wait %1` |

### **Wait for Background Processes:**

```bash
#!/bin/bash

# Start background jobs
sleep 5 &
PID1=$!

sleep 3 &
PID2=$!

sleep 7 &
PID3=$!

echo "Started 3 background jobs"

# Wait for specific process
wait $PID1
echo "First job finished"

# Wait for all background jobs
wait
echo "All jobs finished"
```

---

## ⚡ Process Signals

### **Common Signals:**

| Signal | Number | Meaning | Can be caught? |
|--------|--------|---------|----------------|
| `SIGTERM` | 15 | Terminate (graceful) | ✅ Yes |
| `SIGKILL` | 9 | Kill (forceful) | ❌ No |
| `SIGINT` | 2 | Interrupt (Ctrl+C) | ✅ Yes |
| `SIGQUIT` | 3 | Quit | ✅ Yes |
| `SIGHUP` | 1 | Hangup | ✅ Yes |
| `SIGSTOP` | 19 | Stop (Ctrl+Z) | ❌ No |
| `SIGCONT` | 18 | Continue | ✅ Yes |
| `SIGUSR1` | 10 | User-defined | ✅ Yes |
| `SIGUSR2` | 12 | User-defined | ✅ Yes |

### **Sending Signals:**

```bash
#!/bin/bash

# Start a process
sleep 100 &
PID=$!

# Send SIGTERM (graceful termination)
kill $PID
# or
kill -15 $PID
# or
kill -TERM $PID

# Force kill with SIGKILL (cannot be caught)
kill -9 $PID
# or
kill -KILL $PID

# Send custom signal
kill -USR1 $PID

# Kill all processes by name
killall sleep
pkill sleep
```

---

## 🛡️ The trap Command

**Catch and handle signals:**

### **Basic trap Usage:**

```bash
#!/bin/bash

# Cleanup on exit
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/myfile.tmp
    echo "Done"
}

trap cleanup EXIT

# Script logic
echo "Creating temp file..."
touch /tmp/myfile.tmp
echo "Doing work..."
sleep 5
echo "Finished"

# cleanup() runs automatically on exit
```

### **Catching Specific Signals:**

```bash
#!/bin/bash

# Catch Ctrl+C (SIGINT)
trap 'echo "Caught SIGINT, exiting..."; exit 1' INT

echo "Press Ctrl+C to trigger trap"
while true; do
    echo "Working... $(date)"
    sleep 2
done
```

### **Trap Multiple Signals:**

```bash
#!/bin/bash

# Handle multiple signals
trap 'echo "Terminating..."; exit' INT TERM QUIT

# Ignore signal
trap '' INT         # Ignore Ctrl+C

# Reset to default
trap - INT          # Reset INT to default behavior
```

### **Trap Patterns:**

```bash
#!/bin/bash

# Pattern 1: Cleanup resources
TEMPFILE=$(mktemp)
trap "rm -f '$TEMPFILE'" EXIT

# Pattern 2: Lock file cleanup
LOCKFILE="/var/lock/myapp.lock"
trap "rm -f '$LOCKFILE'" EXIT
touch "$LOCKFILE"

# Pattern 3: Kill child processes
trap 'kill $(jobs -p)' EXIT

# Start background jobs
sleep 100 &
sleep 200 &

# When script exits, all children are killed

# Pattern 4: Graceful shutdown
shutdown() {
    echo "Shutting down gracefully..."
    # Stop services
    # Save state
    exit 0
}

trap shutdown INT TERM

while true; do
    # Main loop
    sleep 1
done
```

---

## ⚡ Parallel Execution

### **Simple Parallel Execution:**

```bash
#!/bin/bash

# Sequential (slow)
for i in {1..5}; do
    sleep 1
    echo "Task $i done"
done
# Takes 5 seconds

# Parallel (fast)
for i in {1..5}; do
    {
        sleep 1
        echo "Task $i done"
    } &
done
wait
# Takes 1 second
```

### **Limit Concurrent Jobs:**

```bash
#!/bin/bash

MAX_JOBS=3
JOB_COUNT=0

for i in {1..10}; do
    # Wait if too many jobs
    while [ $(jobs -r | wc -l) -ge $MAX_JOBS ]; do
        sleep 0.1
    done
    
    # Start job
    {
        echo "Processing task $i"
        sleep 2
    } &
done

# Wait for all to complete
wait
echo "All tasks completed"
```

### **Parallel with xargs:**

```bash
#!/bin/bash

# Process files in parallel
find . -name "*.jpg" | xargs -P 4 -I {} convert {} {}.resized.jpg

# -P 4: use 4 parallel processes
# -I {}: replace string

# Parallel execution of commands
cat urls.txt | xargs -P 10 -n 1 curl -O

# Process list in parallel
printf "%s\n" {1..100} | xargs -P 10 -I {} bash -c 'echo "Processing {}"; sleep 1'
```

---

## 🔍 Process Monitoring

### **Check if Process is Running:**

```bash
#!/bin/bash

# By PID
if ps -p $PID > /dev/null 2>&1; then
    echo "Process is running"
else
    echo "Process is not running"
fi

# By name
if pgrep nginx > /dev/null; then
    echo "nginx is running"
else
    echo "nginx is not running"
fi

# Get PID by name
NGINX_PID=$(pgrep nginx)
echo "nginx PID: $NGINX_PID"

# Get all PIDs by name
pgrep -a nginx      # With command line
pidof nginx         # Just PIDs
```

### **Monitor Process Resource Usage:**

```bash
#!/bin/bash

# CPU and memory for specific process
ps -p $PID -o %cpu,%mem,cmd

# Top CPU consumers
ps aux --sort=-%cpu | head -10

# Top memory consumers
ps aux --sort=-%mem | head -10

# Process tree
pstree -p $PID

# Detailed process info
cat /proc/$PID/status
```

---

## ⏱️ Timeout and Retry Patterns

### **Timeout a Command:**

```bash
#!/bin/bash

# Method 1: timeout command (if available)
timeout 10s long_running_command
if [ $? -eq 124 ]; then
    echo "Command timed out"
fi

# Method 2: Manual timeout
{
    long_running_command &
    CMD_PID=$!
    
    sleep 10
    if ps -p $CMD_PID > /dev/null; then
        echo "Killing process after timeout"
        kill $CMD_PID
    fi
}

# Method 3: timeout function
run_with_timeout() {
    local timeout=$1
    shift
    local command="$@"
    
    $command &
    local pid=$!
    
    local count=0
    while [ $count -lt $timeout ]; do
        if ! ps -p $pid > /dev/null; then
            wait $pid
            return $?
        fi
        sleep 1
        ((count++))
    done
    
    kill $pid 2>/dev/null
    return 124
}

run_with_timeout 5 sleep 10
```

### **Retry Logic:**

```bash
#!/bin/bash

# Simple retry
retry() {
    local max_attempts=$1
    shift
    local command="$@"
    local attempt=1
    
    until $command; do
        if [ $attempt -eq $max_attempts ]; then
            echo "Failed after $max_attempts attempts"
            return 1
        fi
        echo "Attempt $attempt failed, retrying..."
        ((attempt++))
        sleep 2
    done
    
    return 0
}

# Usage
retry 5 curl -s https://api.example.com

# Retry with exponential backoff
retry_backoff() {
    local max_attempts=$1
    shift
    local command="$@"
    local attempt=1
    
    until $command; do
        if [ $attempt -eq $max_attempts ]; then
            echo "Failed after $max_attempts attempts"
            return 1
        fi
        
        local wait_time=$((2 ** attempt))
        echo "Attempt $attempt failed, waiting ${wait_time}s..."
        sleep $wait_time
        ((attempt++))
    done
    
    return 0
}

retry_backoff 5 curl -s https://api.example.com
```

---

## 🎯 Practical Examples

### **Example 1: Service Monitor**

```bash
#!/bin/bash

SERVICE="nginx"

monitor_service() {
    while true; do
        if ! pgrep $SERVICE > /dev/null; then
            echo "[$(date)] $SERVICE is down, restarting..."
            systemctl start $SERVICE
            
            # Send alert
            echo "Service $SERVICE restarted at $(date)" | \
                mail -s "Service Alert" admin@example.com
        fi
        
        sleep 60
    done
}

# Run monitor in background
monitor_service &
echo "Service monitor started (PID: $!)"
```

### **Example 2: Parallel Backup**

```bash
#!/bin/bash

SOURCES=(
    "/home/user/documents"
    "/home/user/pictures"
    "/home/user/videos"
    "/etc"
)

BACKUP_DIR="/backups/$(date +%Y%m%d)"
mkdir -p "$BACKUP_DIR"

echo "Starting parallel backup..."

for source in "${SOURCES[@]}"; do
    {
        name=$(basename "$source")
        echo "Backing up $source..."
        tar -czf "$BACKUP_DIR/${name}.tar.gz" "$source" 2>/dev/null
        echo "Completed: $source"
    } &
done

wait
echo "All backups completed"
```

### **Example 3: Progress Monitor**

```bash
#!/bin/bash

# Long-running task with progress
long_task() {
    for i in {1..100}; do
        # Simulate work
        sleep 0.1
        echo $i
    done
}

# Run task and monitor
long_task | while read progress; do
    printf "\rProgress: %d%%" $progress
done
printf "\nDone!\n"
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you run a command in the background?**
   <details>
   <summary>Answer</summary>
   command &
   </details>

2. **How do you get the PID of the last background process?**
   <details>
   <summary>Answer</summary>
   $!
   </details>

3. **What signal number is SIGKILL?**
   <details>
   <summary>Answer</summary>
   9 (kill -9)
   </details>

4. **How do you wait for all background jobs?**
   <details>
   <summary>Answer</summary>
   wait (with no arguments)
   </details>

5. **How do you catch Ctrl+C in a script?**
   <details>
   <summary>Answer</summary>
   trap 'command' INT
   </details>

---

## 📝 Key Takeaways

✅ **& runs process in background**  
✅ **$! gets last background PID**  
✅ **wait waits for background processes**  
✅ **trap catches signals for cleanup**  
✅ **kill -15 (SIGTERM) for graceful stop**  
✅ **kill -9 (SIGKILL) for forceful stop**  
✅ **jobs, fg, bg for job control**  

---

## 🚀 Next Steps

You now master process management!

**Next lesson:** [19 - Error Handling](19-error-handling.md) - Robust error handling and validation

---

## 💡 Pro Tips

**Always cleanup:**
```bash
trap "rm -f /tmp/*.tmp" EXIT
```

**Safe background execution:**
```bash
command &
BGPID=$!
wait $BGPID || echo "Command failed"
```

**Prevent zombie processes:**
```bash
command & disown
```

Master processes, master automation! ⚙️
