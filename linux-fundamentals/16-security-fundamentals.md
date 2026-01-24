---
render_with_liquid: false
---
# 16 - Security Fundamentals

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- User and group management
- sudo and privilege escalation
- SSH key authentication
- File permissions security
- Basic hardening practices
- Firewall basics

---

## 🔐 Why Linux Security Matters for DevOps

```
DevOps Reality:
✅ Managing production servers
✅ Protecting customer data
✅ Preventing unauthorized access
✅ Compliance requirements
✅ Incident response

One security mistake = Company breach
Security is NOT optional!
```

---

## 👥 User Management

### **Creating Users**

```bash
# Create user
sudo useradd username

# Create with home directory
sudo useradd -m username

# Create with specific shell
sudo useradd -m -s /bin/bash username

# Create with custom home directory
sudo useradd -m -d /home/custom username

# Set password
sudo passwd username

# Create user with all options
sudo useradd -m -s /bin/bash -c "Full Name" -G sudo username
```

### **Managing Users**

```bash
# Modify user
sudo usermod -s /bin/zsh username     # Change shell
sudo usermod -d /new/home username    # Change home
sudo usermod -l newname oldname       # Rename user

# Lock/unlock user
sudo usermod -L username    # Lock
sudo usermod -U username    # Unlock

# Delete user
sudo userdel username            # Keep home directory
sudo userdel -r username         # Remove home directory

# View user info
id username
finger username
cat /etc/passwd | grep username
```

### **User Information**

```bash
# Current user
whoami

# Current user details
id

# All logged-in users
who
w

# User's groups
groups username

# Last login
last username
lastlog
```

---

## 👤 Group Management

### **Creating Groups**

```bash
# Create group
sudo groupadd developers

# Create with specific GID
sudo groupadd -g 1050 developers

# Add user to group
sudo usermod -aG groupname username

# Add user to multiple groups
sudo usermod -aG group1,group2,group3 username

# Remove user from group
sudo gpasswd -d username groupname
```

### **Managing Groups**

```bash
# View all groups
cat /etc/group

# View group members
getent group groupname

# Delete group
sudo groupdel groupname

# Change group name
sudo groupmod -n newname oldname
```

---

## 🔑 sudo - Superuser Do

### **Understanding sudo**

```bash
sudo = Execute command as root

Why sudo instead of root login?
✅ Audit trail (logs who did what)
✅ Limited time elevation
✅ Granular permissions
✅ Safer than root login
```

### **Using sudo**

```bash
# Run single command as root
sudo command

# Examples:
sudo apt update
sudo systemctl restart nginx
sudo nano /etc/hosts

# Run command as specific user
sudo -u username command
sudo -u www-data ls /var/www

# Become root shell
sudo -i                    # Login shell
sudo -s                    # Current shell

# Run previous command with sudo
sudo !!

# Edit file safely
sudoedit /etc/config.txt
```

### **Configuring sudo**

```bash
# Edit sudo configuration (ALWAYS use visudo!)
sudo visudo

# Add user to sudo group (Ubuntu/Debian)
sudo usermod -aG sudo username

# Add user to wheel group (CentOS/RHEL)
sudo usermod -aG wheel username

# Grant sudo without password (careful!)
username ALL=(ALL) NOPASSWD: ALL

# Grant specific commands only
username ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx

# View sudo access
sudo -l
```

---

## 🔐 SSH Security

### **Password vs Key Authentication**

```
Password Authentication:
❌ Can be brute-forced
❌ Easy to guess/steal
❌ Phishing vulnerable

Key Authentication:
✅ Cryptographically secure
✅ No password to guess
✅ Can't be easily stolen
✅ Can use passphrase

Always use SSH keys!
```

### **Setting Up SSH Keys**

**On your local machine:**

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "your@email.com"

# Or RSA (more compatible)
ssh-keygen -t rsa -b 4096 -C "your@email.com"

# Output:
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:

# Keys created:
~/.ssh/id_ed25519       # Private key (KEEP SECRET!)
~/.ssh/id_ed25519.pub   # Public key (can share)
```

**Copy key to server:**

```bash
# Method 1: Automatic (easiest)
ssh-copy-id user@server

# Method 2: Manual
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Method 3: Really manual
# Copy content of ~/.ssh/id_ed25519.pub
# On server: nano ~/.ssh/authorized_keys
# Paste the public key
# Save and exit
```

**Test SSH key:**

```bash
# Should login without password
ssh user@server

# If prompted for password, check:
# 1. Key was copied correctly
# 2. Permissions are correct (see below)
```

### **SSH Key Permissions**

```bash
# On server:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# On local machine:
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

### **Hardening SSH**

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Recommended settings:
PermitRootLogin no              # Disable root login
PasswordAuthentication no       # Require keys
PubkeyAuthentication yes        # Enable key auth
Port 2222                       # Change default port (optional)
Protocol 2                      # Use SSH protocol 2
MaxAuthTries 3                  # Limit login attempts
ClientAliveInterval 300         # Keep connection alive
ClientAliveCountMax 2           # Disconnect idle sessions

# Restart SSH
sudo systemctl restart sshd     # CentOS
sudo systemctl restart ssh      # Ubuntu
```

---

## 🛡️ File Permission Security

### **Secure Permission Patterns**

```bash
# Private files
chmod 600 file              # User can read/write
chown user:user file

# Private directories
chmod 700 directory         # User can access
chown user:user directory

# Shared files (read-only)
chmod 644 file              # Everyone can read
chown user:group file

# Shared directories
chmod 755 directory         # Everyone can access
chown user:group directory

# Web files
chmod 644 file.html
chmod 755 /var/www
chown www-data:www-data file.html

# Scripts
chmod 750 script.sh         # Owner and group can execute
chmod 755 script.sh         # Everyone can execute
```

### **Security Best Practices**

```bash
# ❌ NEVER do this:
chmod 777 file              # Anyone can do anything!
chmod -R 777 /              # DISASTER!

# ✅ Always use minimal permissions:
chmod 600 ~/.ssh/id_rsa     # Only you can read private key
chmod 644 /etc/passwd       # Everyone can read, only root can write
chmod 755 /usr/bin/script   # Everyone can execute, only root can modify

# Find dangerous permissions
find / -type f -perm 0777 2>/dev/null
find / -type f -perm 0666 2>/dev/null
```

---

## 🔥 Firewall Basics

### **UFW (Ubuntu Firewall)**

```bash
# Install (Ubuntu/Debian)
sudo apt install ufw

# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Check status
sudo ufw status
sudo ufw status verbose

# Allow port
sudo ufw allow 22              # SSH
sudo ufw allow 80              # HTTP
sudo ufw allow 443             # HTTPS
sudo ufw allow 3306            # MySQL

# Allow specific service
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow specific port from specific IP
sudo ufw allow from 192.168.1.100 to any port 22

# Deny port
sudo ufw deny 23               # Block telnet

# Delete rule
sudo ufw delete allow 80

# Reset firewall
sudo ufw reset

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### **firewalld (CentOS/RHEL)**

```bash
# Start firewall
sudo systemctl start firewalld
sudo systemctl enable firewalld

# Check status
sudo firewall-cmd --state
sudo firewall-cmd --list-all

# Add service
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --add-service=https --permanent

# Add port
sudo firewall-cmd --add-port=8080/tcp --permanent

# Remove service
sudo firewall-cmd --remove-service=http --permanent

# Reload firewall
sudo firewall-cmd --reload

# List allowed
sudo firewall-cmd --list-services
sudo firewall-cmd --list-ports
```

---

## 🔒 Security Hardening Checklist

### **Basic Hardening**

```bash
# 1. Keep system updated
sudo apt update && sudo apt upgrade -y

# 2. Disable root login
sudo nano /etc/ssh/sshd_config
# Set: PermitRootLogin no

# 3. Use SSH keys only
# Set: PasswordAuthentication no

# 4. Enable firewall
sudo ufw enable
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443

# 5. Create non-root user with sudo
sudo useradd -m -s /bin/bash admin
sudo usermod -aG sudo admin
sudo passwd admin

# 6. Disable unnecessary services
sudo systemctl disable service-name

# 7. Set up fail2ban (blocks brute force)
sudo apt install fail2ban
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

# 8. Regular backups
# (Set up automated backups)

# 9. Monitor logs
sudo journalctl -f
tail -f /var/log/auth.log

# 10. Remove old/unused software
sudo apt autoremove
```

---

## 🚨 Security Monitoring

### **Check Failed Logins**

```bash
# Failed SSH attempts
sudo grep "Failed password" /var/log/auth.log

# Last logins
lastlog

# Login history
last

# Currently logged in
who
w
```

### **Check Open Ports**

```bash
# What's listening
sudo ss -tlnp
sudo netstat -tlnp

# Specific port
sudo ss -tlnp | grep :22
```

### **Check Running Processes**

```bash
# All processes
ps aux

# Suspicious processes
ps aux --sort=-%cpu | head -10
ps aux --sort=-%mem | head -10
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you create a user with home directory?**
   <details>
   <summary>Answer</summary>
   sudo useradd -m username
   </details>

2. **How do you add a user to sudo group?**
   <details>
   <summary>Answer</summary>
   sudo usermod -aG sudo username
   </details>

3. **How do you generate an SSH key?**
   <details>
   <summary>Answer</summary>
   ssh-keygen -t ed25519 -C "email@example.com"
   </details>

4. **What permissions should private SSH key have?**
   <details>
   <summary>Answer</summary>
   600 (chmod 600 ~/.ssh/id_rsa)
   </details>

5. **How do you enable the firewall on Ubuntu?**
   <details>
   <summary>Answer</summary>
   sudo ufw enable
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Create a new user
sudo useradd -m -s /bin/bash testuser

# 2. Set password
sudo passwd testuser

# 3. Add to sudo group
sudo usermod -aG sudo testuser

# 4. Create SSH key
ssh-keygen -t ed25519 -C "test@example.com"

# 5. Check permissions
ls -la ~/.ssh/

# 6. Enable firewall
sudo ufw enable

# 7. Allow SSH
sudo ufw allow 22

# 8. Check firewall status
sudo ufw status

# 9. View logged-in users
who

# 10. Check sudo access
sudo -l

# 11. View authentication logs
sudo tail -20 /var/log/auth.log

# 12. Clean up
sudo userdel -r testuser
```

---

## 📝 Key Takeaways

✅ **sudo useradd -m user** - create user  
✅ **sudo usermod -aG sudo user** - add to sudo  
✅ **ssh-keygen** - create SSH key  
✅ **ssh-copy-id** - copy key to server  
✅ **chmod 600** - secure private files  
✅ **sudo ufw enable** - enable firewall  
✅ **sudo ufw allow PORT** - open port  
✅ **Disable root login** - security best practice  
✅ **Use SSH keys** - no passwords  
✅ **Keep system updated** - patch vulnerabilities  

---

## 🚀 Next Steps

You now understand Linux security fundamentals!

**Next lesson:** [17-troubleshooting.md](17-troubleshooting.md) - Troubleshooting common issues

---

## 💡 Pro Tips

**Quick security audit:**
```bash
# Check for users with UID 0 (root)
awk -F: '($3 == "0") {print}' /etc/passwd

# Find files with SUID bit (potential risk)
find / -perm -4000 2>/dev/null

# Check for empty passwords
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# List all sudo users
grep '^sudo:.*$' /etc/group | cut -d: -f4
getent group sudo
```

**SSH config shortcuts:**
```bash
# Create ~/.ssh/config
nano ~/.ssh/config

Host myserver
    HostName 192.168.1.100
    User admin
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Now just use:
ssh myserver
```

**fail2ban protection:**
```bash
# Install
sudo apt install fail2ban

# Check status
sudo fail2ban-client status

# Check SSH jail
sudo fail2ban-client status sshd

# Unban IP
sudo fail2ban-client set sshd unbanip 192.168.1.100
```

Security is a journey, not a destination! 🔐
