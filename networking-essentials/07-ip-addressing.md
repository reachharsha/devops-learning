---
render_with_liquid: false
---
# 07 - IP Addressing

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What IP addresses are and why they exist
- IPv4 vs IPv6
- Public vs Private IP addresses
- IP address classes
- How to read and work with IP addresses

---

## 🤔 What is an IP Address?

**An IP address is a unique identifier for a device on a network.**

**Think of it like a house address:**
```
House Address:     123 Main Street, City, State, ZIP
IP Address:        192.168.1.100

Both tell you WHERE something is located!
```

### Without IP Addresses:
```
❌ "Send data to... that computer over there?"
❌ No way to identify specific devices
❌ No way to route across networks
```

### With IP Addresses:
```
✅ "Send data to 192.168.1.100"
✅ Unique identifier for each device
✅ Routers know how to forward traffic
```

---

## 🔢 IPv4 Addresses

**The most common IP version (for now)**

### Structure:
```
192.168.1.100
 │   │   │  │
 └───┴───┴──┴─── Four octets (numbers 0-255)

Separated by dots (periods)
Total: 32 bits (4 octets × 8 bits)
```

### Format:
```
Decimal:    192    .    168    .    1      .    100
Binary:  11000000 . 10101000 . 00000001 . 01100100
Bits:    ← 8 bits→   ← 8 bits→  ← 8 bits→  ← 8 bits→
```

### Valid Range:
```
Minimum: 0.0.0.0
Maximum: 255.255.255.255

Total possible addresses: 2^32 = 4,294,967,296
(About 4.3 billion addresses)
```

### Examples:
```
✅ 192.168.1.1       Valid
✅ 10.0.0.1          Valid
✅ 172.16.254.1      Valid
❌ 256.1.1.1         Invalid (256 > 255)
❌ 192.168.1         Invalid (only 3 octets)
❌ 192.168.1.1.1     Invalid (5 octets)
```

---

## 🌍 Public vs Private IP Addresses

### Public IP Addresses

**Routable on the internet - globally unique**

```
Your Home Router
Public IP: 203.0.113.45  ← Visible to the internet
        │
        │ Internet
        │
    Google Server
Public IP: 172.217.14.206
```

**Characteristics:**
- Assigned by ISP
- Unique across entire internet
- Can be reached from anywhere
- Costs money (sometimes)

**Example Public IPs:**
```
8.8.8.8         - Google DNS
1.1.1.1         - Cloudflare DNS
142.250.185.46  - Google.com
```

### Private IP Addresses

**NOT routable on internet - used internally**

```
Your Home Network (Private)
├── Router:    192.168.1.1
├── Laptop:    192.168.1.100
├── Phone:     192.168.1.101
└── Printer:   192.168.1.102

All use SAME private range!
```

**Three Private Ranges (RFC 1918):**
```
1. 10.0.0.0 to 10.255.255.255
   (10.0.0.0/8)
   - 16,777,216 addresses
   - Used by: Large networks, enterprises

2. 172.16.0.0 to 172.31.255.255
   (172.16.0.0/12)
   - 1,048,576 addresses
   - Used by: Medium networks

3. 192.168.0.0 to 192.168.255.255
   (192.168.0.0/16)
   - 65,536 addresses
   - Used by: Home networks, small offices
```

**Special Note:** These ranges can be reused!
```
Your Home:     192.168.1.100
Your Friend's: 192.168.1.100  ← SAME IP, different networks!
```

### How Private IPs Access Internet (NAT):
```
Inside (Private)              Outside (Public)
192.168.1.100  ───┐
192.168.1.101  ───┤
192.168.1.102  ───┤──► Router ──► 203.0.113.45 ──► Internet
192.168.1.103  ───┤     (NAT)
192.168.1.104  ───┘

Router translates private IPs to public IP
```

---

## 📝 Special IP Addresses

### 1. Loopback Address
```
127.0.0.1        - Localhost (IPv4)
::1              - Localhost (IPv6)
127.0.0.0/8      - Entire loopback range

Purpose: Communication within same computer
```

**Example:**
```bash
ping 127.0.0.1    # Always works (if networking is OK)
curl http://localhost:8080  # Access local service
```

### 2. Default Route
```
0.0.0.0          - "Any address" / "All addresses"

Used in routing: "Send everything to..."
```

### 3. Broadcast Address
```
192.168.1.255    - Broadcast to all in 192.168.1.0/24
255.255.255.255  - Limited broadcast (current network)

Purpose: Send packet to ALL devices
```

### 4. Network Address
```
192.168.1.0      - The network itself (not assignable)

First address in range = network identifier
```

### 5. APIPA (Automatic Private IP Addressing)
```
169.254.0.0 to 169.254.255.255

When DHCP fails, OS assigns itself an APIPA address
Example: 169.254.123.45

If you see this: ❌ DHCP is broken!
```

### 6. Multicast Addresses
```
224.0.0.0 to 239.255.255.255

For one-to-many communication
Example: 224.0.0.1 (all hosts on local network)
```

---

## 📚 IP Address Classes (Legacy)

**Historical context - not commonly used today, but good to know:**

```
Class A:  1.0.0.0    to 127.255.255.255   (Large networks)
         /8 networks (16 million hosts)
         First bit: 0

Class B:  128.0.0.0  to 191.255.255.255   (Medium networks)
         /16 networks (65,536 hosts)
         First bits: 10

Class C:  192.0.0.0  to 223.255.255.255   (Small networks)
         /24 networks (254 hosts)
         First bits: 110

Class D:  224.0.0.0  to 239.255.255.255   (Multicast)
         First bits: 1110

Class E:  240.0.0.0  to 255.255.255.255   (Reserved)
         First bits: 1111
```

**Today, we use CIDR (Classless) instead of classes!**

---

## 🆕 IPv6 Addresses

**The future of IP addressing (already here!)**

### Why IPv6?
```
IPv4: 4.3 billion addresses
World population: 8 billion people
IoT devices: Billions more

WE RAN OUT OF IPv4 ADDRESSES! 🚨
```

### IPv6 Structure:
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
 │    │    │    │    │    │    │    │
 └────┴────┴────┴────┴────┴────┴────┴── 8 groups of 4 hex digits

Total: 128 bits (compared to IPv4's 32 bits)
Addresses: 340 undecillion (340 trillion trillion trillion)
```

### Simplified Notation:
```
Full:       2001:0db8:85a3:0000:0000:8a2e:0370:7334
Compressed: 2001:db8:85a3::8a2e:370:7334
            └─────┘ Leading zeros removed
                   └─┘ :: replaces consecutive zeros
```

### IPv6 Address Types:
```
1. Unicast (One-to-one)
   Global:    2001:db8::1           (Like IPv4 public)
   Link-Local: fe80::1              (Like IPv4 169.254)
   Loopback:  ::1                   (Like 127.0.0.1)

2. Multicast (One-to-many)
   ff00::/8                         (Starts with ff)

3. Anycast (One-to-nearest)
   Same as unicast, special routing
```

### IPv6 Example:
```bash
# View IPv6 addresses
ip -6 addr show

# Output:
inet6 2001:db8::1/64 scope global
inet6 fe80::20c:29ff:fe3f:4a5b/64 scope link
```

---

## 🔀 IPv4 vs IPv6 Comparison

```
Feature          IPv4                    IPv6
──────────────────────────────────────────────────────
Address Size     32 bits                 128 bits
Format           192.168.1.1             2001:db8::1
Total Addresses  4.3 billion             340 undecillion
Header Size      Variable                Fixed (simpler)
Fragmentation    Routers & hosts         Hosts only
Broadcast        Yes                     No (uses multicast)
Configuration    DHCP or manual          Auto-config (SLAAC)
Security         Optional (IPsec)        Built-in (IPsec)
```

---

## 🛠️ Working with IP Addresses

### Checking Your IP Address:

**Linux:**
```bash
# IPv4 and IPv6
ip addr show

# Just IPv4
ip -4 addr show

# Just IPv6
ip -6 addr show

# Old method (still works)
ifconfig
```

**Example Output:**
```bash
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.100/24 brd 192.168.1.255 scope global eth0
    inet6 fe80::20c:29ff:fe3f:4a5b/64 scope link
```

### Assigning IP Address:

**Temporary (until reboot):**
```bash
# Add IPv4
sudo ip addr add 192.168.1.100/24 dev eth0

# Add IPv6
sudo ip addr add 2001:db8::1/64 dev eth0

# Remove
sudo ip addr del 192.168.1.100/24 dev eth0
```

**Permanent (configuration file):**
```bash
# Ubuntu (Netplan)
sudo nano /etc/netplan/01-netcfg.yaml

network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
        - 2001:db8::1/64
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

# Apply changes
sudo netplan apply
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Server Configuration
```bash
# Deploying a web server
# Need to assign static IP: 10.0.1.100

sudo ip addr add 10.0.1.100/24 dev eth0
sudo ip link set eth0 up
sudo ip route add default via 10.0.1.1

# Verify
curl http://10.0.1.100
```

### Scenario 2: Troubleshooting Connectivity
```bash
# Check if IP is assigned
ip addr show eth0

# No IP? Check DHCP
sudo dhclient eth0

# Still nothing? Check cable and interface status
ip link show eth0
```

### Scenario 3: Docker Networking
```bash
# Docker default network
docker network inspect bridge

# Shows:
"Subnet": "172.17.0.0/16"
"Gateway": "172.17.0.1"

# Containers get IPs from this range:
# 172.17.0.2, 172.17.0.3, 172.17.0.4, etc.
```

### Scenario 4: Cloud Networking
```
AWS EC2 Instance:
- Private IP:  10.0.1.50      (Inside VPC)
- Public IP:   203.0.113.45   (Internet-facing)

SSH via public: ssh user@203.0.113.45
Access DB via private: mysql -h 10.0.1.100
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between public and private IP addresses?**
   <details>
   <summary>Answer</summary>
   Public IPs are routable on the internet and globally unique. Private IPs are used internally and can be reused across different networks.
   </details>

2. **What are the three private IP ranges?**
   <details>
   <summary>Answer</summary>
   - 10.0.0.0/8
   - 172.16.0.0/12
   - 192.168.0.0/16
   </details>

3. **What is 127.0.0.1?**
   <details>
   <summary>Answer</summary>
   The loopback address (localhost) - used for communication within the same computer.
   </details>

4. **How many bits is an IPv4 address? IPv6?**
   <details>
   <summary>Answer</summary>
   IPv4: 32 bits, IPv6: 128 bits
   </details>

5. **If you see 169.254.x.x, what does it mean?**
   <details>
   <summary>Answer</summary>
   APIPA address - DHCP failed, and the system assigned itself an address.
   </details>

---

## 📊 IP Address Quick Reference

```
Type              Range                    Purpose
─────────────────────────────────────────────────────────
Private (Class A) 10.0.0.0/8              Large internal networks
Private (Class B) 172.16.0.0/12           Medium internal networks
Private (Class C) 192.168.0.0/16          Small internal networks
Loopback          127.0.0.0/8             Local communication
APIPA             169.254.0.0/16          DHCP failure
Multicast         224.0.0.0/4             One-to-many
Public            Everything else         Internet routing
```

---

## 📝 Key Takeaways

✅ IP addresses uniquely identify devices on networks  
✅ IPv4: 32 bits, dotted decimal (192.168.1.1)  
✅ IPv6: 128 bits, hexadecimal (2001:db8::1)  
✅ **Private ranges:** 10.x, 172.16-31.x, 192.168.x  
✅ **Loopback:** 127.0.0.1 (localhost)  
✅ Public IPs are internet-routable, private are not  
✅ NAT allows private IPs to access internet  
✅ IPv6 solves IPv4 address exhaustion  

---

## 🚀 Next Steps

You now understand IP addresses! Next, we'll learn how to divide networks into smaller subnets (subnetting).

**Next lesson:** `08-subnetting.md`

---

## 💡 Pro Tip

**Quick way to identify private vs public:**
```bash
Is the IP:
- 10.x.x.x?           → Private
- 172.(16-31).x.x?    → Private
- 192.168.x.x?        → Private
- 127.x.x.x?          → Loopback
- Everything else?    → Probably Public
```

Memorize the private ranges! 🎯
