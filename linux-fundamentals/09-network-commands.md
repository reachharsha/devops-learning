# 09 - Network Commands

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Checking network configuration (ip, ifconfig)
- Testing connectivity (ping, traceroute)
- Network diagnostics (netstat, ss)
- Transferring files (scp, rsync)
- Web requests (curl, wget)
- DNS lookups (nslookup, dig, host)

---

## 🌐 Why Network Commands Matter for DevOps

```
DevOps Reality:
- Servers in the cloud
- Microservices communicating
- APIs everywhere
- SSH into remote machines
- Download/upload files
- Troubleshoot connectivity
- Monitor network traffic

You NEED network commands daily!
```

---

## 🔍 Network Configuration

### **ip - Network Interface Configuration**

**Modern command (replaces old ifconfig)**

```bash
# Show all network interfaces
ip addr
ip a              # Short form

# Show specific interface
ip addr show eth0
ip a s eth0

# Show only IPv4
ip -4 addr

# Show only IPv6
ip -6 addr

# Show routing table
ip route
ip r              # Short form

# Show network statistics
ip -s link
```

**Understanding ip addr output:**
```bash
$ ip addr

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
    
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::a00:27ff:fe4e:66a1/64 scope link

Breakdown:
lo              Loopback (localhost)
eth0            Ethernet interface
127.0.0.1       Localhost IP
192.168.1.100   Your computer's IP
/24             Subnet mask (255.255.255.0)
UP              Interface is active
mtu 1500        Maximum transmission unit
```

### **ifconfig - Old but Still Common**

```bash
# Install if missing
sudo apt install net-tools

# Show all interfaces
ifconfig

# Show specific interface
ifconfig eth0

# Enable/disable interface
sudo ifconfig eth0 up
sudo ifconfig eth0 down
```

---

## 🏓 Testing Connectivity

### **ping - Test Reachability**

**Real-World Analogy:**
```
ping = Sending "Are you there?" messages

You:     "Hey Google, you alive?"
Google:  "Yes! Got your message in 10ms"
You:     "Cool! Can I access your website?"
```

```bash
# Ping a host (Ctrl+C to stop)
ping google.com

# Output:
PING google.com (142.250.185.46): 56 data bytes
64 bytes from 142.250.185.46: icmp_seq=0 ttl=117 time=10.5 ms
64 bytes from 142.250.185.46: icmp_seq=1 ttl=117 time=9.8 ms
64 bytes from 142.250.185.46: icmp_seq=2 ttl=117 time=10.2 ms

What it means:
64 bytes      Packet size
icmp_seq=0    Sequence number
ttl=117       Time to live (hops remaining)
time=10.5 ms  Round-trip time (lower = faster)

# Send specific number of packets
ping -c 4 google.com

# Ping with specific interval (default 1 second)
ping -i 0.5 google.com    # Every 0.5 seconds

# Ping until success
ping google.com

# Ping IPv4 only
ping -4 google.com

# Ping IPv6 only
ping -6 google.com

# Ping local network gateway
ping 192.168.1.1

# Ping localhost
ping 127.0.0.1
ping localhost
```

**Interpreting ping results:**
```bash
# Success ✅
64 bytes from ...: time=10ms
→ Host is reachable, network is working

# Request timeout ❌
Request timeout for icmp_seq 0
→ Host unreachable or firewall blocking

# Unknown host ❌
ping: unknown host google.com
→ DNS resolution failed

# Network unreachable ❌
Network is unreachable
→ No route to host (check network config)

# High latency ⚠️
time=500ms
→ Network congestion or distance
```

---

### **traceroute - Trace Network Path**

**Real-World Analogy:**
```
traceroute = Tracking your Amazon package

Your PC → Router → ISP → Internet → Server
Each hop shows:
- Which router
- How long it took
- If there's a problem
```

```bash
# Install if missing
sudo apt install traceroute    # Ubuntu
sudo yum install traceroute    # CentOS

# Trace route to host
traceroute google.com

# Output:
 1  192.168.1.1 (192.168.1.1)  1.234 ms  1.123 ms  1.056 ms
 2  10.0.0.1 (10.0.0.1)  5.678 ms  5.432 ms  5.234 ms
 3  142.250.185.46 (142.250.185.46)  10.123 ms  10.056 ms  10.234 ms

Each line:
Hop 1: Your router
Hop 2: ISP gateway
Hop 3: Destination

# Don't resolve hostnames (faster)
traceroute -n google.com

# Use ICMP instead of UDP
traceroute -I google.com

# Max hops
traceroute -m 15 google.com
```

---

## 📊 Network Statistics

### **netstat - Network Statistics**

**Show active connections, ports, routing**

```bash
# Install if missing
sudo apt install net-tools

# All connections
netstat -a

# Listening ports only
netstat -l

# TCP connections
netstat -t

# UDP connections
netstat -u

# Show process/program
netstat -p

# Don't resolve names (faster, shows IPs)
netstat -n

# Commonly used combinations:
netstat -tuln      # All TCP/UDP listening ports (numeric)
netstat -tulnp     # Same + show programs (requires sudo)
netstat -an        # All connections (numeric)
```

**Useful examples:**
```bash
# What's listening on port 80?
sudo netstat -tlnp | grep :80

# Show all established connections
netstat -tn

# Show routing table
netstat -rn

# Network statistics
netstat -s

# Check if port 22 (SSH) is listening
sudo netstat -tlnp | grep :22
```

---

### **ss - Socket Statistics**

**Modern replacement for netstat (faster)**

```bash
# All sockets
ss -a

# Listening ports
ss -l

# TCP sockets
ss -t

# UDP sockets
ss -u

# Show process
ss -p

# Numeric (don't resolve names)
ss -n

# Common combinations:
ss -tuln        # TCP/UDP listening ports
ss -tulnp       # Same + programs (requires sudo)
ss -tan         # All TCP connections (numeric)

# Examples:
# What's using port 3000?
sudo ss -tlnp | grep :3000

# Show all established connections
ss -t -r state established

# Summary statistics
ss -s
```

---

## 📁 File Transfer

### **scp - Secure Copy**

**Copy files over SSH**

```bash
# Copy file TO remote server
scp local-file.txt user@server:/remote/path/

# Example:
scp app.log admin@192.168.1.100:/var/log/

# Copy file FROM remote server
scp user@server:/remote/file.txt /local/path/

# Example:
scp admin@192.168.1.100:/var/log/app.log ./

# Copy directory (recursive)
scp -r /local/folder user@server:/remote/path/

# Copy with specific port
scp -P 2222 file.txt user@server:/path/

# Preserve permissions & timestamps
scp -p file.txt user@server:/path/

# Verbose output
scp -v file.txt user@server:/path/

# Limit bandwidth (KB/s)
scp -l 1000 largefile.zip user@server:/path/
```

**Practical examples:**
```bash
# Backup to remote server
scp -r /var/www/html admin@backup-server:/backups/website/

# Download database backup
scp db-admin@db-server:/backups/db-2024.sql ./

# Copy SSH key to server
scp ~/.ssh/id_rsa.pub user@server:~/.ssh/authorized_keys

# Multiple files
scp file1.txt file2.txt user@server:/path/
```

---

### **rsync - Advanced File Sync**

**Better than scp for large transfers**

```bash
# Basic sync
rsync source destination

# Sync to remote server
rsync -av /local/folder/ user@server:/remote/folder/

# Options explained:
-a    Archive mode (preserves everything)
-v    Verbose
-z    Compress during transfer
-P    Show progress + keep partial files
-h    Human-readable numbers

# Recommended combo:
rsync -avzP /source/ user@server:/dest/

# Delete files in dest that don't exist in source
rsync -av --delete /source/ /dest/

# Dry run (see what would happen)
rsync -avzP --dry-run /source/ user@server:/dest/

# Exclude files
rsync -av --exclude='*.log' /source/ /dest/
rsync -av --exclude='node_modules' /source/ /dest/

# Sync via SSH on specific port
rsync -avzP -e "ssh -p 2222" /source/ user@server:/dest/

# Show progress
rsync -av --progress /source/ /dest/
```

**rsync vs scp:**
```
scp:
✅ Simpler syntax
✅ Pre-installed usually
❌ Always copies everything
❌ No resume
❌ Slower for large transfers

rsync:
✅ Only copies changed files
✅ Can resume interrupted transfers
✅ More options
✅ Faster for updates
❌ Slightly complex syntax
```

---

## 🌐 Web Requests

### **curl - Transfer Data**

**Swiss Army knife for HTTP/FTP/etc.**

```bash
# GET request (fetch webpage)
curl https://google.com

# Save to file
curl https://example.com/file.zip -o file.zip
curl https://example.com/file.zip --output file.zip

# Follow redirects
curl -L https://github.com

# Show only headers
curl -I https://google.com
curl --head https://google.com

# Include headers in output
curl -i https://api.example.com

# Verbose (debug)
curl -v https://google.com

# POST request
curl -X POST https://api.example.com/users

# POST with data
curl -X POST https://api.example.com/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"secret"}'

# POST form data
curl -X POST https://example.com/form \
  -d "name=John" \
  -d "email=john@example.com"

# Upload file
curl -F "file=@document.pdf" https://example.com/upload

# Basic authentication
curl -u username:password https://api.example.com

# Custom headers
curl -H "Authorization: Bearer token123" \
     -H "Accept: application/json" \
     https://api.example.com

# Check if website is up
curl -I -s -o /dev/null -w "%{http_code}" https://google.com
# Returns: 200 (OK), 404 (Not Found), 500 (Error), etc.
```

**Practical examples:**
```bash
# Download file
curl -O https://example.com/script.sh

# Test API endpoint
curl https://api.github.com/users/username

# Check website response time
curl -w "Time: %{time_total}s\n" -o /dev/null -s https://google.com

# Download with progress bar
curl -# -O https://example.com/large-file.zip
```

---

### **wget - Download Files**

**Simpler than curl for downloads**

```bash
# Download file
wget https://example.com/file.zip

# Download with different name
wget -O myfile.zip https://example.com/file.zip

# Download in background
wget -b https://example.com/largefile.zip

# Resume interrupted download
wget -c https://example.com/largefile.zip

# Download entire website (mirror)
wget --mirror --convert-links --page-requisites https://example.com

# Limit download speed
wget --limit-rate=200k https://example.com/file.zip

# Download multiple files from list
wget -i urls.txt

# Retry on failure
wget --tries=10 https://example.com/file.zip

# Quiet mode
wget -q https://example.com/file.zip

# Spider mode (check links, don't download)
wget --spider https://example.com
```

**curl vs wget:**
```
curl:
✅ APIs and data transfer
✅ More protocols
✅ Pipe to other commands
✅ Better for DevOps/scripting

wget:
✅ Recursive downloads
✅ Simpler for file downloads
✅ Better for mirroring sites
✅ Resume downloads easily
```

---

## 🔍 DNS Lookup

### **nslookup - Query DNS**

```bash
# Lookup domain
nslookup google.com

# Output:
Server:  8.8.8.8
Address: 8.8.8.8#53

Non-authoritative answer:
Name:    google.com
Address: 142.250.185.46

# Reverse lookup (IP to domain)
nslookup 8.8.8.8

# Specify DNS server
nslookup google.com 1.1.1.1

# Query specific record type
nslookup -type=mx google.com      # Mail servers
nslookup -type=ns google.com      # Name servers
nslookup -type=txt google.com     # TXT records
```

---

### **dig - Advanced DNS Lookup**

```bash
# Install if missing
sudo apt install dnsutils

# Basic lookup
dig google.com

# Short answer only
dig google.com +short

# Specific record types
dig google.com MX        # Mail servers
dig google.com NS        # Name servers
dig google.com TXT       # TXT records
dig google.com AAAA      # IPv6 address

# Reverse DNS
dig -x 8.8.8.8

# Use specific DNS server
dig @8.8.8.8 google.com
dig @1.1.1.1 google.com

# Trace DNS path
dig google.com +trace

# No recursion
dig google.com +norecurse
```

---

### **host - Simple DNS Lookup**

```bash
# Basic lookup
host google.com

# Reverse lookup
host 8.8.8.8

# All records
host -a google.com

# Specific record
host -t MX google.com
```

---

## 🎯 Quick Check: Do You Understand?

1. **How do you show your IP address?**
   <details>
   <summary>Answer</summary>
   ip addr  OR  ip a
   </details>

2. **How do you test if google.com is reachable?**
   <details>
   <summary>Answer</summary>
   ping google.com
   </details>

3. **How do you see which ports are listening?**
   <details>
   <summary>Answer</summary>
   sudo netstat -tlnp  OR  sudo ss -tlnp
   </details>

4. **How do you copy a file to a remote server?**
   <details>
   <summary>Answer</summary>
   scp file.txt user@server:/path/
   </details>

5. **How do you download a file from a URL?**
   <details>
   <summary>Answer</summary>
   wget URL  OR  curl -O URL
   </details>

---

## 🏋️ Hands-On Exercise

```bash
# 1. Show your network interfaces
ip addr

# 2. Ping Google
ping -c 4 google.com

# 3. Trace route to Google
traceroute google.com

# 4. Show listening ports
sudo ss -tlnp

# 5. Check DNS for google.com
dig google.com +short

# 6. Download Google homepage
curl https://google.com > google.html

# 7. Check what's listening on port 22 (SSH)
sudo ss -tlnp | grep :22

# 8. Test website response
curl -I https://github.com
```

---

## 📝 Key Takeaways

✅ **ip addr** - show network interfaces  
✅ **ping** - test connectivity  
✅ **traceroute** - trace network path  
✅ **ss -tuln** - show listening ports (better than netstat)  
✅ **scp** - secure file copy  
✅ **rsync** - advanced sync (better for large files)  
✅ **curl** - web requests & APIs  
✅ **wget** - download files  
✅ **dig** - DNS lookups  

---

## 🚀 Next Steps

You can now troubleshoot networks like a pro!

**Next lesson:** [10-system-services.md](10-system-services.md) - Managing system services

---

## 💡 Pro Tips

**Quick connectivity check:**
```bash
# Check internet connectivity
ping -c 1 8.8.8.8 && echo "Network OK" || echo "Network DOWN"

# Check DNS
ping -c 1 google.com && echo "DNS OK" || echo "DNS problem"

# Check specific service
curl -I http://localhost:3000
```

**Find what's using a port:**
```bash
# Method 1
sudo ss -tlnp | grep :8080

# Method 2
sudo lsof -i :8080

# Method 3
sudo netstat -tlnp | grep :8080
```

**Test web server:**
```bash
# Quick HTTP check
curl -I http://localhost

# Response time
curl -w "Time: %{time_total}s\n" -o /dev/null -s http://localhost

# Status code only
curl -s -o /dev/null -w "%{http_code}" http://localhost
```

**Network troubleshooting workflow:**
```bash
# 1. Can you reach localhost?
ping -c 1 127.0.0.1

# 2. Can you reach your gateway?
ping -c 1 $(ip route | grep default | awk '{print $3}')

# 3. Can you reach the internet (Google DNS)?
ping -c 1 8.8.8.8

# 4. Can you resolve DNS?
ping -c 1 google.com

# 5. Check route
traceroute google.com
```

Master these network commands = DevOps success! 🚀
