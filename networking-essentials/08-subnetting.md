---
render_with_liquid: false
---
# 08 - Subnetting Made Simple

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What subnetting is and why it's used
- How subnet masks work
- CIDR notation (/24, /16, etc.)
- How to calculate subnets
- Practical subnetting for DevOps

---

## 🤔 What is Subnetting?

**Subnetting is dividing a large network into smaller sub-networks.**

**Real-World Analogy:**
```
Big Apartment Building (Network):
├── Floor 1 (Subnet 1) - Marketing Dept
├── Floor 2 (Subnet 2) - Engineering Dept
├── Floor 3 (Subnet 3) - Sales Dept
└── Floor 4 (Subnet 4) - HR Dept

Each floor has its own range of apartment numbers!
```

### Why Subnet?
```
✅ Better organization
✅ Improved security (isolate departments)
✅ Reduced broadcast traffic
✅ Efficient IP address usage
✅ Easier troubleshooting
```

---

## 🎭 Subnet Mask: The Divider

**A subnet mask defines which part of an IP is the network and which is the host.**

### Visual Representation:
```
IP Address:    192  .  168  .   1   .  100
Subnet Mask:   255  .  255  .  255  .   0
               └────Network─────┘  └─Host─┘

This means:
- Network portion: 192.168.1
- Host portion: 100
- All devices with 192.168.1.x are on the same network
```

### In Binary:
```
IP Address:    11000000.10101000.00000001.01100100
Subnet Mask:   11111111.11111111.11111111.00000000
               └────────Network Bits────┘└Host Bits┘

Rule:
- 1 in subnet mask = Network bit
- 0 in subnet mask = Host bit
```

---

## 📏 CIDR Notation (Slash Notation)

**CIDR = Classless Inter-Domain Routing**

Instead of writing `255.255.255.0`, we use `/24`

### Format:
```
192.168.1.0/24
            ││
            │└─ Number of network bits
            └── Pronounced "slash 24"
```

### Common Subnet Masks:
```
CIDR  Subnet Mask        Binary                            Hosts
───────────────────────────────────────────────────────────────────
/8    255.0.0.0          11111111.00000000.00000000.00000000  16,777,214
/16   255.255.0.0        11111111.11111111.00000000.00000000  65,534
/24   255.255.255.0      11111111.11111111.11111111.00000000  254
/25   255.255.255.128    11111111.11111111.11111111.10000000  126
/26   255.255.255.192    11111111.11111111.11111111.11000000  62
/27   255.255.255.224    11111111.11111111.11111111.11100000  30
/28   255.255.255.240    11111111.11111111.11111111.11110000  14
/29   255.255.255.252    11111111.11111111.11111111.11111100  6
/30   255.255.255.252    11111111.11111111.11111111.11111100  2
/32   255.255.255.255    11111111.11111111.11111111.11111111  1 (host route)
```

### Quick Memory Aid:
```
/24 = 256 addresses (254 usable)   ← Most common for small networks
/16 = 65,536 addresses             ← Medium networks
/8  = 16,777,216 addresses         ← Very large networks
```

---

## 🧮 Calculating Hosts

### Formula:
```
Number of hosts = 2^(host bits) - 2

Why subtract 2?
- Network address (first IP)
- Broadcast address (last IP)
```

### Examples:

**Example 1: /24 Network**
```
192.168.1.0/24

Network bits: 24
Host bits: 32 - 24 = 8

Hosts = 2^8 - 2 = 256 - 2 = 254 usable hosts

Range:
- Network:     192.168.1.0    ❌ Not assignable
- First host:  192.168.1.1    ✅ Can assign
- Last host:   192.168.1.254  ✅ Can assign
- Broadcast:   192.168.1.255  ❌ Not assignable
```

**Example 2: /16 Network**
```
172.16.0.0/16

Host bits: 32 - 16 = 16
Hosts = 2^16 - 2 = 65,536 - 2 = 65,534

Range:
- Network:     172.16.0.0
- First host:  172.16.0.1
- Last host:   172.16.255.254
- Broadcast:   172.16.255.255
```

**Example 3: /30 Network (Point-to-Point)**
```
10.0.0.0/30

Host bits: 32 - 30 = 2
Hosts = 2^2 - 2 = 4 - 2 = 2 hosts

Perfect for router-to-router links!

Range:
- Network:     10.0.0.0      ❌
- Host 1:      10.0.0.1      ✅ Router A
- Host 2:      10.0.0.2      ✅ Router B
- Broadcast:   10.0.0.3      ❌
```

---

## 🔢 Subnet Mask to CIDR Conversion

### Method: Count the 1s

```
255.255.255.0
= 11111111.11111111.11111111.00000000
  └─ 8 ─┘ └─ 8 ─┘ └─ 8 ─┘ └─ 0 ─┘
  8 + 8 + 8 + 0 = 24

Answer: /24
```

### Quick Reference Table:
```
Last Octet  Binary     CIDR  Hosts
─────────────────────────────────────
0           00000000   /24   254
128         10000000   /25   126
192         11000000   /26   62
224         11100000   /27   30
240         11110000   /28   14
248         11111000   /29   6
252         11111100   /30   2
255         11111111   /32   1
```

---

## 🎯 Practical Subnetting Examples

### Example 1: Office Network Design

**Requirements:**
- Marketing: 50 users
- Engineering: 100 users
- Sales: 30 users
- Management: 10 users

**Solution:**
```
Given: 192.168.1.0/24

Subnet 1 - Engineering (100 users = need /25):
- Network:    192.168.1.0/25
- Range:      192.168.1.1 - 192.168.1.126
- Hosts:      126

Subnet 2 - Marketing (50 users = need /26):
- Network:    192.168.1.128/26
- Range:      192.168.1.129 - 192.168.1.190
- Hosts:      62

Subnet 3 - Sales (30 users = need /27):
- Network:    192.168.1.192/27
- Range:      192.168.1.193 - 192.168.1.222
- Hosts:      30

Subnet 4 - Management (10 users = need /28):
- Network:    192.168.1.224/28
- Range:      192.168.1.225 - 192.168.1.238
- Hosts:      14
```

### Example 2: Docker Default Network
```
Docker default bridge: 172.17.0.0/16

Available IPs: 2^16 - 2 = 65,534
Container IPs: 172.17.0.2 - 172.17.255.254

This is why Docker can run thousands of containers!
```

### Example 3: AWS VPC Subnetting
```
VPC: 10.0.0.0/16

Public Subnet 1:  10.0.1.0/24   (254 hosts)
Public Subnet 2:  10.0.2.0/24   (254 hosts)
Private Subnet 1: 10.0.10.0/24  (254 hosts)
Private Subnet 2: 10.0.11.0/24  (254 hosts)
Database Subnet:  10.0.20.0/24  (254 hosts)
```

---

## 🛠️ Determining if IPs are on Same Network

### Method: AND Operation with Subnet Mask

**Question:** Are 192.168.1.50 and 192.168.1.100 on the same /24 network?

```
IP 1:          192.168.1.50
Binary:        11000000.10101000.00000001.00110010

Subnet /24:    255.255.255.0
Binary:        11111111.11111111.11111111.00000000

AND result:    11000000.10101000.00000001.00000000
Network:       192.168.1.0   ← Network address

IP 2:          192.168.1.100
Binary:        11000000.10101000.00000001.01100100

AND /24:       11111111.11111111.11111111.00000000

AND result:    11000000.10101000.00000001.00000000
Network:       192.168.1.0   ← Same network address!

Answer: YES, they're on the same network ✅
```

**Question:** Are 192.168.1.50/26 and 192.168.1.100/26 on the same network?

```
IP 1: 192.168.1.50
Binary: 11000000.10101000.00000001.00110010

Subnet /26: 255.255.255.192
Binary:     11111111.11111111.11111111.11000000

AND:        11000000.10101000.00000001.00000000
Network:    192.168.1.0

IP 2: 192.168.1.100
Binary: 11000000.10101000.00000001.01100100

AND /26:    11111111.11111111.11111111.11000000

AND:        11000000.10101000.00000001.01000000
Network:    192.168.1.64

Answer: NO, different networks! ❌
- 50 is in 192.168.1.0/26   (0-63)
- 100 is in 192.168.1.64/26 (64-127)
```

---

## 🔍 Subnetting Commands in Linux

### Check Your Subnet:
```bash
# View IP and subnet
ip addr show eth0

# Output:
inet 192.168.1.100/24 brd 192.168.1.255

# Breakdown:
# IP: 192.168.1.100
# Subnet: /24
# Broadcast: 192.168.1.255
```

### Calculate Subnet Info:
```bash
# Using ipcalc (install if needed: sudo apt install ipcalc)
ipcalc 192.168.1.100/24

# Output:
Address:   192.168.1.100
Netmask:   255.255.255.0 = 24
Wildcard:  0.0.0.255
Network:   192.168.1.0/24
Broadcast: 192.168.1.255
HostMin:   192.168.1.1
HostMax:   192.168.1.254
Hosts/Net: 254
```

### Find Network Address:
```bash
# Quick one-liner
ip addr show eth0 | grep 'inet ' | awk '{print $2}' | cut -d'/' -f1

# Or use ip route
ip route | grep 'proto kernel'
# 192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Container Networking
```bash
# Docker creates /16 networks by default
docker network create --subnet=172.18.0.0/16 my_network

# Gives you 65,534 possible container IPs
# vs
docker network create --subnet=172.18.0.0/24 my_network

# Only 254 possible container IPs
```

### Scenario 2: Troubleshooting Connectivity
```bash
# Problem: Can't reach database at 10.0.1.50

# Check if it's on your subnet
ip addr show
# inet 10.0.2.100/24

# Different subnets! (10.0.1.0 vs 10.0.2.0)
# Need a router or misconfigured IP
```

### Scenario 3: Security Group Rules
```
AWS Security Group:
Allow inbound from 10.0.1.0/24

This means:
✅ 10.0.1.1 - allowed
✅ 10.0.1.50 - allowed
✅ 10.0.1.254 - allowed
❌ 10.0.2.1 - denied (different subnet)
```

### Scenario 4: Kubernetes Pod CIDR
```bash
# Kubernetes cluster network
kubeadm init --pod-network-cidr=10.244.0.0/16

# Each node gets a /24 subnet:
Node1: 10.244.0.0/24  (254 pods)
Node2: 10.244.1.0/24  (254 pods)
Node3: 10.244.2.0/24  (254 pods)
...
```

---

## 📊 Common Subnet Sizes

```
Prefix  Hosts   Use Case
──────────────────────────────────────────────────────
/32     1       Single host route (specific IP)
/30     2       Point-to-point links (2 routers)
/29     6       Very small network
/28     14      Tiny network (printer subnet)
/27     30      Small department
/26     62      Medium department
/25     126     Large department
/24     254     Standard small network (home/office)
/23     510     2× /24 networks combined
/22     1,022   Small company
/21     2,046   Medium company
/20     4,094   Larger company
/16     65,534  Very large network, AWS VPC default
/8      16.7M   Massive network (Class A)
```

---

## 🎯 Subnetting Cheat Sheet

### Quick Calculations:

**Given IP and CIDR, find:**

1. **Number of hosts:**
   ```
   Hosts = 2^(32 - CIDR) - 2
   
   /24: 2^(32-24) - 2 = 2^8 - 2 = 254
   /16: 2^(32-16) - 2 = 2^16 - 2 = 65,534
   ```

2. **Subnet mask from CIDR:**
   ```
   /24 = 255.255.255.0
   /16 = 255.255.0.0
   /8  = 255.0.0.0
   ```

3. **Network address:**
   ```
   Apply subnet mask to IP (AND operation)
   ```

4. **Broadcast address:**
   ```
   All host bits set to 1
   ```

---

## 🎯 Quick Check: Do You Understand?

1. **What does /24 mean?**
   <details>
   <summary>Answer</summary>
   24 network bits, 8 host bits. Subnet mask: 255.255.255.0. Supports 254 hosts.
   </details>

2. **How many usable hosts in a /26 network?**
   <details>
   <summary>Answer</summary>
   2^(32-26) - 2 = 2^6 - 2 = 62 hosts
   </details>

3. **What's the network address of 192.168.1.130/25?**
   <details>
   <summary>Answer</summary>
   192.168.1.128/25 (because /25 splits .0-.127 and .128-.255)
   </details>

4. **What subnet mask is /28?**
   <details>
   <summary>Answer</summary>
   255.255.255.240
   </details>

---

## 📝 Key Takeaways

✅ Subnetting divides networks into smaller segments  
✅ Subnet mask determines network vs host portions  
✅ CIDR notation (/24) is shorthand for subnet mask  
✅ Formula: Hosts = 2^(host bits) - 2  
✅ /24 = 254 hosts (most common)  
✅ /30 = 2 hosts (router links)  
✅ Network address = first IP (not usable)  
✅ Broadcast address = last IP (not usable)  
✅ Use `ipcalc` tool for quick calculations  

---

## 🚀 Next Steps

You can now subnet like a pro! Next, let's learn how packets find their way across networks (routing).

**Next lesson:** `09-routing.md`

---

## 💡 Pro Tip

**Memorize these common subnets:**
```
/24 = 254 hosts    (home/small office)
/16 = 65,534 hosts (enterprise/AWS VPC)
/8  = 16M hosts    (ISP/very large)
/30 = 2 hosts      (router connections)
```

Everything else you can calculate! 🎯
