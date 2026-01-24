---
render_with_liquid: false
---
# 10 - System Services

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What system services are
- Managing services with systemctl
- Viewing logs with journalctl
- Creating custom services
- Service troubleshooting

---

## 🎛️ What are System Services?

**Real-World Analogy:**
```
Service = Background worker

Restaurant:
- Chef in kitchen (nginx service)
- Dishwasher in back (database service)
- Security guard outside (firewall service)

They work continuously in background
Start when restaurant opens (boot)
Don't need customer interaction
```

**Technical Definition:**
```
Service (Daemon) = Program running in background

Examples:
nginx          Web server
mysql          Database
ssh            Remote access
docker         Container runtime
cron           Task scheduler

Characteristics:
✅ Starts automatically at boot
✅ Runs in background (no terminal)
✅ Usually runs as system user
✅ Managed by systemd (on modern Linux)
```

---

## 🔧 systemd - Service Manager

**systemd = System and Service Manager**

```
systemd:
- Starts system services
- Manages service dependencies
- Handles logging
- Controls service state

Command: systemctl
```

---

## 📋 Basic Service Management

### **systemctl - Control Services**

```bash
# Start a service
sudo systemctl start nginx

# Stop a service
sudo systemctl stop nginx

# Restart a service (stop + start)
sudo systemctl restart nginx

# Reload configuration (without full restart)
sudo systemctl reload nginx

# Restart if possible, otherwise reload
sudo systemctl reload-or-restart nginx

# Check service status
systemctl status nginx

# Enable service (start at boot)
sudo systemctl enable nginx

# Disable service (don't start at boot)
sudo systemctl disable nginx

# Enable and start immediately
sudo systemctl enable --now nginx

# Disable and stop immediately
sudo systemctl disable --now nginx

# Check if service is enabled
systemctl is-enabled nginx

# Check if service is active
systemctl is-active nginx

# Check if service failed
systemctl is-failed nginx
```

---

### **Understanding Service Status**

```bash
$ systemctl status nginx

● nginx.service - A high performance web server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2024-01-15 10:30:00 UTC; 2h 15min ago
       Docs: man:nginx(8)
    Process: 1234 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1235 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1236 (nginx)
      Tasks: 5 (limit: 4915)
     Memory: 12.5M
        CPU: 1.234s
     CGroup: /system.slice/nginx.service
             ├─1236 nginx: master process /usr/sbin/nginx
             └─1237 nginx: worker process

Jan 15 10:30:00 server systemd[1]: Starting nginx.service...
Jan 15 10:30:00 server systemd[1]: Started nginx.service.

Breakdown:
Loaded:     Service file found, enabled at boot
Active:     Currently running, started 2h 15min ago
Main PID:   Process ID of main service
Tasks:      Number of running processes
Memory:     RAM usage
Docs:       Documentation location
Logs:       Recent log entries
```

**Service states:**
```
active (running)    Service is running ✅
active (exited)     One-time service completed ✅
active (waiting)    Service waiting for event ⏳
inactive (dead)     Service stopped 🛑
failed              Service crashed ❌
```

---

## 📝 Listing Services

```bash
# List all services
systemctl list-units --type=service

# List running services only
systemctl list-units --type=service --state=running

# List failed services
systemctl list-units --type=service --state=failed

# List all installed service files
systemctl list-unit-files --type=service

# List enabled services
systemctl list-unit-files --type=service --state=enabled

# List disabled services
systemctl list-unit-files --type=service --state=disabled

# Filter by pattern
systemctl list-units --type=service | grep nginx
```

---

## 📊 Common Services

### **Web Servers**

```bash
# Nginx
sudo systemctl start nginx
sudo systemctl enable nginx
systemctl status nginx

# Apache
sudo systemctl start apache2      # Ubuntu
sudo systemctl start httpd        # CentOS
systemctl status apache2
```

### **Databases**

```bash
# MySQL
sudo systemctl start mysql
sudo systemctl enable mysql
systemctl status mysql

# PostgreSQL
sudo systemctl start postgresql
systemctl status postgresql

# MongoDB
sudo systemctl start mongod
systemctl status mongod

# Redis
sudo systemctl start redis
systemctl status redis
```

### **System Services**

```bash
# SSH Server
sudo systemctl start ssh          # Ubuntu
sudo systemctl start sshd         # CentOS
systemctl status ssh

# Firewall
sudo systemctl start ufw          # Ubuntu
sudo systemctl start firewalld    # CentOS
systemctl status ufw

# Cron (scheduled tasks)
systemctl status cron

# Docker
sudo systemctl start docker
systemctl status docker
```

---

## 📋 Viewing Logs

### **journalctl - SystemD Logs**

**View service logs:**

```bash
# All logs
journalctl

# Logs for specific service
journalctl -u nginx
journalctl -u mysql

# Follow logs (like tail -f)
journalctl -u nginx -f

# Last 50 lines
journalctl -u nginx -n 50

# Logs since boot
journalctl -b

# Logs from previous boot
journalctl -b -1

# Logs since specific time
journalctl --since "2024-01-15 10:00:00"
journalctl --since "1 hour ago"
journalctl --since "yesterday"
journalctl --since "10 min ago"

# Logs until specific time
journalctl --until "2024-01-15 11:00:00"

# Time range
journalctl --since "2024-01-15 10:00" --until "2024-01-15 11:00"

# Logs with specific priority
journalctl -p err           # Errors only
journalctl -p warning       # Warnings and above
journalctl -p debug         # All including debug

# Reverse order (newest first)
journalctl -r

# Specific unit + time range
journalctl -u nginx --since "1 hour ago"

# Output as JSON
journalctl -u nginx -o json

# Kernel messages only
journalctl -k
```

**Practical examples:**
```bash
# Check why service failed
journalctl -u nginx -n 100

# Monitor service in real-time
journalctl -u nginx -f

# Find errors in last hour
journalctl -u nginx --since "1 hour ago" -p err

# Check service after restart
sudo systemctl restart nginx
journalctl -u nginx -n 20
```

---

### **Traditional Logs**

```bash
# System logs location
ls /var/log/

# Common log files:
/var/log/syslog         General system log (Debian/Ubuntu)
/var/log/messages       General system log (CentOS/RHEL)
/var/log/auth.log       Authentication log
/var/log/kern.log       Kernel log
/var/log/boot.log       Boot log
/var/log/dmesg          Hardware messages

# Application logs:
/var/log/nginx/         Nginx logs
/var/log/mysql/         MySQL logs
/var/log/apache2/       Apache logs

# View logs
tail -f /var/log/syslog
tail -n 100 /var/log/auth.log
less /var/log/syslog
```

---

## 🛠️ Creating Custom Services

**Create a simple service:**

```bash
# 1. Create your script
sudo nano /usr/local/bin/my-app.sh

#!/bin/bash
while true; do
    echo "My app is running: $(date)"
    sleep 60
done

# 2. Make it executable
sudo chmod +x /usr/local/bin/my-app.sh

# 3. Create service file
sudo nano /etc/systemd/system/my-app.service

[Unit]
Description=My Custom Application
After=network.target

[Service]
Type=simple
User=www-data
WorkingDirectory=/opt/my-app
ExecStart=/usr/local/bin/my-app.sh
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

# 4. Reload systemd
sudo systemctl daemon-reload

# 5. Start and enable service
sudo systemctl start my-app
sudo systemctl enable my-app

# 6. Check status
systemctl status my-app

# 7. View logs
journalctl -u my-app -f
```

**Service file sections explained:**

```ini
[Unit]
Description=What this service does
After=Start after these services
Requires=Required dependencies
Wants=Optional dependencies

[Service]
Type=simple            # Service runs in foreground
Type=forking           # Service forks to background
Type=oneshot           # Runs once and exits

User=username          # Run as this user
Group=groupname        # Run as this group
WorkingDirectory=/path # Start in this directory
ExecStart=/path/cmd    # Command to start
ExecStop=/path/cmd     # Command to stop
ExecReload=/path/cmd   # Command to reload

Restart=always         # Always restart if crashes
Restart=on-failure     # Restart only on failure
RestartSec=10          # Wait 10s before restart

[Install]
WantedBy=multi-user.target    # Start at boot
```

**Common service examples:**

```bash
# Node.js app
[Service]
ExecStart=/usr/bin/node /opt/app/server.js

# Python app
[Service]
ExecStart=/usr/bin/python3 /opt/app/main.py

# Custom binary
[Service]
ExecStart=/usr/local/bin/myapp

# With environment variables
[Service]
Environment="PORT=3000"
Environment="NODE_ENV=production"
ExecStart=/usr/bin/node /opt/app/server.js
```

---

## 🚨 Troubleshooting Services

### **Service Won't Start**

```bash
# 1. Check status
systemctl status nginx

# 2. Check logs
journalctl -u nginx -n 50

# 3. Test configuration
nginx -t                    # Nginx
apachectl configtest        # Apache
mysqld --verbose --help     # MySQL

# 4. Check if port is already in use
sudo ss -tlnp | grep :80

# 5. Check file permissions
ls -l /etc/nginx/nginx.conf

# 6. Check SELinux (if on CentOS)
getenforce
sudo setenforce 0    # Temporarily disable to test
```

### **Service Keeps Crashing**

```bash
# 1. Check logs
journalctl -u nginx -f

# 2. Check system resources
free -h
df -h

# 3. Increase restart delay
sudo systemctl edit nginx

[Service]
RestartSec=30

# 4. Check for conflicting services
sudo ss -tlnp | grep :80
```

### **Service Enabled But Not Starting at Boot**

```bash
# 1. Verify it's enabled
systemctl is-enabled nginx

# 2. Check dependencies
systemctl list-dependencies nginx

# 3. Re-enable service
sudo systemctl disable nginx
sudo systemctl enable nginx

# 4. Rebuild dependencies
sudo systemctl daemon-reload
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you start a service?**
   <details>
   <summary>Answer</summary>
   sudo systemctl start service-name
   </details>

2. **How do you make a service start at boot?**
   <details>
   <summary>Answer</summary>
   sudo systemctl enable service-name
   </details>

3. **How do you check if a service is running?**
   <details>
   <summary>Answer</summary>
   systemctl status service-name
   </details>

4. **How do you view logs for a service?**
   <details>
   <summary>Answer</summary>
   journalctl -u service-name
   </details>

5. **How do you restart a service?**
   <details>
   <summary>Answer</summary>
   sudo systemctl restart service-name
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Install nginx (if not installed)
sudo apt install nginx

# 2. Check status
systemctl status nginx

# 3. Stop nginx
sudo systemctl stop nginx

# 4. Verify it's stopped
systemctl status nginx

# 5. Start nginx
sudo systemctl start nginx

# 6. Enable it (start at boot)
sudo systemctl enable nginx

# 7. Check if enabled
systemctl is-enabled nginx

# 8. Restart nginx
sudo systemctl restart nginx

# 9. View logs
journalctl -u nginx -n 20

# 10. Follow logs (Ctrl+C to stop)
journalctl -u nginx -f

# 11. List all running services
systemctl list-units --type=service --state=running
```

---

## 📝 Key Takeaways

✅ **sudo systemctl start service** - start service  
✅ **sudo systemctl stop service** - stop service  
✅ **sudo systemctl restart service** - restart service  
✅ **sudo systemctl enable service** - start at boot  
✅ **systemctl status service** - check status  
✅ **journalctl -u service** - view logs  
✅ **journalctl -f** - follow logs in real-time  
✅ **systemctl list-units --type=service** - list services  

---

## 🚀 Next Steps

You can now manage system services like a pro!

**Next lesson:** [11-environment-variables.md](11-environment-variables.md) - Environment variables and shell configuration

---

## 💡 Pro Tips

**Quick service management:**
```bash
# Start, enable, and check in one go
sudo systemctl enable --now nginx && systemctl status nginx

# Restart and check logs
sudo systemctl restart nginx && journalctl -u nginx -n 20

# Check multiple services at once
systemctl status nginx mysql docker
```

**Service dependencies:**
```bash
# See what depends on a service
systemctl list-dependencies nginx

# See what a service depends on
systemctl list-dependencies nginx --reverse
```

**Emergency troubleshooting:**
```bash
# If service won't stop
sudo systemctl kill nginx
sudo systemctl kill -s SIGKILL nginx

# Force reload systemd
sudo systemctl daemon-reexec

# Reset failed state
sudo systemctl reset-failed
```

**Monitoring:**
```bash
# Watch service status
watch -n 1 systemctl status nginx

# Alert on service failure (add to cron)
systemctl is-failed nginx && echo "nginx failed!" | mail -s "Alert" admin@example.com
```

Master systemctl = Master your servers! 🚀
