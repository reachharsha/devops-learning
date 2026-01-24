---
render_with_liquid: false
---
# 16 - Firewall Basics

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What firewalls do and why they're critical
- Linux firewall architecture (netfilter/iptables)
- How to use ufw (simple firewall)
- Basic iptables commands
- Common firewall rules for DevOps

---

## 🔥 What is a Firewall?

**Firewall = Security guard for your network**

```
Internet (Dangerous)
      │
      ▼
 ┌─────────┐
 │ FIREWALL│  ← Blocks bad traffic
 └─────────┘   ← Allows good traffic
      │
      ▼
Your Server (Protected)
```

**Real-World Analogy:**
```
Nightclub Bouncer:
- Checks ID (source IP)
- Verifies guest list (allowed ports)
- Rejects troublemakers (blocks malicious traffic)
- Lets approved people in (allows legitimate connections)

Firewall does the same for network traffic!
```

---

## 🏗️ How Firewalls Work

### Packet Filtering:
```
Incoming packet:
┌──────────────────────────────┐
│ From: 203.0.113.50           │
│ To: 10.0.1.100:22 (SSH)      │
│ Protocol: TCP                │
└──────────────────────────────┘
         │
         ▼
   ┌──────────┐
   │ Firewall │
   │  Rules:  │
   │          │
   │ Allow    │
   │ port 22  │
   │ from     │
   │ anywhere │
   └──────────┘
         │
         ▼
    ✅ ALLOWED
```

### Firewall Decisions:
```
For each packet, firewall checks:
1. Source IP address - Where is it from?
2. Destination IP - Where is it going?
3. Port number - Which service?
4. Protocol - TCP, UDP, ICMP?
5. Direction - Incoming or outgoing?

Then:
✅ ACCEPT - Let packet through
❌ DROP - Silently discard packet
🚫 REJECT - Discard and notify sender
```

---

## 🐧 Linux Firewall Architecture

### The Stack:
```
┌─────────────────────────────────┐
│ User Tools (what you use)       │
│  - ufw (simple)                 │
│  - firewalld (RHEL/CentOS)      │
│  - iptables (powerful)          │
└─────────────────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│ netfilter (kernel module)       │
│ The actual packet filtering     │
└─────────────────────────────────┘
```

**All tools configure the same underlying system (netfilter)!**

---

## 🛡️ UFW: Uncomplicated Firewall

**UFW = Easy firewall for Ubuntu/Debian**

### Installation:
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ufw

# Check status
sudo ufw status
```

### Basic Commands:

**Enable/Disable:**
```bash
# Enable firewall
sudo ufw enable

# Disable firewall
sudo ufw disable

# Check status
sudo ufw status

# Detailed status
sudo ufw status verbose
```

**Default Policies:**
```bash
# Block all incoming by default
sudo ufw default deny incoming

# Allow all outgoing by default
sudo ufw default allow outgoing

# Check defaults
sudo ufw status verbose
```

### Allowing Traffic:

**By port:**
```bash
# Allow SSH (port 22)
sudo ufw allow 22

# Same thing, explicit protocol
sudo ufw allow 22/tcp

# Allow HTTP (port 80)
sudo ufw allow 80/tcp

# Allow HTTPS (port 443)
sudo ufw allow 443/tcp

# Allow DNS (port 53, UDP and TCP)
sudo ufw allow 53
```

**By service name:**
```bash
# Allow SSH
sudo ufw allow ssh

# Allow HTTP
sudo ufw allow http

# Allow HTTPS
sudo ufw allow https

# Service names from /etc/services
```

**From specific IP:**
```bash
# Allow SSH only from specific IP
sudo ufw allow from 203.0.113.50 to any port 22

# Allow any traffic from specific IP
sudo ufw allow from 203.0.113.50

# Allow from subnet
sudo ufw allow from 192.168.1.0/24
```

**To specific interface:**
```bash
# Allow on specific network interface
sudo ufw allow in on eth0 to any port 80

# Allow SSH on internal interface only
sudo ufw allow in on eth1 to any port 22
```

### Denying Traffic:

```bash
# Deny specific port
sudo ufw deny 23

# Deny from specific IP
sudo ufw deny from 203.0.113.100

# Deny specific IP to specific port
sudo ufw deny from 203.0.113.100 to any port 22
```

### Deleting Rules:

```bash
# Delete by rule specification
sudo ufw delete allow 80

# Delete by rule number
sudo ufw status numbered
sudo ufw delete 3

# Example:
# Status numbered:
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 443/tcp                    ALLOW IN    Anywhere

# Delete rule 2:
sudo ufw delete 2
```

### Common UFW Setups:

**Web Server:**
```bash
# Reset to defaults
sudo ufw --force reset
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH (management)
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS (web traffic)
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Enable
sudo ufw enable

# Verify
sudo ufw status
```

**Database Server:**
```bash
# Default deny
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH
sudo ufw allow 22/tcp

# Allow PostgreSQL only from app servers
sudo ufw allow from 10.0.1.0/24 to any port 5432

# Enable
sudo ufw enable
```

**Docker Host:**
```bash
# UFW and Docker conflict! Docker bypasses UFW.
# Solution: Configure UFW for Docker

# Edit /etc/ufw/after.rules, add:
# *filter
# :DOCKER-USER - [0:0]
# -A DOCKER-USER -j RETURN
# COMMIT

# Or use firewalld instead of ufw
```

---

## ⚙️ iptables: The Powerful Way

**iptables = Lower-level, more powerful**

### Concepts:

**Tables:**
```
filter  - Default table, for filtering packets
nat     - Network Address Translation
mangle  - Packet alteration
raw     - Connection tracking exemptions
```

**Chains (in filter table):**
```
INPUT   - Incoming packets to this server
OUTPUT  - Outgoing packets from this server
FORWARD - Packets routed through this server
```

**Targets:**
```
ACCEPT  - Allow packet
DROP    - Silently discard packet
REJECT  - Discard and send error back
LOG     - Log packet details
```

### Basic iptables Commands:

**View rules:**
```bash
# List all rules
sudo iptables -L

# List with line numbers
sudo iptables -L --line-numbers

# List with packet counts
sudo iptables -L -v

# List specific chain
sudo iptables -L INPUT
sudo iptables -L OUTPUT
```

**Allow SSH:**
```bash
# Allow incoming SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Explanation:
# -A INPUT       - Append to INPUT chain
# -p tcp         - Protocol TCP
# --dport 22     - Destination port 22
# -j ACCEPT      - Jump to ACCEPT (allow)
```

**Allow HTTP/HTTPS:**
```bash
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

**Allow from specific IP:**
```bash
# Allow all traffic from specific IP
sudo iptables -A INPUT -s 203.0.113.50 -j ACCEPT

# Allow SSH only from specific IP
sudo iptables -A INPUT -p tcp -s 203.0.113.50 --dport 22 -j ACCEPT
```

**Allow established connections:**
```bash
# Very important! Allow responses to outgoing requests
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

**Allow loopback:**
```bash
# Allow localhost traffic (required for many apps)
sudo iptables -A INPUT -i lo -j ACCEPT
```

**Block specific IP:**
```bash
sudo iptables -A INPUT -s 203.0.113.100 -j DROP
```

**Default policies:**
```bash
# Set default policy to DROP
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT
```

**Delete rules:**
```bash
# Delete by line number
sudo iptables -L INPUT --line-numbers
sudo iptables -D INPUT 3

# Delete by specification
sudo iptables -D INPUT -p tcp --dport 80 -j ACCEPT
```

**Flush all rules:**
```bash
# Delete all rules (DANGER!)
sudo iptables -F

# Delete all rules in specific chain
sudo iptables -F INPUT
```

### Complete iptables Setup Example:

**Secure Web Server:**
```bash
#!/bin/bash

# Flush existing rules
sudo iptables -F

# Default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ping (ICMP)
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log dropped packets (optional)
sudo iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "

# Save rules (Ubuntu/Debian)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### Save and Restore Rules:

**Ubuntu/Debian:**
```bash
# Save rules
sudo iptables-save > /etc/iptables/rules.v4

# Restore rules
sudo iptables-restore < /etc/iptables/rules.v4

# Or install persistent
sudo apt install iptables-persistent
sudo netfilter-persistent save
sudo netfilter-persistent reload
```

**RHEL/CentOS:**
```bash
# Save rules
sudo service iptables save

# Or manually
sudo iptables-save > /etc/sysconfig/iptables
```

---

## 🔍 Testing Firewall Rules

### From Another Machine:

```bash
# Test if port is open
nc -zv server-ip 22
nc -zv server-ip 80

# Or use nmap
nmap -p 22,80,443 server-ip

# Test with curl
curl http://server-ip:80
```

### From the Server:

```bash
# Check listening ports
sudo ss -tlnp

# Test localhost
curl http://localhost:80

# Check firewall status
sudo ufw status          # UFW
sudo iptables -L -v      # iptables
```

### Watch Firewall Logs:

```bash
# UFW logs
sudo tail -f /var/log/ufw.log

# iptables logs (if LOG target used)
sudo tail -f /var/log/syslog | grep "IPTables-Dropped"

# Or kernel logs
sudo dmesg | grep -i firewall
```

---

## 🚀 DevOps Firewall Scenarios

### Scenario 1: Web Application Stack

**Requirements:**
```
- Public: HTTP (80), HTTPS (443)
- SSH from office IP only: 203.0.113.50
- Database (5432) from app servers only: 10.0.1.0/24
```

**UFW solution:**
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Web traffic
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# SSH from office only
sudo ufw allow from 203.0.113.50 to any port 22

# Database from app servers
sudo ufw allow from 10.0.1.0/24 to any port 5432

sudo ufw enable
```

### Scenario 2: Docker Host

**Problem:** Docker bypasses UFW!

**Solution:**
```bash
# Edit /etc/ufw/after.rules
sudo nano /etc/ufw/after.rules

# Add before final COMMIT:
*filter
:DOCKER-USER - [0:0]
# Allow from specific network only
-A DOCKER-USER -s 10.0.1.0/24 -j RETURN
# Block everything else
-A DOCKER-USER -j DROP
COMMIT

# Reload
sudo ufw reload
```

### Scenario 3: Kubernetes Node

**Requirements:**
```
- API Server: 6443
- Kubelet API: 10250
- Node Ports: 30000-32767
```

```bash
sudo ufw allow 6443/tcp
sudo ufw allow 10250/tcp
sudo ufw allow 30000:32767/tcp
```

### Scenario 4: Rate Limiting

**Prevent SSH brute force:**
```bash
# UFW: limit SSH attempts
sudo ufw limit 22/tcp

# iptables: allow max 5 connections per minute
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set
sudo iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 5 -j DROP
```

---

## 🐛 Troubleshooting Firewalls

### Problem: Can't connect to service

**Check 1: Is service running?**
```bash
sudo ss -tlnp | grep :80
```

**Check 2: Is firewall blocking?**
```bash
# Check rules
sudo ufw status
sudo iptables -L INPUT -v

# Temporarily disable (DANGER in production!)
sudo ufw disable
# Try connection
# If works, firewall is the issue
```

**Check 3: Test from localhost**
```bash
curl http://localhost:80
# Works locally but not remotely? → Firewall
```

**Check 4: Check logs**
```bash
sudo tail -f /var/log/ufw.log
# Look for BLOCK entries
```

### Problem: Rule not working

**Check order:**
```bash
sudo iptables -L --line-numbers

# Rules evaluated top-to-bottom
# First match wins!

# Wrong order:
[ 1] DROP all from anywhere
[ 2] ACCEPT port 22        ← Never reached!

# Fix: Delete and re-add in correct order
```

### Problem: Locked yourself out

**Prevention:**
```bash
# Before enabling firewall, ensure SSH is allowed!
sudo ufw allow 22
sudo ufw enable

# Or use at command
echo "sudo ufw disable" | at now + 5 minutes
sudo ufw enable
# If you get locked out, firewall disables in 5 min
```

**Recovery:**
```bash
# Physical/console access
sudo ufw disable

# Or cloud provider console
# Or recovery mode
```

---

## 🎯 Quick Check: Do You Understand?

1. **What does a firewall do?**
   <details>
   <summary>Answer</summary>
   Filters network traffic based on rules, allowing/blocking packets based on IP, port, protocol.
   </details>

2. **How do you allow HTTP with UFW?**
   <details>
   <summary>Answer</summary>
   `sudo ufw allow 80/tcp` or `sudo ufw allow http`
   </details>

3. **What's the difference between DROP and REJECT?**
   <details>
   <summary>Answer</summary>
   DROP silently discards packet. REJECT discards and sends error message to sender.
   </details>

4. **Why is rule order important in iptables?**
   <details>
   <summary>Answer</summary>
   Rules evaluated top-to-bottom, first match wins. If DROP comes before ALLOW, traffic is blocked.
   </details>

---

## 📝 Key Takeaways

✅ Firewalls filter network traffic (allow/block)  
✅ UFW = simple, iptables = powerful  
✅ Always allow SSH before enabling firewall!  
✅ Default deny incoming, allow outgoing  
✅ Rules are evaluated in order (first match wins)  
✅ Use `ufw status` or `iptables -L` to check rules  
✅ Test rules from another machine  
✅ Docker bypasses UFW (needs special config)  
✅ Save rules to persist after reboot  

---

## 🚀 Next Steps

You understand firewalls! Next, let's learn network monitoring and performance testing.

**Next lesson:** `17-network-monitoring.md`

---

## 💡 Pro Tip

**Create firewall testing script:**
```bash
#!/bin/bash
# fw-test.sh <target-ip>

IP=$1
echo "=== Firewall Test for $IP ==="
echo
echo "Common Ports:"
for port in 22 80 443 3306 5432; do
  echo -n "Port $port: "
  nc -zv -w 2 $IP $port 2>&1 | grep -q succeeded && echo "OPEN ✅" || echo "CLOSED ❌"
done
```

Quick port check! 🔥
