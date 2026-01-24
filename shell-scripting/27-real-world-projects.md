---
render_with_liquid: false
---
# 27 - Real-World Projects

## 🎯 Learning Objectives
By the end of this lesson, you'll have:
- Complete, production-ready script examples
- Automated backup system
- Log analyzer and monitoring tool
- Deployment automation
- System health checker
- Database backup utility
- Multi-server management script

---

## 💾 Project 1: Automated Backup System

### **Complete Backup Solution:**

```bash
#!/bin/bash

###############################################################################
# Automated Backup System
# Features: Incremental backups, encryption, rotation, monitoring
###############################################################################

set -euo pipefail

#------------------------------------------------------------------------------
# CONFIGURATION
#------------------------------------------------------------------------------

SCRIPT_NAME="$(basename "$0")"
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Configuration file
CONFIG_FILE="${CONFIG_FILE:-/etc/backup/config.conf}"

# Default values
BACKUP_SOURCES=()
BACKUP_DEST="/var/backups"
RETENTION_DAYS=30
ENCRYPTION_ENABLED=false
ENCRYPTION_KEY=""
REMOTE_BACKUP_ENABLED=false
REMOTE_USER=""
REMOTE_HOST=""
REMOTE_PATH=""
NOTIFY_EMAIL=""
LOG_FILE="/var/log/backup.log"

#------------------------------------------------------------------------------
# FUNCTIONS
#------------------------------------------------------------------------------

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

error() {
    log "ERROR: $*"
    exit 1
}

send_notification() {
    local subject=$1
    local message=$2
    
    if [ -n "$NOTIFY_EMAIL" ]; then
        echo "$message" | mail -s "$subject" "$NOTIFY_EMAIL"
    fi
}

load_config() {
    if [ ! -f "$CONFIG_FILE" ]; then
        log "Config file not found: $CONFIG_FILE"
        return 1
    fi
    
    # shellcheck disable=SC1090
    source "$CONFIG_FILE"
    
    log "Configuration loaded"
}

validate_config() {
    # Check backup sources
    if [ ${#BACKUP_SOURCES[@]} -eq 0 ]; then
        error "No backup sources defined"
    fi
    
    for source in "${BACKUP_SOURCES[@]}"; do
        if [ ! -d "$source" ]; then
            error "Backup source not found: $source"
        fi
    done
    
    # Check backup destination
    if [ ! -d "$BACKUP_DEST" ]; then
        log "Creating backup destination: $BACKUP_DEST"
        mkdir -p "$BACKUP_DEST" || error "Cannot create backup destination"
    fi
    
    # Check disk space
    local required=0
    for source in "${BACKUP_SOURCES[@]}"; do
        local size=$(du -sb "$source" | cut -f1)
        required=$((required + size))
    done
    
    local available=$(df "$BACKUP_DEST" | tail -1 | awk '{print $4}')
    available=$((available * 1024))
    
    if [ $available -lt $required ]; then
        error "Insufficient disk space (Required: $required, Available: $available)"
    fi
    
    log "Configuration validated"
}

create_backup() {
    local timestamp=$(date +%Y%m%d_%H%M%S)
    local backup_name="backup_${timestamp}"
    local backup_file="${BACKUP_DEST}/${backup_name}.tar.gz"
    local temp_dir="/tmp/${backup_name}"
    
    log "=== Backup Started ==="
    log "Backup file: $backup_file"
    
    # Create temporary directory
    mkdir -p "$temp_dir"
    
    # Copy sources to temp directory
    for source in "${BACKUP_SOURCES[@]}"; do
        log "Backing up: $source"
        rsync -a "$source" "$temp_dir/" || {
            error "Failed to copy $source"
        }
    done
    
    # Create archive
    log "Creating archive..."
    tar -czf "$backup_file" -C "$temp_dir" . || {
        error "Failed to create archive"
    }
    
    # Encrypt if enabled
    if $ENCRYPTION_ENABLED; then
        log "Encrypting backup..."
        openssl enc -aes-256-cbc -salt \
            -in "$backup_file" \
            -out "${backup_file}.enc" \
            -pass pass:"$ENCRYPTION_KEY" || {
            error "Encryption failed"
        }
        
        rm -f "$backup_file"
        backup_file="${backup_file}.enc"
    fi
    
    # Calculate checksum
    local checksum=$(sha256sum "$backup_file" | cut -d' ' -f1)
    echo "$checksum" > "${backup_file}.sha256"
    
    # Get file size
    local size=$(du -h "$backup_file" | cut -f1)
    
    log "Backup created successfully"
    log "Size: $size"
    log "Checksum: $checksum"
    
    # Cleanup temp directory
    rm -rf "$temp_dir"
    
    echo "$backup_file"
}

upload_to_remote() {
    local backup_file=$1
    
    if ! $REMOTE_BACKUP_ENABLED; then
        return 0
    fi
    
    log "Uploading to remote server..."
    
    rsync -avz --progress \
        "$backup_file" \
        "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}/" || {
        log "WARNING: Remote upload failed"
        return 1
    }
    
    # Upload checksum too
    rsync -avz \
        "${backup_file}.sha256" \
        "${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_PATH}/" || {
        log "WARNING: Checksum upload failed"
    }
    
    log "Upload completed"
}

rotate_backups() {
    log "Rotating old backups (keeping last $RETENTION_DAYS days)..."
    
    local count=0
    find "$BACKUP_DEST" -name "backup_*.tar.gz*" -mtime +$RETENTION_DAYS | while read file; do
        log "Deleting: $file"
        rm -f "$file" "${file}.sha256"
        ((count++))
    done
    
    log "Deleted $count old backup(s)"
}

verify_backup() {
    local backup_file=$1
    
    log "Verifying backup integrity..."
    
    # Verify checksum
    if [ -f "${backup_file}.sha256" ]; then
        local stored_sum=$(cat "${backup_file}.sha256")
        local actual_sum=$(sha256sum "$backup_file" | cut -d' ' -f1)
        
        if [ "$stored_sum" != "$actual_sum" ]; then
            error "Checksum verification failed!"
        fi
        
        log "Checksum verified"
    fi
    
    # Test archive integrity
    if [[ $backup_file =~ \.enc$ ]]; then
        # Decrypt and test
        openssl enc -aes-256-cbc -d \
            -in "$backup_file" \
            -pass pass:"$ENCRYPTION_KEY" | \
            tar -tzf - >/dev/null || {
            error "Archive integrity check failed"
        }
    else
        tar -tzf "$backup_file" >/dev/null || {
            error "Archive integrity check failed"
        }
    fi
    
    log "Backup verified successfully"
}

#------------------------------------------------------------------------------
# MAIN
#------------------------------------------------------------------------------

main() {
    log "=== Backup System Started ==="
    
    # Load configuration
    load_config || error "Failed to load configuration"
    
    # Validate
    validate_config
    
    # Create backup
    local backup_file=$(create_backup)
    
    # Verify backup
    verify_backup "$backup_file"
    
    # Upload to remote
    upload_to_remote "$backup_file"
    
    # Rotate old backups
    rotate_backups
    
    log "=== Backup Completed Successfully ==="
    
    # Send success notification
    send_notification \
        "Backup Successful" \
        "Backup completed at $(date)\nFile: $backup_file"
}

# Error handler
trap 'send_notification "Backup Failed" "Backup failed at line $LINENO"' ERR

# Run main function
main "$@"
```

### **Configuration File (config.conf):**

```bash
# Backup sources (directories to backup)
BACKUP_SOURCES=(
    "/home/user/documents"
    "/var/www"
    "/etc"
)

# Backup destination
BACKUP_DEST="/backups"

# Retention (days)
RETENTION_DAYS=30

# Encryption
ENCRYPTION_ENABLED=true
ENCRYPTION_KEY="your-secure-key-here"

# Remote backup
REMOTE_BACKUP_ENABLED=true
REMOTE_USER="backup"
REMOTE_HOST="backup.example.com"
REMOTE_PATH="/backups"

# Notifications
NOTIFY_EMAIL="admin@example.com"

# Logging
LOG_FILE="/var/log/backup.log"
```

---

## 📊 Project 2: Log Analyzer

### **Advanced Log Analysis Tool:**

```bash
#!/bin/bash

###############################################################################
# Log Analyzer
# Analyzes system and application logs, generates reports, detects anomalies
###############################################################################

set -euo pipefail

#------------------------------------------------------------------------------
# CONFIGURATION
#------------------------------------------------------------------------------

LOG_FILES=(
    "/var/log/syslog"
    "/var/log/auth.log"
    "/var/log/apache2/access.log"
    "/var/log/apache2/error.log"
)

REPORT_DIR="/var/reports"
ALERT_THRESHOLD=100  # Number of errors to trigger alert

#------------------------------------------------------------------------------
# FUNCTIONS
#------------------------------------------------------------------------------

generate_summary() {
    local logfile=$1
    
    if [ ! -f "$logfile" ]; then
        echo "Log file not found: $logfile"
        return 1
    fi
    
    echo "=== $(basename "$logfile") ==="
    echo "Total lines: $(wc -l < "$logfile")"
    echo "Errors: $(grep -ci "error" "$logfile" || echo 0)"
    echo "Warnings: $(grep -ci "warn" "$logfile" || echo 0)"
    echo "Critical: $(grep -ci "critical" "$logfile" || echo 0)"
    echo ""
}

analyze_errors() {
    local logfile=$1
    
    echo "=== Top 10 Error Messages ==="
    grep -i "error" "$logfile" 2>/dev/null | \
        sed 's/.*ERROR: //' | \
        sort | uniq -c | sort -rn | head -10
    echo ""
}

analyze_ips() {
    local logfile=$1
    
    echo "=== Top 10 IP Addresses ==="
    grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" "$logfile" 2>/dev/null | \
        sort | uniq -c | sort -rn | head -10
    echo ""
}

detect_brute_force() {
    local logfile=$1
    
    echo "=== Potential Brute Force Attacks ==="
    grep "Failed password" "$logfile" 2>/dev/null | \
        grep -oE "\b([0-9]{1,3}\.){3}[0-9]{1,3}\b" | \
        sort | uniq -c | sort -rn | \
        awk '$1 > 5 {print $2, "(" $1, "attempts)"}' || echo "None detected"
    echo ""
}

analyze_time_distribution() {
    local logfile=$1
    
    echo "=== Hourly Distribution ==="
    grep -oE "[0-9]{2}:[0-9]{2}:[0-9]{2}" "$logfile" 2>/dev/null | \
        cut -d: -f1 | sort | uniq -c | \
        awk '{printf "%02d:00 - %d events\n", $2, $1}'
    echo ""
}

generate_report() {
    local report_file="$REPORT_DIR/log_report_$(date +%Y%m%d_%H%M%S).txt"
    
    mkdir -p "$REPORT_DIR"
    
    {
        echo "========================================="
        echo "Log Analysis Report"
        echo "Generated: $(date)"
        echo "========================================="
        echo ""
        
        for logfile in "${LOG_FILES[@]}"; do
            if [ -f "$logfile" ]; then
                generate_summary "$logfile"
                analyze_errors "$logfile"
                analyze_ips "$logfile"
                detect_brute_force "$logfile"
                analyze_time_distribution "$logfile"
                echo "========================================="
                echo ""
            fi
        done
        
    } > "$report_file"
    
    echo "Report generated: $report_file"
    cat "$report_file"
}

# Main execution
generate_report
```

---

## 🚀 Project 3: Deployment Automation

### **Zero-Downtime Deployment:**

```bash
#!/bin/bash

###############################################################################
# Zero-Downtime Deployment Script
# Blue-Green deployment with automated rollback
###############################################################################

set -euo pipefail

APP_NAME="myapp"
DEPLOY_USER="deploy"
SERVERS=("server1.example.com" "server2.example.com")
BLUE_PORT=8080
GREEN_PORT=8081
HEALTH_ENDPOINT="/health"
TIMEOUT=300

log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*"
}

get_active_port() {
    local server=$1
    # Determine which port is currently active
    ssh "$DEPLOY_USER@$server" "curl -sf localhost:$BLUE_PORT$HEALTH_ENDPOINT" &>/dev/null && echo "$BLUE_PORT" || echo "$GREEN_PORT"
}

get_inactive_port() {
    local active_port=$1
    [ "$active_port" = "$BLUE_PORT" ] && echo "$GREEN_PORT" || echo "$BLUE_PORT"
}

deploy_to_server() {
    local server=$1
    local port=$2
    local version=$3
    
    log "Deploying version $version to $server:$port..."
    
    ssh "$DEPLOY_USER@$server" << EOF
        cd /opt/$APP_NAME
        
        # Stop inactive instance
        pm2 stop $APP_NAME-$port || true
        
        # Pull new version
        git fetch
        git checkout $version
        
        # Install dependencies
        npm ci --production
        
        # Start on inactive port
        PORT=$port pm2 start ecosystem.config.js --name $APP_NAME-$port
        
        # Wait for startup
        sleep 10
EOF
    
    log "Deployment completed on $server:$port"
}

health_check() {
    local server=$1
    local port=$2
    local attempt=1
    local max_attempts=30
    
    log "Running health check on $server:$port..."
    
    while [ $attempt -le $max_attempts ]; do
        if curl -sf "http://$server:$port$HEALTH_ENDPOINT" >/dev/null; then
            log "Health check passed"
            return 0
        fi
        
        log "Waiting for application... (attempt $attempt/$max_attempts)"
        sleep 2
        ((attempt++))
    done
    
    log "Health check failed"
    return 1
}

switch_traffic() {
    local server=$1
    local new_port=$2
    
    log "Switching traffic to $server:$new_port..."
    
    ssh "$DEPLOY_USER@$server" << EOF
        # Update nginx configuration
        sed -i "s/localhost:[0-9]*/localhost:$new_port/" /etc/nginx/sites-enabled/$APP_NAME
        
        # Reload nginx
        sudo nginx -t && sudo systemctl reload nginx
EOF
    
    log "Traffic switched"
}

rollback() {
    local server=$1
    local old_port=$2
    
    log "Rolling back on $server..."
    
    switch_traffic "$server" "$old_port"
    
    log "Rollback completed"
}

deploy() {
    local version=${1:-main}
    
    log "=== Starting Deployment ==="
    log "Version: $version"
    
    for server in "${SERVERS[@]}"; do
        local active_port=$(get_active_port "$server")
        local inactive_port=$(get_inactive_port "$active_port")
        
        log "Server: $server (Active: $active_port, Target: $inactive_port)"
        
        # Deploy to inactive port
        if ! deploy_to_server "$server" "$inactive_port" "$version"; then
            log "Deployment failed on $server"
            continue
        fi
        
        # Health check
        if ! health_check "$server" "$inactive_port"; then
            log "Health check failed, skipping traffic switch"
            continue
        fi
        
        # Switch traffic
        switch_traffic "$server" "$inactive_port"
        
        # Final health check
        sleep 5
        if ! health_check "$server" "$inactive_port"; then
            log "Post-switch health check failed, rolling back"
            rollback "$server" "$active_port"
            continue
        fi
        
        # Stop old instance
        ssh "$DEPLOY_USER@$server" "pm2 stop $APP_NAME-$active_port"
        
        log "Deployment successful on $server"
    done
    
    log "=== Deployment Completed ==="
}

# Run deployment
deploy "$@"
```

---

## 🔍 Project 4: System Health Monitor

```bash
#!/bin/bash

###############################################################################
# System Health Monitor
# Monitors CPU, memory, disk, services, and sends alerts
###############################################################################

set -euo pipefail

# Thresholds
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80

# Services to monitor
SERVICES=("nginx" "mysql" "redis")

# Alert configuration
ALERT_EMAIL="admin@example.com"
SLACK_WEBHOOK="https://hooks.slack.com/services/YOUR/WEBHOOK"

alert() {
    local severity=$1
    local message=$2
    
    # Email alert
    echo "$message" | mail -s "[$severity] System Alert" "$ALERT_EMAIL"
    
    # Slack alert
    curl -X POST "$SLACK_WEBHOOK" \
        -H "Content-Type: application/json" \
        -d "{\"text\":\"[$severity] $message\"}"
}

check_cpu() {
    local usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{print 100 - $1}')
    local usage_int=${usage%.*}
    
    if [ "$usage_int" -gt "$CPU_THRESHOLD" ]; then
        alert "WARNING" "High CPU usage: ${usage}%"
    fi
}

check_memory() {
    local usage=$(free | grep Mem | awk '{printf "%.0f", $3/$2 * 100}')
    
    if [ "$usage" -gt "$MEMORY_THRESHOLD" ]; then
        alert "WARNING" "High memory usage: ${usage}%"
    fi
}

check_disk() {
    df -h | tail -n +2 | while read line; do
        local usage=$(echo "$line" | awk '{print $5}' | tr -d '%')
        local mount=$(echo "$line" | awk '{print $6}')
        
        if [ "$usage" -gt "$DISK_THRESHOLD" ]; then
            alert "WARNING" "High disk usage on $mount: ${usage}%"
        fi
    done
}

check_services() {
    for service in "${SERVICES[@]}"; do
        if ! systemctl is-active "$service" &>/dev/null; then
            alert "CRITICAL" "Service $service is down"
            
            # Auto-restart
            systemctl start "$service"
            
            if systemctl is-active "$service" &>/dev/null; then
                alert "INFO" "Service $service auto-restarted"
            fi
        fi
    done
}

# Run all checks
check_cpu
check_memory
check_disk
check_services
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's a blue-green deployment?**
   <details>
   <summary>Answer</summary>
   Deploy to inactive environment, test, then switch traffic
   </details>

2. **Why verify backups?**
   <details>
   <summary>Answer</summary>
   Ensure integrity and ability to restore
   </details>

3. **What should you monitor in production?**
   <details>
   <summary>Answer</summary>
   CPU, memory, disk, services, logs, response times
   </details>

4. **How to handle deployment failures?**
   <details>
   <summary>Answer</summary>
   Automated rollback to previous version
   </details>

5. **Why rotate logs and backups?**
   <details>
   <summary>Answer</summary>
   Prevent disk space exhaustion
   </details>

---

## 📝 Key Takeaways

✅ **Always validate before operations**  
✅ **Implement health checks**  
✅ **Automate rollback on failures**  
✅ **Monitor and alert**  
✅ **Log everything**  
✅ **Verify backups**  
✅ **Use configuration files**  
✅ **Implement retry logic**  

---

## 🚀 Next Steps

You have production-ready scripts!

**Next lesson:** [28 - Portability and Compatibility](28-portability-compatibility.md) - Cross-platform scripts

---

## 💡 Pro Tips

**Make scripts idempotent:**
Running multiple times = same result

**Always have rollback:**
Every deployment needs an undo

**Test before production:**
Test in staging environment first

Real-world experience = expertise! 🏆
