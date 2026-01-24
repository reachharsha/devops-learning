---
render_with_liquid: false
---
# 13 - Essential Linux Networking Commands

## 🎯 Learning Objectives
By the end of this lesson, you'll master:
- ip command suite (modern)
- ping, traceroute, mtr
- ss and netstat
- dig and nslookup
- tcpdump and packet analysis
- Network performance tools

---

## 💡 **NEW vs OLD Commands**

```
OLD (deprecated)          NEW (use these!)
────────────────────────────────────────
ifconfig                  ip addr
route                     ip route
netstat                   ss
arp                       ip neigh
ifconfig eth0 up          ip link set eth0 up
route add                 ip route add
```

**Learn the NEW commands!** They're faster, more powerful, and actively maintained.

---

## 1️⃣ **ip** - The Swiss Army Knife

### View Network Interfaces:
```bash
# Show all interfaces
ip link show

# Show specific interface
ip link show eth0

# Brief format (easier to read)
ip -br link show

# Output:
lo      UNKNOWN  00:00:00:00:00:00 <LOOPBACK,UP,LOWER_UP>
eth0    UP       00:0c:29:3f:4a:5b <BROADCAST,MULTICAST,UP,LOWER_UP>
```

### View IP Addresses:
```bash
# All IP addresses
ip addr show

# Specific interface
ip addr show eth0

# Brief format
ip -br addr show

# Output:
eth0    UP    192.168.1.100/24 fe80::20c:29ff:fe3f:4a5b/64

# Only IPv4
ip -4 addr show

# Only IPv6
ip -6 addr show
```

### Manage IP Addresses:
```bash
# Add IP address
sudo ip addr add 192.168.1.150/24 dev eth0

# Remove IP address
sudo ip addr del 192.168.1.150/24 dev eth0

# Flush all IPs from interface
sudo ip addr flush dev eth0
```

### Manage Interfaces:
```bash
# Bring interface up
sudo ip link set eth0 up

# Bring interface down
sudo ip link set eth0 down

# Change MTU
sudo ip link set eth0 mtu 9000

# Change MAC address (spoofing)
sudo ip link set eth0 address aa:bb:cc:dd:ee:ff
```

### Routing Commands:
```bash
# Show routing table
ip route show

# Add default route
sudo ip route add default via 192.168.1.1 dev eth0

# Add specific route
sudo ip route add 10.0.0.0/8 via 192.168.1.254 dev eth0

# Delete route
sudo ip route del 10.0.0.0/8

# Show route to specific destination
ip route get 8.8.8.8

# Output:
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100
```

### ARP/Neighbor Table:
```bash
# Show ARP table
ip neigh show

# Flush ARP cache
sudo ip neigh flush all

# Add static ARP entry
sudo ip neigh add 192.168.1.100 lladdr aa:bb:cc:dd:ee:ff dev eth0

# Delete ARP entry
sudo ip neigh del 192.168.1.100 dev eth0
```

### Network Statistics:
```bash
# Show interface statistics
ip -s link show eth0

# Output:
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    RX: bytes  packets  errors  dropped overrun mcast
    1048576    8192     0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    524288     4096     0       0       0       0

# Detailed stats
ip -s -s link show eth0  # Double -s for even more detail
```

---

## 2️⃣ **ping** - Test Connectivity

### Basic Usage:
```bash
# Ping Google DNS
ping 8.8.8.8

# Output:
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=11.9 ms
```

### Useful Options:
```bash
# Send only 4 packets
ping -c 4 8.8.8.8

# Change packet size
ping -s 1000 8.8.8.8  # 1000 bytes

# Flood ping (root only - use carefully!)
sudo ping -f 8.8.8.8

# Set specific interval (default is 1 second)
ping -i 0.5 8.8.8.8  # Every 0.5 seconds

# Use specific interface
ping -I eth0 8.8.8.8

# Ping until success (useful in scripts)
ping -c 1 -W 1 8.8.8.8 && echo "Host is up!"

# IPv6 ping
ping6 2001:4860:4860::8888
```

### What Ping Tells You:
```
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=12.3 ms
                │          │       │       │
                │          │       │       └─ Round-trip time (latency)
                │          │       └───────── Time To Live (hops remaining)
                │          └───────────────── Sequence number
                └──────────────────────────── Responding IP
```

---

## 3️⃣ **traceroute / tracepath** - Trace Route

### Traceroute:
```bash
# Trace route to Google
traceroute 8.8.8.8

# Output:
 1  192.168.1.1 (192.168.1.1)     1.234 ms    ← Your router
 2  10.0.0.1 (10.0.0.1)           5.678 ms    ← ISP router
 3  72.14.204.1 (72.14.204.1)    12.345 ms    ← ISP backbone
 4  * * *                                      ← Router not responding to ICMP
 5  108.170.250.1                 15.678 ms
 6  8.8.8.8 (8.8.8.8)             18.901 ms    ← Destination!

# Don't resolve hostnames (faster)
traceroute -n 8.8.8.8

# Use ICMP instead of UDP
sudo traceroute -I 8.8.8.8

# Show AS numbers
traceroute -A 8.8.8.8
```

### Tracepath (no root required):
```bash
# Simpler traceroute alternative
tracepath 8.8.8.8

# Also shows MTU
tracepath -n 8.8.8.8
```

---

## 4️⃣ **mtr** - Better Than Traceroute

**MTR = My TraceRoute (combines ping + traceroute)**

```bash
# Install first
sudo apt install mtr  # Ubuntu/Debian
sudo yum install mtr  # RHEL/CentOS

# Real-time interactive view
mtr 8.8.8.8

# Output (live updating):
Host                Loss%  Snt  Last   Avg  Best  Wrst
1. 192.168.1.1       0.0%   10   1.2   1.3   1.1   1.5
2. 10.0.0.1          0.0%   10   5.6   5.8   5.5   6.2
3. 72.14.204.1       0.0%   10  12.3  12.5  12.1  13.0
4. 8.8.8.8           0.0%   10  18.9  19.1  18.5  20.3

# Report mode (non-interactive, 10 packets)
mtr -r -c 10 8.8.8.8

# JSON output (for scripts)
mtr --json 8.8.8.8

# TCP mode (useful when ICMP is blocked)
mtr --tcp -P 80 example.com
```

---

## 5️⃣ **ss** - Socket Statistics

**Modern replacement for netstat**

### Show All Connections:
```bash
# All sockets
ss

# All TCP connections
ss -t

# All UDP connections
ss -u

# Listening sockets only
ss -l

# TCP listening sockets
ss -tln

# Output:
State    Recv-Q Send-Q  Local Address:Port  Peer Address:Port
LISTEN   0      128     0.0.0.0:22           0.0.0.0:*
LISTEN   0      128     0.0.0.0:80           0.0.0.0:*
ESTAB    0      0       192.168.1.100:22     192.168.1.50:54321
```

### Useful Options:
```bash
# Show process using socket (-p)
sudo ss -tlnp

# Output includes PID:
LISTEN  0  128  0.0.0.0:80  0.0.0.0:*  users:(("nginx",pid=1234,fd=6))

# Show all established connections
ss -t state established

# Show numerical addresses (no DNS lookup)
ss -tn

# Show summary statistics
ss -s

# Filter by port
ss -tn sport = :80  # Source port 80
ss -tn dport = :443 # Destination port 443

# Show connections to specific IP
ss dst 8.8.8.8

# Show memory usage
ss -tm
```

### Connection States:
```
LISTEN      - Waiting for connections
ESTAB       - Established connection
SYN-SENT    - Trying to establish connection
SYN-RECV    - Received connection request
FIN-WAIT-1  - Closing connection
FIN-WAIT-2  - Closing connection
TIME-WAIT   - Waiting to ensure connection is closed
CLOSE-WAIT  - Waiting for close
LAST-ACK    - Final acknowledgment
CLOSED      - No connection
```

---

## 6️⃣ **dig** - DNS Lookup

### Basic DNS Query:
```bash
# Simple lookup
dig google.com

# Output (simplified):
;; ANSWER SECTION:
google.com.  300  IN  A  142.250.185.46

# Short answer only
dig +short google.com
# Output: 142.250.185.46

# Query specific record type
dig google.com A      # IPv4 address
dig google.com AAAA   # IPv6 address
dig google.com MX     # Mail servers
dig google.com NS     # Name servers
dig google.com TXT    # Text records
dig google.com SOA    # Start of Authority
```

### Advanced dig:
```bash
# Use specific DNS server
dig @8.8.8.8 google.com

# Reverse DNS lookup
dig -x 8.8.8.8
# Output: 8.8.8.8.in-addr.arpa. 3600 IN PTR dns.google.

# Trace DNS resolution path
dig +trace google.com

# No recursion (query only this server)
dig +norecurs google.com

# Show query time
dig google.com | grep "Query time"

# Multiple queries
dig google.com amazon.com microsoft.com

# Read queries from file
dig -f domains.txt
```

---

## 7️⃣ **nslookup** - Simple DNS Lookup

```bash
# Interactive mode
nslookup
> google.com
> exit

# Non-interactive
nslookup google.com

# Use specific DNS server
nslookup google.com 8.8.8.8

# Reverse lookup
nslookup 8.8.8.8

# Query specific record type
nslookup -type=MX google.com
nslookup -type=NS google.com
```

**Note:** `dig` is more powerful and preferred!

---

## 8️⃣ **tcpdump** - Packet Capture

**Capture and analyze network traffic**

### Basic Capture:
```bash
# Capture on eth0 (requires root)
sudo tcpdump -i eth0

# Output (scrolling):
12:34:56.789012 IP 192.168.1.100.54321 > 8.8.8.8.53: Flags [S], seq 123456...

# Capture 10 packets and stop
sudo tcpdump -i eth0 -c 10

# Save to file
sudo tcpdump -i eth0 -w capture.pcap

# Read from file
tcpdump -r capture.pcap
```

### Filters:
```bash
# Capture only HTTP traffic
sudo tcpdump -i eth0 port 80

# Capture specific host
sudo tcpdump -i eth0 host 8.8.8.8

# Capture specific network
sudo tcpdump -i eth0 net 192.168.1.0/24

# Capture TCP only
sudo tcpdump -i eth0 tcp

# Capture UDP only
sudo tcpdump -i eth0 udp

# Capture DNS queries
sudo tcpdump -i eth0 port 53

# Capture SSH traffic
sudo tcpdump -i eth0 port 22

# Combine filters (AND)
sudo tcpdump -i eth0 'host 8.8.8.8 and port 53'

# OR filter
sudo tcpdump -i eth0 'port 80 or port 443'

# NOT filter
sudo tcpdump -i eth0 'not port 22'
```

### Advanced Options:
```bash
# Verbose output
sudo tcpdump -i eth0 -v
sudo tcpdump -i eth0 -vv   # More verbose
sudo tcpdump -i eth0 -vvv  # Even more!

# Show packet contents in hex and ASCII
sudo tcpdump -i eth0 -X

# Don't resolve hostnames (faster)
sudo tcpdump -i eth0 -n

# Don't resolve ports
sudo tcpdump -i eth0 -nn

# Show ethernet headers
sudo tcpdump -i eth0 -e

# Capture with timestamp
sudo tcpdump -i eth0 -tttt
```

---

## 9️⃣ **nc (netcat)** - Network Swiss Army Knife

### Test Port Connectivity:
```bash
# Check if port is open
nc -zv example.com 80

# Output if open:
Connection to example.com 80 port [tcp/http] succeeded!

# Output if closed:
nc: connect to example.com port 81 (tcp) failed: Connection refused

# Scan port range
nc -zv example.com 20-80
```

### Simple Chat:
```bash
# Server (listen on port 1234)
nc -l 1234

# Client (connect to server)
nc server-ip 1234

# Now type messages on either side!
```

### File Transfer:
```bash
# Receiver
nc -l 1234 > received-file.txt

# Sender
nc receiver-ip 1234 < file-to-send.txt
```

### Simple Web Server:
```bash
# Serve single response
echo -e "HTTP/1.1 200 OK\n\nHello World" | nc -l 8080
```

---

## 🔟 **curl** - Transfer Data

### HTTP Requests:
```bash
# Simple GET
curl http://example.com

# Show headers only
curl -I http://example.com

# Verbose output (see full request/response)
curl -v http://example.com

# Follow redirects
curl -L http://example.com

# Save output to file
curl -o output.html http://example.com
curl -O http://example.com/file.zip  # Use remote filename

# POST request
curl -X POST -d "name=John&age=30" http://example.com/api

# POST JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"John","age":30}' \
  http://example.com/api

# Custom headers
curl -H "Authorization: Bearer token123" http://example.com/api

# Download with progress
curl --progress-bar -O http://example.com/largefile.zip

# Resume download
curl -C - -O http://example.com/largefile.zip

# Timeout
curl --connect-timeout 10 --max-time 30 http://example.com

# Test HTTPS certificate
curl -v https://example.com 2>&1 | grep SSL
```

---

## 1️⃣1️⃣ **iftop** - Network Bandwidth Monitor

```bash
# Install
sudo apt install iftop

# Monitor eth0
sudo iftop -i eth0

# Output (live updating):
192.168.1.100  =>  8.8.8.8        1.23Mb  512Kb  256Kb
                <=                512Kb  256Kb  128Kb

# Show port numbers
sudo iftop -P

# Don't resolve hostnames
sudo iftop -n

# Filter by network
sudo iftop -F 192.168.1.0/24
```

---

## 1️⃣2️⃣ **nmap** - Network Scanner

```bash
# Install
sudo apt install nmap

# Scan single host
nmap 192.168.1.1

# Scan network range
nmap 192.168.1.0/24

# Scan specific ports
nmap -p 80,443 192.168.1.1

# Scan port range
nmap -p 1-1000 192.168.1.1

# Service version detection
nmap -sV 192.168.1.1

# OS detection
sudo nmap -O 192.168.1.1

# Fast scan (top 100 ports)
nmap -F 192.168.1.1

# Aggressive scan
sudo nmap -A 192.168.1.1
```

---

## 📝 Quick Reference Cheat Sheet

```bash
# View network config
ip addr show
ip route show
ip neigh show

# Test connectivity
ping -c 4 <host>
traceroute <host>
mtr <host>

# Check ports
ss -tlnp                    # Listening TCP
ss -tn state established    # Active connections
nc -zv <host> <port>        # Test specific port

# DNS lookup
dig +short <domain>
dig @8.8.8.8 <domain>

# Packet capture
sudo tcpdump -i eth0 -nn host <ip>
sudo tcpdump -i eth0 port <port> -w file.pcap

# HTTP testing
curl -I <url>
curl -v <url>

# Network monitoring
sudo iftop -i eth0
sudo nethogs eth0
```

---

## 🎯 Quick Check: Do You Understand?

1. **What command shows listening ports?**
   <details>
   <summary>Answer</summary>
   `ss -tln` or `sudo ss -tlnp` (with process info)
   </details>

2. **How do you test if a port is open?**
   <details>
   <summary>Answer</summary>
   `nc -zv <host> <port>` or `telnet <host> <port>`
   </details>

3. **What's the modern replacement for ifconfig?**
   <details>
   <summary>Answer</summary>
   `ip addr` command
   </details>

4. **How do you capture only HTTP traffic?**
   <details>
   <summary>Answer</summary>
   `sudo tcpdump -i eth0 port 80`
   </details>

---

## 📝 Key Takeaways

✅ Use **ip** instead of ifconfig/route  
✅ Use **ss** instead of netstat  
✅ **ping** tests basic connectivity  
✅ **mtr** combines ping + traceroute  
✅ **dig** for DNS lookups  
✅ **ss -tlnp** shows listening services  
✅ **tcpdump** captures packets  
✅ **nc** tests port connectivity  
✅ **curl** for HTTP testing  

---

## 🚀 Next Steps

You've mastered the essential commands! Next, let's learn a systematic troubleshooting methodology.

**Next lesson:** `14-troubleshooting-methodology.md`

---

## 💡 Pro Tip

**Create command aliases:**
```bash
# Add to ~/.bashrc or ~/.zshrc
alias ports='sudo ss -tlnp'
alias myip='ip -br addr show'
alias routes='ip route show'
alias listening='sudo ss -tlnp | grep LISTEN'

# Reload
source ~/.bashrc
```

Save typing and work faster! 🚀
