---
render_with_liquid: false
---
# 14 - Network Troubleshooting Methodology

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Systematic troubleshooting approach
- OSI layer-by-layer methodology
- Common network issues and solutions
- Diagnostic tools for each problem type
- Documentation best practices

---

## 🧠 The Golden Rule of Troubleshooting

**"Don't guess. Test systematically. Document everything."**

```
❌ Bad approach:
"It's probably the firewall. Let me check 50 things randomly."

✅ Good approach:
"Let me test each layer methodically, starting from the bottom."
```

---

## 🎯 The Systematic Approach

### Step 1: Define the Problem
```
Ask these questions:
❓ What exactly is broken?
❓ When did it start?
❓ What changed recently?
❓ Can you reproduce it?
❓ Is it affecting everyone or specific users?
❓ What's the error message (exact wording)?
```

### Step 2: Gather Information
```
Collect:
✅ Error messages (screenshots, logs)
✅ Recent changes (deployments, config updates)
✅ Affected systems (IP addresses, hostnames)
✅ Timeline (when it started, frequency)
✅ Impact (how many users, which services)
```

### Step 3: Form Hypothesis
```
Based on symptoms, hypothesize:
"If X is the problem, then Y test should show Z result"

Example:
"If DNS is broken, then ping by IP will work but ping by hostname won't"
```

### Step 4: Test Hypothesis
```
Run specific tests to prove/disprove:
✅ Test passes → Hypothesis correct, fix the issue
❌ Test fails → Form new hypothesis, repeat
```

### Step 5: Implement Fix
```
Fix the root cause (not symptoms!)
Test the fix thoroughly
Document the solution
```

### Step 6: Verify and Document
```
✅ Confirm issue is resolved
✅ Monitor for recurrence
✅ Document what happened and how you fixed it
```

---

## 🏗️ OSI Layer-by-Layer Troubleshooting

**Always start from Layer 1 and work up!**

### 🔌 Layer 1: Physical

**Question:** Is the cable/hardware working?

```bash
# Check if interface is up
ip link show eth0

# Look for: <UP,LOWER_UP>
# UP = interface enabled
# LOWER_UP = cable connected

# Check physical link
ethtool eth0 | grep "Link detected"
# Output: Link detected: yes ✅

# Check cable connection status
cat /sys/class/net/eth0/carrier
# 1 = connected, 0 = disconnected

# View physical errors
ip -s link show eth0
# Look for: errors, dropped, overrun
```

**Common Issues:**
```
❌ Cable unplugged
❌ Bad cable (replace it)
❌ Wrong port on switch
❌ Interface disabled
❌ Hardware failure (NIC)
```

**Quick Tests:**
```bash
# Bring interface up
sudo ip link set eth0 up

# Check dmesg for hardware errors
dmesg | grep eth0 | tail -20

# Check for link flapping
dmesg | grep -i "link up\|link down"
```

---

### 🔗 Layer 2: Data Link

**Question:** Can devices communicate locally?

```bash
# Check MAC address
ip link show eth0 | grep link/ether

# Check ARP table
ip neigh show

# Look for:
192.168.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE ✅
192.168.1.100 dev eth0 FAILED ❌

# Send ARP request
arping -c 3 -I eth0 192.168.1.1

# Check for duplicate IPs (ARP conflicts)
sudo arping -D -I eth0 192.168.1.100
# If duplicates exist, you'll see multiple responses
```

**Common Issues:**
```
❌ Duplicate IP addresses
❌ ARP cache issues
❌ Wrong subnet/VLAN
❌ Switch port disabled
❌ MAC address filtering
```

**Quick Fixes:**
```bash
# Flush ARP cache
sudo ip neigh flush all

# Check for duplicate MAC
arp-scan --interface=eth0 --localnet | sort -k2 | uniq -d -f1

# Verify you're on correct VLAN
ip link show | grep vlan
```

---

### 🌐 Layer 3: Network

**Question:** Can packets be routed?

```bash
# Check IP address assigned
ip addr show eth0

# Check routing table
ip route show

# Ping gateway
ping -c 4 192.168.1.1

# Ping external IP (tests routing)
ping -c 4 8.8.8.8

# Trace route
traceroute 8.8.8.8

# Check route to specific destination
ip route get 8.8.8.8
```

**Common Issues:**
```
❌ No IP address assigned
❌ Wrong IP/subnet
❌ No default gateway
❌ Incorrect routing table
❌ Firewall blocking ICMP
❌ IP conflict
```

**Quick Fixes:**
```bash
# Check if DHCP assigned IP
sudo dhclient -v eth0

# Manually assign IP
sudo ip addr add 192.168.1.100/24 dev eth0

# Add default gateway
sudo ip route add default via 192.168.1.1

# Check for IP conflicts
sudo arping -D -I eth0 $(ip -4 addr show eth0 | grep -oP '(?<=inet\s)\d+(\.\d+){3}')
```

---

### 🚚 Layer 4: Transport

**Question:** Are ports open and accessible?

```bash
# Check if service is listening
sudo ss -tlnp | grep :80

# Test local port
telnet localhost 80
# or
nc -zv localhost 80

# Test remote port
telnet server.com 80
nc -zv server.com 80

# Check established connections
ss -tn state established

# Look for connection in specific state
ss -tn state syn-sent  # Stuck connecting
ss -tn state time-wait # Waiting to close
```

**Common Issues:**
```
❌ Service not running
❌ Service not listening on expected port
❌ Firewall blocking port
❌ Wrong port number
❌ Port already in use
❌ Too many connections (port exhaustion)
```

**Quick Tests:**
```bash
# Is the service running?
systemctl status nginx

# Is it listening?
sudo ss -tlnp | grep nginx

# Can you connect locally?
curl http://localhost:80

# Can you connect remotely?
curl http://server-ip:80

# Check firewall
sudo iptables -L -n | grep 80
```

---

### 🎭 Layer 7: Application

**Question:** Is the application working correctly?

```bash
# Check application logs
sudo tail -f /var/log/nginx/error.log
sudo journalctl -u nginx -f

# Test HTTP response
curl -I http://localhost:80

# Test with verbose output
curl -v http://localhost:80

# Check application status
systemctl status application-name

# View recent errors
sudo journalctl -u application-name -n 50 --no-pager
```

**Common Issues:**
```
❌ Application crashed
❌ Configuration error
❌ Database unreachable
❌ Authentication failing
❌ SSL/TLS certificate issues
❌ Resource exhaustion (memory, disk)
```

**Quick Checks:**
```bash
# Application running?
ps aux | grep application-name

# Check resource usage
top -u application-user

# Check disk space
df -h

# Check memory
free -h

# Recent application errors
grep -i error /var/log/application.log | tail -20
```

---

## 🔍 Common Problem Patterns

### Problem: "Can't reach server"

**Systematic Test:**
```bash
# 1. Can you ping by IP?
ping -c 4 192.168.1.100

YES → Layer 3 works, check Layer 4+
NO  → Check Layer 1-3

# 2. Can you ping by hostname?
ping -c 4 server.example.com

YES → Everything works, check application
NO  → DNS problem (Layer 7)

# 3. Can you reach the port?
nc -zv 192.168.1.100 80

YES → Network works, check application
NO  → Port blocked (firewall or service down)

# 4. Does DNS resolve?
dig server.example.com +short

YES → DNS works
NO  → DNS misconfigured
```

### Problem: "Service is slow"

**Systematic Test:**
```bash
# 1. Check network latency
ping -c 100 server.example.com | tail -5
# Look at avg/max latency

# 2. Check packet loss
mtr -r -c 100 server.example.com
# Look for Loss% column

# 3. Check bandwidth
iperf3 -c server.example.com
# Tests actual throughput

# 4. Check server resources
ssh server.example.com 'top -bn1 | head -20'

# 5. Check application logs
ssh server.example.com 'tail -100 /var/log/app.log | grep -i slow'
```

### Problem: "Intermittent connectivity"

**Diagnostic Steps:**
```bash
# 1. Continuous ping to detect drops
ping -i 0.5 server.example.com | tee ping.log
# Let run for several minutes, look for drops

# 2. MTR for packet loss
mtr -r -c 1000 server.example.com > mtr-report.txt

# 3. Check for link flapping
dmesg | grep -i "link up\|link down"

# 4. Monitor interface errors
watch -n 1 'ip -s link show eth0'

# 5. Check ARP cache stability
watch -n 2 'ip neigh show'
```

### Problem: "DNS not resolving"

**Systematic Test:**
```bash
# 1. Can you reach DNS server?
ping -c 4 8.8.8.8

# 2. Is DNS port accessible?
nc -zvu 8.8.8.8 53

# 3. Test DNS query
dig @8.8.8.8 google.com

# 4. Check local DNS config
cat /etc/resolv.conf

# 5. Test with different DNS server
dig @1.1.1.1 google.com

# 6. Check if DNS is cached incorrectly
sudo systemd-resolve --flush-caches
dig google.com
```

---

## 📋 Troubleshooting Checklists

### Web Application Down Checklist

```bash
# Layer 1: Physical
[ ] Cable connected? ethtool eth0 | grep "Link detected"
[ ] Interface up? ip link show eth0

# Layer 2: Data Link
[ ] IP assigned? ip addr show eth0
[ ] Gateway reachable? ping -c 4 <gateway>

# Layer 3: Network
[ ] Can ping external IP? ping -c 4 8.8.8.8
[ ] Routing correct? ip route show

# Layer 4: Transport
[ ] Service listening? sudo ss -tlnp | grep :80
[ ] Port accessible locally? nc -zv localhost 80
[ ] Port accessible remotely? nc -zv <server-ip> 80

# Layer 7: Application
[ ] Service running? systemctl status nginx
[ ] No errors in logs? tail -50 /var/log/nginx/error.log
[ ] Config valid? nginx -t
[ ] Disk space? df -h
[ ] Memory available? free -h
```

### Database Connection Failed Checklist

```bash
# From application server:
[ ] Can ping DB server? ping -c 4 db-server
[ ] Can resolve DB hostname? dig db-server +short
[ ] Can reach DB port? nc -zv db-server 5432
[ ] Firewall allowing? sudo iptables -L -n | grep 5432

# On database server:
[ ] DB service running? systemctl status postgresql
[ ] DB listening on network? ss -tlnp | grep 5432
[ ] DB allowing remote connections? grep listen_addresses /etc/postgresql/*/main/postgresql.conf
[ ] DB has connection slots? psql -c "SELECT count(*) FROM pg_stat_activity;"
[ ] Authentication correct? grep -v "^#" /etc/postgresql/*/main/pg_hba.conf
```

### Container Can't Reach Internet Checklist

```bash
# From container:
[ ] Has IP address? docker exec <container> ip addr
[ ] Can ping gateway? docker exec <container> ping -c 4 172.17.0.1
[ ] Can ping external? docker exec <container> ping -c 4 8.8.8.8
[ ] DNS resolving? docker exec <container> nslookup google.com

# On host:
[ ] IP forwarding enabled? cat /proc/sys/net/ipv4/ip_forward
[ ] Docker network exists? docker network ls
[ ] NAT rules present? sudo iptables -t nat -L -n | grep docker
[ ] DNS configured? docker exec <container> cat /etc/resolv.conf
```

---

## 🛠️ Essential Troubleshooting Tools

```bash
# Layer 1-2
ethtool eth0              # Physical link status
ip link show              # Interface status
ip neigh show             # ARP table

# Layer 3
ping                      # Basic connectivity
traceroute                # Route tracing
mtr                       # Continuous route monitoring
ip route show             # Routing table

# Layer 4
ss -tlnp                  # Listening ports
nc -zv                    # Port testing
telnet                    # Port testing (interactive)

# Layer 7
curl -v                   # HTTP testing
dig                       # DNS queries
tcpdump                   # Packet capture

# All layers
tcpdump                   # See everything
wireshark                 # GUI packet analysis
```

---

## 📝 Documentation Template

**Always document your findings!**

```markdown
## Issue Report: [Brief Description]

**Date/Time:** 2026-01-23 14:30 UTC
**Reporter:** John Doe
**Severity:** High / Medium / Low
**Affected Systems:** web-01, web-02

### Symptoms
- Users receiving "Connection timeout"
- Started at 14:00 UTC
- Affects 50% of users

### Investigation

**Layer 1: Physical**
- ✅ Cable connected
- ✅ Interface UP

**Layer 2: Data Link**
- ✅ IP assigned: 192.168.1.100
- ✅ ARP resolving gateway

**Layer 3: Network**
- ✅ Can ping gateway
- ❌ Cannot ping 8.8.8.8
- Finding: No default route!

**Layer 4: Transport**
- N/A (didn't reach this layer)

### Root Cause
Default gateway was removed during network maintenance

### Solution
```bash
sudo ip route add default via 192.168.1.1 dev eth0
```

### Verification
- ✅ Can now ping external IPs
- ✅ Users can access application
- ✅ Monitored for 15 minutes, stable

### Prevention
- Added default route to persistent network config
- Created monitoring alert for missing default route
- Documented change in runbook

### Timeline
14:00 - Issue began
14:05 - Reported by users
14:10 - Investigation started
14:25 - Root cause identified
14:30 - Fix applied
14:35 - Verified resolution
```

---

## 💼 Real-World Scenarios

### Scenario 1: Slow Application

```
Symptom: Application response time 30s (normally <1s)

Investigation:
1. ping server → avg 150ms (normally 10ms) ← Network issue!
2. mtr server → 40% packet loss on hop 3
3. traceroute → Timeout on hop 3
4. Contact ISP → Router failure, rerouting traffic

Resolution: ISP fixed router
Lesson: Not always your fault! Check external dependencies
```

### Scenario 2: Can Ping But Can't Access Service

```
Symptom: ping works, curl fails

Investigation:
1. ping server → Works ✅ (Layer 3 OK)
2. nc -zv server 80 → Connection refused ❌
3. ssh server 'ss -tlnp | grep :80' → Nothing listening!
4. ssh server 'systemctl status nginx' → inactive (dead)

Resolution: systemctl start nginx
Root Cause: Service crashed, no monitoring
Lesson: Monitor critical services!
```

### Scenario 3: DNS Resolution Intermittent

```
Symptom: Sometimes resolves, sometimes times out

Investigation:
1. dig google.com → Works
2. dig google.com (again) → Timeout
3. dig @8.8.8.8 google.com → Always works
4. Check /etc/resolv.conf → Multiple nameservers, first one is bad!

Resolution: Remove broken DNS server from /etc/resolv.conf
Lesson: First DNS server in list is tried first
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the first layer to check when troubleshooting?**
   <details>
   <summary>Answer</summary>
   Layer 1 (Physical) - Check if cables are connected and interfaces are up
   </details>

2. **How do you test if a port is listening?**
   <details>
   <summary>Answer</summary>
   `sudo ss -tlnp | grep <port>` or `nc -zv localhost <port>`
   </details>

3. **What does "can ping by IP but not by hostname" indicate?**
   <details>
   <summary>Answer</summary>
   DNS problem - network layer works but DNS resolution is broken
   </details>

4. **What tool shows packet loss on each hop?**
   <details>
   <summary>Answer</summary>
   `mtr` - combines ping and traceroute with statistics
   </details>

---

## 📝 Key Takeaways

✅ **Always work bottom-up** (Layer 1 → Layer 7)  
✅ **Don't guess** - test systematically  
✅ **Document everything** - symptoms, tests, fixes  
✅ **One change at a time** - know what fixed it  
✅ **Verify the fix** - don't assume it worked  
✅ **Learn from issues** - update monitoring, docs  
✅ **Common pattern:** ping works → Layer 3 OK, check Layer 4+  
✅ **Common pattern:** ping fails → Check Layer 1-3  

---

## 🚀 Next Steps

You have a systematic troubleshooting methodology! Next, let's deep dive into DNS - one of the most common failure points.

**Next lesson:** `15-dns-deep-dive.md`

---

## 💡 Pro Tip

**Create a troubleshooting script:**
```bash
#!/bin/bash
# quick-check.sh

echo "=== Quick Network Check ==="
echo
echo "1. Interface Status:"
ip -br link show
echo
echo "2. IP Addresses:"
ip -br addr show
echo
echo "3. Default Route:"
ip route show | grep default
echo
echo "4. DNS Servers:"
cat /etc/resolv.conf | grep nameserver
echo
echo "5. Gateway Reachable:"
ping -c 2 $(ip route | grep default | awk '{print $3}')
echo
echo "6. External Connectivity:"
ping -c 2 8.8.8.8
echo
echo "7. DNS Working:"
dig +short google.com
echo
echo "8. Listening Ports:"
sudo ss -tlnp
```

Run this first when troubleshooting! 🎯
