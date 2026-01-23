# 06 - The OSI Model (DevOps-Focused)

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What the OSI model is and why it exists
- All 7 layers and their purposes
- How layers work together
- Real-world examples for each layer
- Why this matters for DevOps troubleshooting

---

## 🤔 What is the OSI Model?

**OSI (Open Systems Interconnection) Model** is a framework that explains how network communication works in 7 layers.

**Think of it like building a house:**
- Each layer does a specific job
- Each layer depends on the layer below it
- Each layer provides services to the layer above it

```
┌────────────────────────────────┐
│  7. Application  │  Your apps  │  ← What you see
├────────────────────────────────┤
│  6. Presentation │  Formatting │
├────────────────────────────────┤
│  5. Session      │  Connections│
├────────────────────────────────┤
│  4. Transport    │  Delivery   │  ← TCP/UDP
├────────────────────────────────┤
│  3. Network      │  Routing    │  ← IP addresses
├────────────────────────────────┤
│  2. Data Link    │  Switching  │  ← MAC addresses
├────────────────────────────────┤
│  1. Physical     │  Cables     │  ← Wires, signals
└────────────────────────────────┘
```

---

## 🎓 The 7 Layers Explained

### Mnemonic to Remember:
```
"Please Do Not Throw Sausage Pizza Away"

P hysical
D ata Link
N etwork
T ransport
S ession
P resentation
A pplication
```

---

## 1️⃣ Layer 1: Physical Layer

**The hardware layer - actual physical transmission of bits**

### What it does:
- Transmits raw bits (0s and 1s)
- Defines cables, voltages, signals
- No intelligence, just transmission

### Components:
```
- Ethernet cables
- Fiber optic cables
- WiFi radio waves
- USB cables
- Network hubs (old)
- Repeaters
```

### Real-World Example:
```
Sending bit '1':
Electrical signal: +5 volts
Light pulse: LED ON
Radio wave: Frequency shift

Sending bit '0':
Electrical signal: 0 volts
Light pulse: LED OFF
Radio wave: Different frequency
```

### DevOps Relevance:
```bash
# Check physical link
ethtool eth0 | grep "Link detected"
# Link detected: yes ✅

# Check cable speed
ethtool eth0 | grep Speed
# Speed: 1000Mb/s
```

**Troubleshooting:**
```
- Is the cable plugged in?
- Is the cable damaged?
- Are the port lights blinking?
```

---

## 2️⃣ Layer 2: Data Link Layer

**The local delivery layer - MAC addresses and switches**

### What it does:
- Organizes bits into frames
- Uses MAC addresses for local delivery
- Error detection
- Controls access to physical medium

### Components:
```
- Network Interface Cards (NICs)
- Switches
- Bridges
- MAC addresses
- Ethernet protocol
```

### Frame Structure:
```
┌──────────────────────────────────────┐
│ Destination MAC | Source MAC | Data  │
│  (6 bytes)      | (6 bytes)  | ...   │
└──────────────────────────────────────┘

Example:
Dest MAC: AA:BB:CC:DD:EE:FF
Src MAC:  11:22:33:44:55:66
```

### Real-World Example:
```
Computer A wants to send to Computer B (same network):

A: "I need to send data to IP 192.168.1.100"
ARP: "192.168.1.100's MAC is AA:BB:CC:DD:EE:FF"
A creates frame:
   [Dest MAC: AA:BB...] [Src MAC: 11:22...] [Data]
Switch reads MAC address and forwards ONLY to B
```

### DevOps Commands:
```bash
# View MAC address
ip link show eth0
# link/ether 00:0c:29:3f:4a:5b

# View ARP table (IP to MAC mapping)
ip neigh show
# 192.168.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE

# View switch MAC table (on managed switches)
show mac address-table
```

**Troubleshooting:**
```
- Is the MAC address correct?
- Is the switch port active?
- ARP table populated?
```

---

## 3️⃣ Layer 3: Network Layer

**The routing layer - IP addresses and routers**

### What it does:
- Logical addressing (IP addresses)
- Routing between different networks
- Packet forwarding
- Fragmentation and reassembly

### Components:
```
- IP addresses (IPv4/IPv6)
- Routers
- IP protocol
- ICMP (ping)
```

### Packet Structure:
```
┌──────────────────────────────────────┐
│ Source IP | Dest IP | TTL | Data     │
│ 10.0.1.5  |10.0.2.10| 64  | ...      │
└──────────────────────────────────────┘
```

### Real-World Example:
```
Sending packet from 192.168.1.10 to 8.8.8.8 (Google DNS):

1. Your computer: "8.8.8.8 is not on my network"
2. Sends to default gateway (router)
3. Router checks routing table
4. Router forwards to next hop
5. Repeat until packet reaches 8.8.8.8
```

### Routing Table Example:
```
Destination      Gateway         Interface
0.0.0.0/0        192.168.1.1     eth0      ← Default route
192.168.1.0/24   0.0.0.0         eth0      ← Local network
10.0.0.0/8       10.1.1.1        eth1      ← Another network
```

### DevOps Commands:
```bash
# View routing table
ip route show
# default via 192.168.1.1 dev eth0
# 192.168.1.0/24 dev eth0 proto kernel scope link

# Trace route
traceroute google.com
# Shows every router (hop) to destination

# Ping (ICMP - Layer 3 protocol)
ping 8.8.8.8
```

**Troubleshooting:**
```
- Can you ping the destination?
- Is routing table correct?
- Is the gateway reachable?
- Check TTL (Time To Live)
```

---

## 4️⃣ Layer 4: Transport Layer

**The delivery layer - TCP/UDP and ports**

### What it does:
- End-to-end communication
- Segmentation and reassembly
- Error correction (TCP)
- Flow control
- Port numbers

### Components:
```
- TCP (Reliable)
- UDP (Fast)
- Port numbers
```

### TCP vs UDP:
```
TCP (Transmission Control Protocol):
✅ Reliable - guarantees delivery
✅ Ordered - packets arrive in order
✅ Error checking
❌ Slower
Uses: HTTP, SSH, Database connections

UDP (User Datagram Protocol):
✅ Fast - no overhead
❌ Unreliable - may lose packets
❌ No ordering
Uses: DNS, Video streaming, Gaming
```

### Port Numbers:
```
Well-known ports (0-1023):
  22  - SSH
  80  - HTTP
  443 - HTTPS
  3306 - MySQL
  5432 - PostgreSQL

Registered ports (1024-49151):
  8080 - Alternative HTTP
  3000 - Node.js apps
  
Ephemeral ports (49152-65535):
  54321 - Client-side random port
```

### Real-World Example:
```
Web browser connecting to web server:

Client (Your PC):
  IP: 192.168.1.10
  Port: 54321 (random)
  ↓
Server (Website):
  IP: 203.0.113.5
  Port: 443 (HTTPS)

Connection: 192.168.1.10:54321 → 203.0.113.5:443
```

### DevOps Commands:
```bash
# Show listening ports
ss -tulpn
# tcp   LISTEN  0.0.0.0:22    (SSH)
# tcp   LISTEN  0.0.0.0:80    (HTTP)

# Check established connections
ss -tn
# ESTAB  192.168.1.10:54321  203.0.113.5:443

# Old command (still works)
netstat -tulpn
```

**Troubleshooting:**
```
- Is the port open?
- Is the application listening?
- Firewall blocking the port?
- Too many connections?
```

---

## 5️⃣ Layer 5: Session Layer

**The conversation layer - establishing and managing sessions**

### What it does:
- Establishes, maintains, terminates sessions
- Synchronization
- Dialog control

### Components:
```
- Session establishment
- Authentication tokens
- Session IDs
```

### Real-World Example:
```
You log into a website:

1. Session established (login)
2. Session ID: abc123xyz
3. Multiple requests use same session
4. Session terminated (logout)

Your browser sends: Cookie: sessionid=abc123xyz
Server knows: "This is the same user"
```

### DevOps Relevance:
```
- Database connections (connection pooling)
- SSH sessions
- API sessions with tokens
- WebSocket connections
```

**In practice, Layers 5-7 are often handled together by applications.**

---

## 6️⃣ Layer 6: Presentation Layer

**The formatting layer - data translation and encryption**

### What it does:
- Data encryption/decryption
- Data compression
- Character encoding
- Format conversion

### Components:
```
- SSL/TLS encryption
- Character encoding (UTF-8, ASCII)
- Data compression (gzip)
- Image formats (JPEG, PNG)
```

### Real-World Example:
```
HTTPS connection:

1. Browser: "Hello server, let's use TLS"
2. Encryption negotiated (Presentation layer)
3. Data encrypted before sending
4. Server decrypts received data

Plaintext:  "password123"
Encrypted:  "a7f3c9d1e2b4..."
```

### DevOps Commands:
```bash
# Check TLS/SSL certificate
openssl s_client -connect google.com:443

# View encryption details
curl -vI https://google.com 2>&1 | grep TLS
```

---

## 7️⃣ Layer 7: Application Layer

**The user layer - what you interact with**

### What it does:
- Network services to applications
- User interface
- Protocols for specific applications

### Components:
```
- HTTP/HTTPS (Web)
- SSH (Remote access)
- FTP (File transfer)
- SMTP (Email)
- DNS (Name resolution)
```

### Real-World Example:
```
Opening a website:

Browser (Application Layer):
  "GET /index.html HTTP/1.1
   Host: example.com"

DNS query (Application Layer):
  "What's the IP for example.com?"
  Response: "93.184.216.34"
```

### DevOps Commands:
```bash
# HTTP request
curl http://example.com

# DNS query
dig example.com

# SSH connection
ssh user@server

# Database query
mysql -h dbserver -u user -p
```

---

## 🔄 How Layers Work Together

### Sending Data (Encapsulation):
```
Application Layer (7):  "Send email"
    ↓ Add email headers
Presentation Layer (6): Encrypt with TLS
    ↓ Add TLS headers
Session Layer (5):      Maintain connection
    ↓
Transport Layer (4):    Add TCP header (port 25)
    ↓ Segment into chunks
Network Layer (3):      Add IP header (source/dest IP)
    ↓ Create packets
Data Link Layer (2):    Add Ethernet header (MAC addresses)
    ↓ Create frames
Physical Layer (1):     Convert to electrical signals
    ↓
    ══════════ TRANSMITTED ══════════
```

### Receiving Data (Decapsulation):
```
Physical Layer (1):     Receive electrical signals
    ↓ Convert to bits
Data Link Layer (2):    Check MAC address "Is this for me?"
    ↓ Remove Ethernet header
Network Layer (3):      Check IP address "Is this for me?"
    ↓ Remove IP header
Transport Layer (4):    Port 25? Send to email service
    ↓ Reassemble segments
Session Layer (5):      Match to existing session
    ↓
Presentation Layer (6): Decrypt TLS
    ↓ Remove TLS headers
Application Layer (7):  "New email received!"
```

---

## 💼 Why This Matters for DevOps

### Systematic Troubleshooting:

When something breaks, check layers bottom-up:

```
❌ "Application is down!"

Layer 1 (Physical):
  ✅ Cable connected? → YES

Layer 2 (Data Link):
  ✅ Interface up? → YES
  ✅ Switch port active? → YES

Layer 3 (Network):
  ✅ IP address assigned? → YES
  ✅ Can ping gateway? → YES
  ✅ Can ping destination? → YES

Layer 4 (Transport):
  ✅ Port open? → YES
  ❌ Application listening on port? → NO!
  
Found it! Application not running on correct port.
```

### Real DevOps Scenarios:

**Scenario 1: Web App Not Accessible**
```
Layer 7: curl shows "Connection refused"
Layer 4: Port 80 closed
Layer 3: IP reachable (ping works)
Layer 2: MAC address resolved
Layer 1: Cable connected

Issue: Firewall blocking port 80 (Layer 4)
```

**Scenario 2: Container Can't Reach Database**
```
Layer 7: Application error "Can't connect to DB"
Layer 4: Port 5432 timing out
Layer 3: IP not routable
Layer 2: No route to network

Issue: Docker network misconfiguration (Layer 3)
```

---

## 📊 OSI Model Quick Reference

```
Layer  Name          Protocol/Examples      DevOps Tools
─────────────────────────────────────────────────────────
7      Application   HTTP, SSH, DNS         curl, dig, ssh
6      Presentation  TLS/SSL, compression   openssl
5      Session       NetBIOS, RPC           n/a
4      Transport     TCP, UDP               ss, netstat
3      Network       IP, ICMP, routing      ping, traceroute, ip route
2      Data Link     Ethernet, MAC, ARP     ip link, ip neigh
1      Physical      Cables, signals        ethtool
```

---

## 🎯 Quick Check: Do You Understand?

1. **Which layer uses IP addresses?**
   <details>
   <summary>Answer</summary>
   Layer 3 (Network Layer)
   </details>

2. **Which layer uses MAC addresses?**
   <details>
   <summary>Answer</summary>
   Layer 2 (Data Link Layer)
   </details>

3. **Which layer do TCP and UDP operate at?**
   <details>
   <summary>Answer</summary>
   Layer 4 (Transport Layer)
   </details>

4. **If a cable is unplugged, which layer is the problem?**
   <details>
   <summary>Answer</summary>
   Layer 1 (Physical Layer)
   </details>

5. **Which layer does HTTP operate at?**
   <details>
   <summary>Answer</summary>
   Layer 7 (Application Layer)
   </details>

---

## 📝 Key Takeaways

✅ OSI has 7 layers, each with a specific job  
✅ **Layer 1:** Physical (cables, signals)  
✅ **Layer 2:** Data Link (MAC, switches)  
✅ **Layer 3:** Network (IP, routing)  
✅ **Layer 4:** Transport (TCP/UDP, ports)  
✅ **Layers 5-7:** Session, Presentation, Application  
✅ Data is encapsulated going down, decapsulated going up  
✅ Troubleshoot from bottom layer up  
✅ Most DevOps issues are Layer 3, 4, or 7  

---

## 🚀 Next Steps

Now you understand the framework of networking! Next, we'll dive deep into Layer 3: IP addressing.

**Next lesson:** `07-ip-addressing.md`

---

## 💡 Pro Tip

**The "TCP/IP Model" (4 layers) is also common:**
```
OSI Model          TCP/IP Model
─────────────      ────────────
Application   \
Presentation   } → Application
Session       /
Transport      → Transport
Network        → Internet
Data Link     \
Physical       } → Network Access
```

Both models describe the same thing - use whichever helps you understand! 🎯
