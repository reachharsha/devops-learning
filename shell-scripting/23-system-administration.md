---
render_with_liquid: false
---
# 23 - System Administration Scripts

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- User and group management
- Service management (systemctl)
- Log rotation and analysis
- Backup automation
- System monitoring
- Cron jobs and scheduling
- Disk and resource management

---

## 👥 User and Group Management

### **User Management:**

```bash
#!/bin/bash

# Create user
create_user() {
    local username=$1
    local fullname=$2
    
    if id "$username" &>/dev/null; then
        echo "User $username already exists"
        return 1
    fi
    
    sudo useradd -m -c "$fullname" -s /bin/bash "$username"
    echo "User $username created"
}

# Set password
set_password() {
    local username=$1
    echo "Setting password for $username"
    sudo passwd "$username"
}

# Delete user
delete_user() {
    local username=$1
    
    if ! id "$username" &>/dev/null; then
        echo "User $username does not exist"
        return 1
    fi
    
    sudo userdel -r "$username"
    echo "User $username deleted"
}

# Modify user
modify_user() {
    local username=$1
    local new_shell=$2
    
    sudo usermod -s "$new_shell" "$username"
    echo "Shell changed for $username"
}

# Lock/Unlock user
lock_user() {
    sudo usermod -L "$1"
    echo "User $1 locked"
}

unlock_user() {
    sudo usermod -U "$1"
    echo "User $1 unlocked"
}

# List users
list_users() {
    cut -d: -f1 /etc/passwd
}

# Get user info
user_info() {
    local username=$1
    id "$username"
    finger "$username" 2>/dev/null || getent passwd "$username"
}
```

### **Group Management:**

```bash
#!/bin/bash

# Create group
create_group() {
    local groupname=$1
    
    if getent group "$groupname" &>/dev/null; then
        echo "Group $groupname already exists"
        return 1
    fi
    
    sudo groupadd "$groupname"
    echo "Group $groupname created"
}

# Add user to group
add_user_to_group() {
    local username=$1
    local groupname=$2
    
    sudo usermod -aG "$groupname" "$username"
    echo "User $username added to group $groupname"
}

# Remove user from group
remove_user_from_group() {
    local username=$1
    local groupname=$2
    
    sudo gpasswd -d "$username" "$groupname"
    echo "User $username removed from group $groupname"
}

# List groups
list_groups() {
    cut -d: -f1 /etc/group
}

# List users in group
list_group_members() {
    local groupname=$1
    getent group "$groupname" | cut -d: -f4 | tr ',' '\n'
}
```

---

## 🔧 Service Management

### **systemctl Commands:**

```bash
#!/bin/bash

# Start service
start_service() {
    local service=$1
    sudo systemctl start "$service"
    echo "Service $service started"
}

# Stop service
stop_service() {
    local service=$1
    sudo systemctl stop "$service"
    echo "Service $service stopped"
}

# Restart service
restart_service() {
    local service=$1
    sudo systemctl restart "$service"
    echo "Service $service restarted"
}

# Reload service configuration
reload_service() {
    local service=$1
    sudo systemctl reload "$service"
    echo "Service $service reloaded"
}

# Enable service (start on boot)
enable_service() {
    local service=$1
    sudo systemctl enable "$service"
    echo "Service $service enabled"
}

# Disable service
disable_service() {
    local service=$1
    sudo systemctl disable "$service"
    echo "Service $service disabled"
}

# Check service status
check_service() {
    local service=$1
    systemctl is-active "$service" &>/dev/null
    
    if [ $? -eq 0 ]; then
        echo "✓ $service is running"
        return 0
    else
        echo "✗ $service is not running"
        return 1
    fi
}

# Get service status details
service_status() {
    local service=$1
    systemctl status "$service"
}

# List all services
list_services() {
    systemctl list-units --type=service --all
}

# List running services
list_running_services() {
    systemctl list-units --type=service --state=running
}

# List failed services
list_failed_services() {
    systemctl --failed --type=service
}
```

### **Service Monitor:**

```bash
#!/bin/bash

SERVICES=("nginx" "mysql" "redis")
LOG_FILE="/var/log/service_monitor.log"

monitor_services() {
    for service in "${SERVICES[@]}"; do
        if ! systemctl is-active "$service" &>/dev/null; then
            echo "[$(date)] $service is down, attempting restart" | tee -a "$LOG_FILE"
            sudo systemctl start "$service"
            
            if systemctl is-active "$service" &>/dev/null; then
                echo "[$(date)] $service restarted successfully" | tee -a "$LOG_FILE"
            else
                echo "[$(date)] Failed to restart $service" | tee -a "$LOG_FILE"
                # Send alert
                send_alert "$service failed to restart"
            fi
        fi
    done
}

send_alert() {
    local message=$1
    # Send email, Slack message, etc.
    echo "$message" | mail -s "Service Alert" admin@example.com
}

# Run monitor
monitor_services
```

---

## 📋 Log Management

### **Log Rotation:**

```bash
#!/bin/bash

# Manual log rotation
rotate_log() {
    local logfile=$1
    local keep_days=${2:-7}
    
    if [ ! -f "$logfile" ]; then
        echo "Log file not found: $logfile"
        return 1
    fi
    
    # Create backup with timestamp
    local backup="${logfile}.$(date +%Y%m%d-%H%M%S)"
    
    # Copy and truncate
    cp "$logfile" "$backup"
    : > "$logfile"
    
    # Compress old backup
    gzip "$backup"
    
    # Delete old backups
    find "$(dirname "$logfile")" -name "$(basename "$logfile")*.gz" \
        -mtime +$keep_days -delete
    
    echo "Log rotated: $logfile"
}

# Usage
rotate_log "/var/log/app.log" 7
```

### **Log Analyzer:**

```bash
#!/bin/bash

LOGFILE="/var/log/syslog"

# Count errors
count_errors() {
    grep -c "ERROR" "$LOGFILE"
}

# Count warnings
count_warnings() {
    grep -c "WARN" "$LOGFILE"
}

# Get unique errors
unique_errors() {
    grep "ERROR" "$LOGFILE" | \
        sed 's/.*ERROR: //' | \
        sort | uniq -c | sort -rn
}

# Top 10 error messages
top_errors() {
    grep "ERROR" "$LOGFILE" | \
        sed 's/.*ERROR: //' | \
        sort | uniq -c | sort -rn | head -10
}

# Errors in last hour
recent_errors() {
    local hour_ago=$(date -d '1 hour ago' '+%Y-%m-%d %H' 2>/dev/null || \
                     date -v-1H '+%Y-%m-%d %H')
    grep "$hour_ago" "$LOGFILE" | grep "ERROR"
}

# Generate report
generate_log_report() {
    cat << EOF
Log Analysis Report
===================
Date: $(date)
Log File: $LOGFILE

Total Errors: $(count_errors)
Total Warnings: $(count_warnings)

Top 10 Errors:
$(top_errors)

Recent Errors (last hour):
$(recent_errors)
EOF
}

# Run report
generate_log_report
```

---

## 💾 Backup Scripts

### **Simple Backup:**

```bash
#!/bin/bash

BACKUP_SOURCE="/home/user/data"
BACKUP_DEST="/backups"
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="backup_$DATE.tar.gz"

# Create backup
create_backup() {
    echo "Starting backup..."
    
    # Create backup directory
    mkdir -p "$BACKUP_DEST"
    
    # Create compressed archive
    tar -czf "$BACKUP_DEST/$BACKUP_FILE" "$BACKUP_SOURCE" 2>&1 | \
        tee "$BACKUP_DEST/backup_$DATE.log"
    
    if [ ${PIPESTATUS[0]} -eq 0 ]; then
        echo "Backup completed: $BACKUP_FILE"
        echo "Size: $(du -h "$BACKUP_DEST/$BACKUP_FILE" | cut -f1)"
    else
        echo "Backup failed!"
        return 1
    fi
}

# Delete old backups
cleanup_old_backups() {
    local keep_days=${1:-7}
    
    echo "Cleaning up backups older than $keep_days days..."
    find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +$keep_days -delete
}

# Run backup
create_backup
cleanup_old_backups 7
```

### **Advanced Backup with Rotation:**

```bash
#!/bin/bash

BACKUP_SOURCE="/var/www"
BACKUP_DEST="/backups"
RETENTION_DAYS=30
LOG_FILE="$BACKUP_DEST/backup.log"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Check prerequisites
check_prerequisites() {
    # Check source exists
    if [ ! -d "$BACKUP_SOURCE" ]; then
        log "ERROR: Source directory not found: $BACKUP_SOURCE"
        return 1
    fi
    
    # Check backup destination
    mkdir -p "$BACKUP_DEST" || {
        log "ERROR: Cannot create backup directory"
        return 1
    }
    
    # Check disk space
    local required=$(du -sb "$BACKUP_SOURCE" | cut -f1)
    local available=$(df "$BACKUP_DEST" | tail -1 | awk '{print $4}')
    available=$((available * 1024))
    
    if [ $available -lt $required ]; then
        log "ERROR: Insufficient disk space"
        return 1
    fi
    
    return 0
}

# Create backup
create_backup() {
    local date=$(date +%Y%m%d_%H%M%S)
    local backup_file="backup_$date.tar.gz"
    
    log "Starting backup: $BACKUP_SOURCE"
    
    if tar -czf "$BACKUP_DEST/$backup_file" "$BACKUP_SOURCE" 2>&1 | tee -a "$LOG_FILE"; then
        local size=$(du -h "$BACKUP_DEST/$backup_file" | cut -f1)
        log "Backup completed: $backup_file (Size: $size)"
        
        # Create latest symlink
        ln -sf "$backup_file" "$BACKUP_DEST/latest.tar.gz"
        
        return 0
    else
        log "ERROR: Backup failed"
        return 1
    fi
}

# Cleanup old backups
cleanup() {
    log "Cleaning up backups older than $RETENTION_DAYS days"
    
    find "$BACKUP_DEST" -name "backup_*.tar.gz" -mtime +$RETENTION_DAYS | while read file; do
        log "Deleting: $file"
        rm -f "$file"
    done
}

# Send notification
send_notification() {
    local status=$1
    local message="Backup $status at $(date)"
    
    # Send email
    echo "$message" | mail -s "Backup $status" admin@example.com
}

# Main execution
main() {
    log "=== Backup Started ==="
    
    if ! check_prerequisites; then
        send_notification "FAILED (Prerequisites)"
        exit 1
    fi
    
    if create_backup; then
        cleanup
        send_notification "SUCCESS"
        log "=== Backup Completed Successfully ==="
        exit 0
    else
        send_notification "FAILED"
        log "=== Backup Failed ==="
        exit 1
    fi
}

main
```

---

## 📊 System Monitoring

### **Resource Monitor:**

```bash
#!/bin/bash

# CPU usage
get_cpu_usage() {
    top -bn1 | grep "Cpu(s)" | \
        sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | \
        awk '{print 100 - $1"%"}'
}

# Memory usage
get_memory_usage() {
    free | grep Mem | awk '{printf "%.2f%%", $3/$2 * 100.0}'
}

# Disk usage
get_disk_usage() {
    df -h / | tail -1 | awk '{print $5}'
}

# Load average
get_load_average() {
    uptime | awk -F'load average:' '{print $2}'
}

# System uptime
get_uptime() {
    uptime -p
}

# Generate system report
system_report() {
    cat << EOF
System Status Report
====================
Date: $(date)
Hostname: $(hostname)
Uptime: $(get_uptime)

Resources:
  CPU Usage: $(get_cpu_usage)
  Memory Usage: $(get_memory_usage)
  Disk Usage: $(get_disk_usage)
  Load Average: $(get_load_average)

Top Processes (CPU):
$(ps aux --sort=-%cpu | head -6 | tail -5 | awk '{printf "  %-10s %5s%%  %s\n", $1, $3, $11}')

Top Processes (Memory):
$(ps aux --sort=-%mem | head -6 | tail -5 | awk '{printf "  %-10s %5s%%  %s\n", $1, $4, $11}')
EOF
}

# Check thresholds
check_thresholds() {
    local cpu=$(get_cpu_usage | tr -d '%')
    local mem=$(get_memory_usage | tr -d '%')
    local disk=$(get_disk_usage | tr -d '%')
    
    if (( $(echo "$cpu > 80" | bc -l) )); then
        echo "⚠️  High CPU usage: $cpu%"
    fi
    
    if (( $(echo "$mem > 80" | bc -l) )); then
        echo "⚠️  High memory usage: $mem%"
    fi
    
    if (( $(echo "$disk > 80" | bc -l) )); then
        echo "⚠️  High disk usage: $disk%"
    fi
}

# Run report
system_report
echo ""
check_thresholds
```

---

## ⏰ Cron Job Management

### **Cron Syntax:**

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
# │ │ │ │ │
# * * * * * command
```

### **Common Cron Examples:**

```bash
#!/bin/bash

# Every minute
# * * * * * /path/to/script.sh

# Every 5 minutes
# */5 * * * * /path/to/script.sh

# Every hour
# 0 * * * * /path/to/script.sh

# Every day at 2:30 AM
# 30 2 * * * /path/to/script.sh

# Every Monday at 9:00 AM
# 0 9 * * 1 /path/to/script.sh

# First day of every month
# 0 0 1 * * /path/to/script.sh

# Every weekday at 6:00 PM
# 0 18 * * 1-5 /path/to/script.sh
```

### **Cron Management Script:**

```bash
#!/bin/bash

# Add cron job
add_cron_job() {
    local schedule=$1
    local command=$2
    
    # Add to crontab
    (crontab -l 2>/dev/null; echo "$schedule $command") | crontab -
    
    echo "Cron job added: $schedule $command"
}

# Remove cron job
remove_cron_job() {
    local pattern=$1
    
    crontab -l | grep -v "$pattern" | crontab -
    
    echo "Cron job removed: $pattern"
}

# List cron jobs
list_cron_jobs() {
    echo "Current cron jobs:"
    crontab -l
}

# Usage
add_cron_job "0 2 * * *" "/home/user/backup.sh"
list_cron_jobs
```

---

## 🎯 Practical Examples

### **Complete System Admin Script:**

```bash
#!/bin/bash

###############################################################################
# System Administration Script
# Performs various system maintenance tasks
###############################################################################

set -euo pipefail

LOG_FILE="/var/log/sysadmin.log"
ALERT_EMAIL="admin@example.com"

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

# Update system packages
update_system() {
    log "Updating system packages..."
    
    if command -v apt-get &>/dev/null; then
        sudo apt-get update && sudo apt-get upgrade -y
    elif command -v yum &>/dev/null; then
        sudo yum update -y
    fi
    
    log "System updated"
}

# Clean package cache
clean_cache() {
    log "Cleaning package cache..."
    
    if command -v apt-get &>/dev/null; then
        sudo apt-get clean
        sudo apt-get autoclean
        sudo apt-get autoremove -y
    elif command -v yum &>/dev/null; then
        sudo yum clean all
    fi
    
    log "Cache cleaned"
}

# Check disk usage
check_disk_usage() {
    log "Checking disk usage..."
    
    df -h | awk 'NR>1 {gsub(/%/,"",$5); if($5>80) print $0}' | while read line; do
        log "WARNING: High disk usage - $line"
        echo "High disk usage detected: $line" | \
            mail -s "Disk Usage Alert" "$ALERT_EMAIL"
    done
}

# Clean temporary files
clean_temp_files() {
    log "Cleaning temporary files..."
    
    sudo find /tmp -type f -mtime +7 -delete
    sudo find /var/tmp -type f -mtime +7 -delete
    
    log "Temporary files cleaned"
}

# Check failed login attempts
check_failed_logins() {
    log "Checking failed login attempts..."
    
    local failed=$(grep "Failed password" /var/log/auth.log 2>/dev/null | wc -l || echo 0)
    
    if [ "$failed" -gt 10 ]; then
        log "WARNING: $failed failed login attempts detected"
        echo "$failed failed login attempts detected" | \
            mail -s "Security Alert" "$ALERT_EMAIL"
    fi
}

# Main execution
main() {
    log "=== System Maintenance Started ==="
    
    update_system
    clean_cache
    clean_temp_files
    check_disk_usage
    check_failed_logins
    
    log "=== System Maintenance Completed ==="
}

main
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you create a new user?**
   <details>
   <summary>Answer</summary>
   sudo useradd -m -s /bin/bash username
   </details>

2. **How do you restart a service?**
   <details>
   <summary>Answer</summary>
   sudo systemctl restart service_name
   </details>

3. **How do you check if a service is running?**
   <details>
   <summary>Answer</summary>
   systemctl is-active service_name
   </details>

4. **How do you schedule a daily cron job?**
   <details>
   <summary>Answer</summary>
   0 0 * * * /path/to/script.sh (midnight daily)
   </details>

5. **How do you create a compressed backup?**
   <details>
   <summary>Answer</summary>
   tar -czf backup.tar.gz /path/to/directory
   </details>

---

## 📝 Key Takeaways

✅ **useradd/usermod for user management**  
✅ **systemctl for service control**  
✅ **tar -czf for backups**  
✅ **cron for scheduling tasks**  
✅ **Always log administrative actions**  
✅ **Monitor disk, CPU, memory usage**  
✅ **Rotate and archive logs**  

---

## 🚀 Next Steps

You're now a system administrator!

**Next lesson:** [24 - DevOps Automation](24-devops-automation.md) - CI/CD, containers, deployments

---

## 💡 Pro Tips

**Always backup before changes:**
```bash
cp /etc/config /etc/config.bak.$(date +%Y%m%d)
```

**Log everything:**
```bash
exec 1> >(tee -a /var/log/script.log)
exec 2>&1
```

**Test cron jobs:**
```bash
run-parts --test /etc/cron.daily
```

System administration = keeping systems healthy! 🔧
