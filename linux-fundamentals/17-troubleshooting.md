---
render_with_liquid: false
---
# 17 - Troubleshooting

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Systematic troubleshooting methodology
- Common issues and solutions
- Log analysis
- Performance debugging
- Network troubleshooting
- Service issues

---

## 🔍 Troubleshooting Methodology

**The Systematic Approach:**

```
1. IDENTIFY the problem
   What's broken? When did it start?

2. GATHER information
   Logs, error messages, system state

3. FORM hypothesis
   What might be causing this?

4. TEST hypothesis
   Try one thing at a time

5. DOCUMENT solution
   For future reference

Don't randomly try things!
Be systematic and methodical.
```

---

## 🚨 Common Issues & Solutions

### **1. Cannot Connect via SSH**

**Symptoms:**
```
ssh: connect to host server port 22: Connection refused
```

**Diagnosis:**
```bash
# 1. Is SSH service running?
sudo systemctl status ssh     # Ubuntu
sudo systemctl status sshd    # CentOS

# 2. Is port 22 open?
sudo ss -tlnp | grep :22

# 3. Is firewall blocking?
sudo ufw status
sudo iptables -L

# 4. Can you ping the server?
ping server-ip

# 5. Check SSH config
sudo nano /etc/ssh/sshd_config
```

**Solutions:**
```bash
# Start SSH service
sudo systemctl start ssh
sudo systemctl enable ssh

# Allow through firewall
sudo ufw allow 22

# Check for correct port
# In /etc/ssh/sshd_config:
Port 22

# Restart SSH
sudo systemctl restart ssh

# Check network connectivity
ping google.com
```

---

### **2. Disk Full**

**Symptoms:**
```
No space left on device
Write failed
Cannot create file
```

**Diagnosis:**
```bash
# 1. Check disk usage
df -h

# Output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G   19G  100M  99% /
                                  ^^^^ PROBLEM!

# 2. Find large directories
du -h / | sort -rh | head -20

# 3. Find large files
find / -type f -size +100M -exec ls -lh {} \; 2>/dev/null

# 4. Check specific directories
du -sh /var/log/*
du -sh /tmp/*
du -sh /home/*
```

**Solutions:**
```bash
# Clean package cache
sudo apt clean
sudo apt autoclean

# Remove old logs
sudo journalctl --vacuum-size=100M
sudo journalctl --vacuum-time=7d

# Find and remove old log files
sudo find /var/log -name "*.log" -mtime +30 -delete
sudo find /var/log -name "*.gz" -delete

# Clear temp files
sudo rm -rf /tmp/*

# Remove unused packages
sudo apt autoremove

# Find large files in home
du -sh ~/* | sort -rh | head -10

# Remove Docker images/containers
docker system prune -a
```

---

### **3. High CPU Usage**

**Symptoms:**
```
System slow
Applications unresponsive
High load average
```

**Diagnosis:**
```bash
# 1. Check current CPU usage
top
htop

# 2. Find CPU hogs
ps aux --sort=-%cpu | head -10

# 3. Check load average
uptime

# Output:
10:30:15 up 5 days, 2:15, 3 users, load average: 8.15, 7.20, 6.18
                                                   ^^^^ HIGH!

# 4. Check for runaway processes
ps aux | awk '$3 > 80.0'
```

**Solutions:**
```bash
# Kill problematic process
kill -15 PID        # Graceful
kill -9 PID         # Force

# Lower process priority
renice +10 PID

# Find and kill all instances
pkill -9 process-name

# Restart problematic service
sudo systemctl restart service-name

# Check for scheduled tasks
crontab -l
sudo crontab -l
```

---

### **4. High Memory Usage**

**Symptoms:**
```
System swapping
Applications killed (OOM)
Slow performance
```

**Diagnosis:**
```bash
# 1. Check memory usage
free -h

# Output:
              total     used     free   buff/cache available
Mem:           7.7G     7.2G     100M        400M      200M
                        ^^^^     ^^^^ PROBLEM!

# 2. Top memory consumers
ps aux --sort=-%mem | head -10

# 3. Check for memory leaks
# (Same process using more memory over time)
while true; do
    ps aux | grep process-name
    sleep 10
done

# 4. Check OOM killer logs
dmesg | grep -i "out of memory"
sudo grep -i "killed process" /var/log/syslog
```

**Solutions:**
```bash
# Kill memory hogs
kill -9 PID

# Restart service
sudo systemctl restart service-name

# Clear cache (safe, kernel will rebuild)
sudo sync
sudo sysctl -w vm.drop_caches=3

# Add swap space (if needed)
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make swap permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# Adjust swappiness
sudo sysctl vm.swappiness=10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
```

---

### **5. Service Won't Start**

**Symptoms:**
```
Failed to start service
Service exits immediately
systemctl status shows "failed"
```

**Diagnosis:**
```bash
# 1. Check service status
systemctl status service-name

# Look for:
Active: failed (Result: exit-code)
Process: ... (code=exited, status=1/FAILURE)

# 2. Check logs
journalctl -u service-name -n 50
journalctl -u service-name -f

# 3. Check configuration
# For nginx:
nginx -t

# For Apache:
apachectl configtest

# For MySQL:
mysqld --verbose --help

# 4. Check port conflicts
sudo ss -tlnp | grep :80

# 5. Check file permissions
ls -l /etc/nginx/nginx.conf
ls -ld /var/www

# 6. Check dependencies
systemctl list-dependencies service-name
```

**Solutions:**
```bash
# Fix configuration
sudo nano /etc/service/config

# Test configuration
service-binary -t

# Fix permissions
sudo chown user:group /path/to/file
sudo chmod 644 /path/to/file

# Kill process using port
sudo lsof -ti:80 | xargs sudo kill -9

# Restart dependencies
sudo systemctl restart dependency-service

# Reload systemd
sudo systemctl daemon-reload

# Reset failed state
sudo systemctl reset-failed

# Start service
sudo systemctl start service-name
```

---

### **6. Network Issues**

**Symptoms:**
```
Cannot connect to internet
DNS not resolving
Slow network
Connection timeouts
```

**Diagnosis:**
```bash
# 1. Check network interface
ip addr
ifconfig

# 2. Test connectivity layers
ping 127.0.0.1          # Loopback (should work)
ping 192.168.1.1        # Gateway
ping 8.8.8.8            # Internet (Google DNS)
ping google.com         # DNS resolution

# 3. Check routing
ip route
route -n

# 4. Check DNS
cat /etc/resolv.conf
nslookup google.com
dig google.com

# 5. Test ports
telnet google.com 80
nc -zv google.com 80
curl -I https://google.com

# 6. Check firewall
sudo iptables -L
sudo ufw status
```

**Solutions:**
```bash
# Restart network
sudo systemctl restart networking       # Debian/Ubuntu
sudo systemctl restart NetworkManager   # Ubuntu desktop
sudo systemctl restart network          # CentOS

# Renew DHCP
sudo dhclient -r
sudo dhclient

# Flush DNS cache
sudo systemd-resolve --flush-caches
sudo resolvectl flush-caches

# Fix DNS
sudo nano /etc/resolv.conf
nameserver 8.8.8.8
nameserver 1.1.1.1

# Test different DNS
nslookup google.com 8.8.8.8

# Disable/enable interface
sudo ip link set eth0 down
sudo ip link set eth0 up
```

---

### **7. Permission Denied**

**Symptoms:**
```
Permission denied
Operation not permitted
Cannot access file/directory
```

**Diagnosis:**
```bash
# 1. Check file permissions
ls -l file
ls -ld directory/

# 2. Check ownership
ls -l file

# 3. Check your user/groups
id
groups

# 4. Check parent directory permissions
ls -ld /path/to/
ls -ld /path/to/parent/

# 5. Check SELinux (if on CentOS)
getenforce
ls -Z file
```

**Solutions:**
```bash
# Fix file permissions
chmod 644 file          # Files
chmod 755 directory     # Directories

# Fix ownership
sudo chown user:group file
sudo chown -R user:group directory/

# Add yourself to group
sudo usermod -aG groupname $USER
# Log out and back in

# Use sudo
sudo command

# Disable SELinux temporarily (to test)
sudo setenforce 0
```

---

### **8. Command Not Found**

**Symptoms:**
```
bash: command: command not found
```

**Diagnosis:**
```bash
# 1. Is it installed?
which command
type command

# 2. Check PATH
echo $PATH

# 3. Find the binary
find /usr -name command 2>/dev/null

# 4. Check spelling
# (typo in command name?)
```

**Solutions:**
```bash
# Install package
sudo apt install package-name
sudo yum install package-name

# Add to PATH
export PATH=$PATH:/new/directory

# Use full path
/usr/local/bin/command

# Create alias
alias command='/full/path/to/command'

# Fix PATH in .bashrc
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## 📊 Log Analysis

**Key log locations:**
```bash
/var/log/syslog             General system log (Ubuntu)
/var/log/messages           General system log (CentOS)
/var/log/auth.log           Authentication log
/var/log/kern.log           Kernel log
/var/log/nginx/             Nginx logs
/var/log/apache2/           Apache logs
/var/log/mysql/             MySQL logs

# journalctl (systemd)
journalctl                  All logs
journalctl -f               Follow (like tail -f)
journalctl -u service       Service logs
journalctl -b               Boot logs
journalctl --since "1 hour ago"
```

**Analyzing logs:**
```bash
# Find errors
grep -i error /var/log/syslog
journalctl -p err

# Count occurrences
grep "404" /var/log/nginx/access.log | wc -l

# Top errors
cat /var/log/app.log | grep ERROR | sort | uniq -c | sort -rn

# Time-based search
grep "2024-01-15 14:" /var/log/app.log

# Follow multiple logs
tail -f /var/log/syslog /var/log/auth.log

# Search all logs
sudo grep -r "error" /var/log/ 2>/dev/null
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you find what's using the most CPU?**
   <details>
   <summary>Answer</summary>
   top  OR  ps aux --sort=-%cpu | head -10
   </details>

2. **How do you check disk usage?**
   <details>
   <summary>Answer</summary>
   df -h
   </details>

3. **How do you view service logs?**
   <details>
   <summary>Answer</summary>
   journalctl -u service-name
   </details>

4. **How do you test network connectivity?**
   <details>
   <summary>Answer</summary>
   ping 8.8.8.8  (and other ping tests)
   </details>

5. **How do you find large files?**
   <details>
   <summary>Answer</summary>
   find / -type f -size +100M 2>/dev/null
   </details>

---

## 🏋️ Hands-On Exercise

**Create a troubleshooting script:**

```bash
#!/bin/bash
# system-check.sh - Quick system health check

echo "=== System Health Check ==="
echo

# Disk space
echo "1. DISK SPACE:"
df -h | grep -E '^/dev'
echo

# Memory
echo "2. MEMORY USAGE:"
free -h
echo

# CPU Load
echo "3. CPU LOAD:"
uptime
echo

# Top CPU processes
echo "4. TOP CPU PROCESSES:"
ps aux --sort=-%cpu | head -6
echo

# Top Memory processes
echo "5. TOP MEMORY PROCESSES:"
ps aux --sort=-%mem | head -6
echo

# Services status
echo "6. CRITICAL SERVICES:"
for svc in ssh nginx mysql; do
    if systemctl is-active --quiet $svc 2>/dev/null; then
        echo "✓ $svc is running"
    else
        echo "✗ $svc is NOT running"
    fi
done
echo

# Recent errors
echo "7. RECENT ERRORS (last 10):"
journalctl -p err -n 10 --no-pager
echo

# Network connectivity
echo "8. NETWORK:"
if ping -c 1 8.8.8.8 > /dev/null 2>&1; then
    echo "✓ Internet connection OK"
else
    echo "✗ No internet connection"
fi

echo
echo "=== Check Complete ==="
```

---

## 📝 Key Takeaways

✅ **Be systematic** - don't randomly try things  
✅ **Check logs first** - journalctl, /var/log/  
✅ **df -h** - check disk space  
✅ **top/htop** - check CPU/memory  
✅ **systemctl status** - check services  
✅ **ping** - test network connectivity  
✅ **ps aux** - find problematic processes  
✅ **One change at a time** - isolate the issue  
✅ **Document solutions** - for next time  

---

## 🚀 Next Steps

You can now troubleshoot common Linux issues!

**Final lesson:** [18-cheat-sheet.md](18-cheat-sheet.md) - Quick reference for all commands

---

## 💡 Pro Tips

**Emergency commands:**
```bash
# System is unresponsive - SSH still works

# Find what's using resources
top
htop

# Kill top CPU process
kill -9 $(ps aux --sort=-%cpu | awk 'NR==2{print $2}')

# Clear cache
sudo sync && sudo sysctl -w vm.drop_caches=3

# Reboot (last resort)
sudo reboot

# Force reboot if system frozen
sudo reboot -f
```

**Monitoring in real-time:**
```bash
# Watch disk usage
watch -n 1 df -h

# Watch memory
watch -n 1 free -h

# Watch processes
watch -n 1 'ps aux --sort=-%cpu | head -10'

# Combined monitoring
# Terminal 1:
htop

# Terminal 2:
journalctl -f

# Terminal 3:
tail -f /var/log/app.log
```

**Create helpful aliases:**
```bash
# Add to ~/.bashrc
alias checkdisk='df -h | grep -E "^/dev"'
alias checkmem='free -h'
alias checkload='uptime'
alias checklogs='sudo journalctl -p err -n 20'
alias checkports='sudo ss -tlnp'

source ~/.bashrc
```

Troubleshooting = Essential DevOps skill! 🔍
