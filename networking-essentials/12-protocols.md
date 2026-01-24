---
render_with_liquid: false
---
# 12 - Network Protocols

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What network protocols are
- TCP vs UDP
- HTTP/HTTPS
- DNS
- DHCP
- SSH and other common protocols

---

## 🤔 What is a Protocol?

**A protocol is a set of rules for how devices communicate.**

**Real-World Analogy:**
```
Phone Call Protocol:
1. Dial number
2. Wait for answer
3. Say "Hello"
4. Have conversation
5. Say "Goodbye"
6. Hang up

Both parties follow these rules!

Network protocols work the same way.
```

---

## 🔄 TCP: Transmission Control Protocol

**The reliable, connection-oriented protocol.**

### Characteristics:
```
✅ Reliable - Guarantees delivery
✅ Ordered - Packets arrive in order
✅ Error-checked - Detects and fixes errors
✅ Connection-oriented - Establishes connection first
❌ Slower - Due to all the checks
```

### TCP Three-Way Handshake:

**Establishing Connection:**
```
Client                          Server
  │                               │
  │──── SYN ────────────────────→ │  "Can we talk?"
  │                               │
  │ ←───── SYN-ACK ──────────────│  "Yes! Can you hear me?"
  │                               │
  │──── ACK ────────────────────→ │  "Yes! Let's start!"
  │                               │
  │  [Connection Established]     │
  │                               │
  │──── Data ───────────────────→ │
  │ ←─── Data ───────────────────│
```

**Mnemonic:** **S**ome **Y**oung **N**etworker **A**sked **C**isco **K**nowledge

### TCP Header:
```
Source Port: 54321
Destination Port: 80
Sequence Number: 1000
Acknowledgment Number: 5000
Flags: ACK, PSH
Window Size: 65535
Checksum: 0x1234
```

### Common TCP Ports:
```
20/21  - FTP (File Transfer)
22     - SSH (Secure Shell)
23     - Telnet (Insecure remote access)
25     - SMTP (Email sending)
80     - HTTP (Web)
443    - HTTPS (Secure web)
3306   - MySQL
5432   - PostgreSQL
3389   - RDP (Remote Desktop)
8080   - Alternative HTTP
```

### Use Cases:
```
✅ Web browsing (HTTP/HTTPS)
✅ Email (SMTP, IMAP)
✅ File transfer (FTP, SFTP)
✅ SSH connections
✅ Database connections
✅ Any application needing reliability
```

### Viewing TCP Connections:
```bash
# Show all TCP connections
ss -t

# Show listening TCP ports
ss -tln

# Example output:
State   Recv-Q Send-Q  Local Address:Port  Peer Address:Port
ESTAB   0      0       192.168.1.100:22    192.168.1.50:54321
LISTEN  0      128     0.0.0.0:80          0.0.0.0:*
LISTEN  0      128     0.0.0.0:443         0.0.0.0:*
```

---

## 📡 UDP: User Datagram Protocol

**The fast, connectionless protocol.**

### Characteristics:
```
✅ Fast - Minimal overhead
✅ Connectionless - No handshake
✅ Small header - Less data overhead
❌ Unreliable - No delivery guarantee
❌ Unordered - Packets may arrive out of order
❌ No error recovery
```

### UDP Operation:
```
Client                          Server
  │                               │
  │──── Data ───────────────────→ │  "Here's some data!"
  │                               │
  │──── Data ───────────────────→ │  "Here's more!"
  │                               │
  No handshake, just send!
  Lost packets? Too bad!
```

### Common UDP Ports:
```
53     - DNS (Domain Name System)
67/68  - DHCP (Dynamic Host Configuration)
69     - TFTP (Trivial FTP)
123    - NTP (Network Time Protocol)
161    - SNMP (Network monitoring)
514    - Syslog
```

### Use Cases:
```
✅ DNS queries (speed > reliability)
✅ Video streaming (losing 1 frame OK)
✅ Online gaming (speed critical)
✅ VoIP (Voice over IP)
✅ DHCP (small, simple requests)
✅ Broadcast/multicast traffic
```

### Viewing UDP Connections:
```bash
# Show UDP connections
ss -u

# Show listening UDP ports
ss -uln

# Example:
State   Recv-Q Send-Q  Local Address:Port
UNCONN  0      0       0.0.0.0:53
UNCONN  0      0       0.0.0.0:67
```

---

## 📊 TCP vs UDP Comparison

```
Feature          TCP                    UDP
─────────────────────────────────────────────────────
Connection       Yes (handshake)        No
Reliability      Guaranteed             Best effort
Ordering         In-order               May be out of order
Error checking   Yes                    Minimal
Speed            Slower                 Faster
Header size      20-60 bytes            8 bytes
Use case         Accuracy critical      Speed critical
Example          Web, Email, SSH        DNS, Streaming, Gaming
```

**When to use which?**
```
Use TCP when:
- Data must arrive correctly
- Order matters
- Example: Downloading files, browsing websites

Use UDP when:
- Speed is critical
- Some data loss is acceptable
- Example: Live video, gaming, DNS queries
```

---

## 🌐 HTTP: HyperText Transfer Protocol

**The protocol of the web (Layer 7).**

### HTTP Request:
```
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
Connection: keep-alive

[Optional body data]
```

### HTTP Response:
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 1234
Server: nginx/1.18.0

<html>
<body>Hello World!</body>
</html>
```

### HTTP Methods:
```
GET     - Retrieve data (most common)
POST    - Send data to server
PUT     - Update/create resource
DELETE  - Remove resource
PATCH   - Partial update
HEAD    - Get headers only (no body)
OPTIONS - Get allowed methods
```

### HTTP Status Codes:
```
1xx - Informational
100 Continue

2xx - Success
200 OK
201 Created
204 No Content

3xx - Redirection
301 Moved Permanently
302 Found (Temporary redirect)
304 Not Modified

4xx - Client Errors
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests

5xx - Server Errors
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

### Testing HTTP:
```bash
# Simple GET request
curl http://example.com

# With headers
curl -I http://example.com

# POST request
curl -X POST -d "name=John" http://example.com/api

# Follow redirects
curl -L http://example.com

# Verbose output
curl -v http://example.com
```

---

## 🔒 HTTPS: HTTP Secure

**HTTP with encryption (TLS/SSL).**

### How HTTPS Works:
```
1. Client connects to server (port 443)
2. Server sends SSL certificate
3. Client verifies certificate
4. Encrypted connection established (TLS handshake)
5. All data encrypted thereafter
```

### TLS Handshake:
```
Client                          Server
  │                               │
  │──── ClientHello ────────────→ │
  │ ←─── ServerHello ────────────│
  │ ←─── Certificate ────────────│
  │ ←─── ServerHelloDone ────────│
  │──── ClientKeyExchange ──────→ │
  │──── ChangeCipherSpec ───────→ │
  │──── Finished ───────────────→ │
  │ ←─── ChangeCipherSpec ───────│
  │ ←─── Finished ───────────────│
  │                               │
  │  [Encrypted Communication]    │
```

### Testing HTTPS:
```bash
# Connect to HTTPS site
curl https://example.com

# View certificate
openssl s_client -connect example.com:443

# Show certificate details
openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -text

# Check certificate expiration
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## 🔍 DNS: Domain Name System

**Translates domain names to IP addresses.**

### The Problem:
```
Humans remember: google.com
Computers need: 142.250.185.46

DNS bridges this gap!
```

### DNS Query Process:
```
1. You type: google.com

2. Computer checks local cache
   - Found? Use it!
   - Not found? Continue...

3. Query recursive DNS server (usually ISP)
   - Example: 8.8.8.8 (Google DNS)

4. Recursive server queries:
   - Root server: "Who handles .com?"
   - TLD server: "Who handles google.com?"
   - Authoritative server: "google.com is 142.250.185.46"

5. Response cached and returned to you

6. Your browser connects to 142.250.185.46
```

### Visual DNS Hierarchy:
```
                    Root (.)
                       │
         ┌─────────────┼─────────────┐
        .com          .org          .net
         │
    ┌────┼────┐
 google amazon microsoft
    │
   www
```

### DNS Record Types:
```
A      - IPv4 address
         example.com → 93.184.216.34

AAAA   - IPv6 address
         example.com → 2606:2800:220:1:248:1893:25c8:1946

CNAME  - Alias (canonical name)
         www.example.com → example.com

MX     - Mail server
         example.com → mail.example.com

NS     - Name server
         example.com → ns1.example.com

TXT    - Text record (various uses)
         example.com → "v=spf1 include:_spf.example.com"

PTR    - Reverse lookup (IP to domain)
         34.216.184.93 → example.com
```

### DNS Commands:
```bash
# Simple DNS lookup
nslookup google.com

# Better tool: dig
dig google.com

# Output:
;; ANSWER SECTION:
google.com.  300  IN  A  142.250.185.46

# Query specific record type
dig google.com MX
dig google.com AAAA
dig google.com TXT

# Reverse DNS lookup
dig -x 8.8.8.8

# Use specific DNS server
dig @8.8.8.8 google.com

# Short answer only
dig +short google.com

# Trace DNS path
dig +trace google.com
```

### Common DNS Servers:
```
Google DNS:
8.8.8.8
8.8.4.4

Cloudflare DNS:
1.1.1.1
1.0.0.1

OpenDNS:
208.67.222.222
208.67.220.220
```

### Configure DNS on Linux:
```bash
# View current DNS servers
cat /etc/resolv.conf

# Example:
nameserver 8.8.8.8
nameserver 8.8.4.4

# Add DNS server
echo "nameserver 1.1.1.1" | sudo tee -a /etc/resolv.conf

# Using systemd-resolved
sudo systemd-resolve --set-dns=8.8.8.8 --interface=eth0
```

---

## 📬 DHCP: Dynamic Host Configuration Protocol

**Automatically assigns IP addresses to devices.**

### DHCP Process (DORA):
```
D - Discover
O - Offer
R - Request
A - Acknowledge

Client                          DHCP Server
  │                               │
  │──── DHCPDISCOVER (broadcast)→ │  "I need an IP!"
  │                               │
  │ ←─── DHCPOFFER ──────────────│  "How about 192.168.1.100?"
  │                               │
  │──── DHCPREQUEST ────────────→ │  "Yes, I'll take it!"
  │                               │
  │ ←─── DHCPACK ────────────────│  "It's yours! Here's the details:"
  │                               │
  │     IP: 192.168.1.100         │
  │     Subnet: 255.255.255.0     │
  │     Gateway: 192.168.1.1      │
  │     DNS: 8.8.8.8              │
  │     Lease: 24 hours           │
```

### DHCP Information Provided:
```
✅ IP address
✅ Subnet mask
✅ Default gateway
✅ DNS servers
✅ Lease duration
✅ (Optional) NTP servers, domain name, etc.
```

### DHCP Commands:
```bash
# Request new IP (release and renew)
sudo dhclient -r eth0  # Release
sudo dhclient eth0     # Renew

# View DHCP lease info
cat /var/lib/dhcp/dhclient.leases

# Ubuntu (newer)
cat /var/lib/dhcp/dhclient.eth0.leases

# Manual renewal (systemd)
sudo systemctl restart NetworkManager
```

### DHCP Server Configuration:
```bash
# /etc/dhcp/dhcpd.conf example
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    default-lease-time 86400;
    max-lease-time 172800;
}
```

---

## 🔐 SSH: Secure Shell

**Secure remote access protocol (Port 22).**

### SSH Features:
```
✅ Encrypted connection
✅ Authentication (password or key)
✅ Remote command execution
✅ File transfer (SCP, SFTP)
✅ Port forwarding (tunneling)
```

### SSH Commands:
```bash
# Connect to remote server
ssh user@server.com

# Specify port
ssh -p 2222 user@server.com

# Copy file to remote
scp file.txt user@server.com:/path/

# Copy from remote
scp user@server.com:/path/file.txt .

# SSH key generation
ssh-keygen -t rsa -b 4096

# Copy SSH key to server
ssh-copy-id user@server.com

# SSH tunnel (local port forward)
ssh -L 8080:localhost:80 user@server.com
# Access remote port 80 via local port 8080
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Application Not Accessible
```bash
# Check if service is listening
ss -tln | grep :80

# Test HTTP locally
curl http://localhost:80

# Test from another machine
curl http://server-ip:80

# Check DNS resolution
dig myapp.example.com

# Trace network path
traceroute myapp.example.com
```

### Scenario 2: Database Connection Issues
```bash
# Check if DB is listening
ss -tln | grep :5432

# Test connection
telnet db-server 5432

# Check DNS
dig db-server.example.com

# Test with netcat
nc -zv db-server 5432
```

### Scenario 3: DHCP Troubleshooting
```bash
# No IP assigned?
ip addr show

# Release and renew
sudo dhclient -r eth0
sudo dhclient -v eth0

# Check for DHCP server response
sudo tcpdump -i eth0 port 67 or port 68
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the main difference between TCP and UDP?**
   <details>
   <summary>Answer</summary>
   TCP is reliable, connection-oriented, and ordered. UDP is fast, connectionless, and unreliable.
   </details>

2. **What does DNS do?**
   <details>
   <summary>Answer</summary>
   Translates domain names (google.com) to IP addresses (142.250.185.46).
   </details>

3. **What is DHCP used for?**
   <details>
   <summary>Answer</summary>
   Automatically assigns IP addresses and network configuration to devices.
   </details>

4. **What port does HTTPS use?**
   <details>
   <summary>Answer</summary>
   Port 443 (HTTP uses port 80)
   </details>

---

## 📝 Key Takeaways

✅ **TCP** = Reliable, ordered, connection-oriented (web, email, SSH)  
✅ **UDP** = Fast, connectionless, unreliable (DNS, streaming, gaming)  
✅ **HTTP** = Web protocol, runs on TCP port 80  
✅ **HTTPS** = Encrypted HTTP, port 443  
✅ **DNS** = Domain name to IP address translation  
✅ **DHCP** = Automatic IP address assignment  
✅ **SSH** = Secure remote access, port 22  
✅ Use `ss`, `curl`, `dig` to troubleshoot protocols  

---

## 🚀 Next Steps

You understand network protocols! Next, let's learn essential Linux networking commands for troubleshooting.

**Next lesson:** `13-linux-network-commands.md`

---

## 💡 Pro Tip

**Protocol troubleshooting order:**
```bash
# 1. Is service listening?
ss -tln | grep <port>

# 2. Can you connect locally?
curl http://localhost:<port>

# 3. Can you connect from another host?
curl http://<ip>:<port>

# 4. DNS resolving correctly?
dig <domain>

# 5. Firewall blocking?
sudo iptables -L -n | grep <port>
```

This workflow solves 80% of connectivity issues! 🎯
