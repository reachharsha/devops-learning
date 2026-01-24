---
render_with_liquid: false
---
# 11 - NAT: Network Address Translation

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What NAT is and why it exists
- How NAT works
- Different types of NAT
- Port forwarding
- NAT in Docker and cloud environments

---

## 🤔 What is NAT?

**NAT (Network Address Translation) = Translating private IP addresses to public IP addresses (and vice versa).**

**Real-World Analogy:**
```
Company Receptionist:

External caller: "Can I speak to John?"
Receptionist: "Sure, transferring to extension 2156"

External caller sees: Company main number (555-1000)
John's actual number: Extension 2156

Receptionist = NAT Router
Main number = Public IP
Extensions = Private IPs
```

---

## 🌍 Why Does NAT Exist?

### Problem: IPv4 Address Exhaustion
```
IPv4 addresses: 4.3 billion
World population: 8 billion
Devices per person: Multiple (phone, laptop, tablet, IoT...)

WE DON'T HAVE ENOUGH PUBLIC IPs! 🚨
```

### Solution: NAT
```
One public IP serves many private IPs:

Internet sees: 203.0.113.5 (one public IP)
        ↓
    [NAT Router]
        ↓
Inside network (private IPs):
├── 192.168.1.10 (Laptop)
├── 192.168.1.11 (Phone)
├── 192.168.1.12 (Tablet)
├── 192.168.1.13 (Smart TV)
└── 192.168.1.14 (IoT device)

5 devices share 1 public IP!
```

---

## 🔄 How NAT Works

### Outbound Traffic (Private → Public)

**Step-by-Step:**

```
Step 1: Internal device sends packet
┌────────────────────────────────────┐
│ Source: 192.168.1.10:54321        │
│ Dest: 8.8.8.8:53 (Google DNS)     │
└────────────────────────────────────┘

Step 2: Packet reaches NAT router
Router thinks: "Internet doesn't know 192.168.1.10"
Router action: Translate!

Step 3: Router translates source
┌────────────────────────────────────┐
│ Source: 203.0.113.5:12345 ←──────│ New! (public IP)
│ Dest: 8.8.8.8:53                  │
└────────────────────────────────────┘

Router stores in NAT table:
Internal             External
192.168.1.10:54321 → 203.0.113.5:12345

Step 4: Packet sent to internet
Google sees: 203.0.113.5:12345
(Doesn't know about 192.168.1.10)
```

### Inbound Traffic (Public → Private)

```
Step 1: Google replies
┌────────────────────────────────────┐
│ Source: 8.8.8.8:53                │
│ Dest: 203.0.113.5:12345           │
└────────────────────────────────────┘

Step 2: Router checks NAT table
Router finds: 203.0.113.5:12345 → 192.168.1.10:54321

Step 3: Router translates destination
┌────────────────────────────────────┐
│ Source: 8.8.8.8:53                │
│ Dest: 192.168.1.10:54321 ←───────│ Translated!
└────────────────────────────────────┘

Step 4: Packet delivered to internal device
Laptop receives response!
```

### Visual Flow:
```
Internal Device                Router (NAT)              Internet
192.168.1.10                  203.0.113.5               8.8.8.8

Request →
[192.168.1.10:54321]     →    Translate    →    [203.0.113.5:12345]
to 8.8.8.8:53                 source              to 8.8.8.8:53

                              Store mapping:
                              192.168.1.10:54321 ↔ 203.0.113.5:12345

Response ←
[192.168.1.10:54321]     ←    Translate    ←    [203.0.113.5:12345]
from 8.8.8.8:53               destination         from 8.8.8.8:53
```

---

## 🎭 Types of NAT

### 1. Static NAT (One-to-One)

**One private IP always maps to one specific public IP.**

```
Private IP        Public IP
192.168.1.10  →   203.0.113.10
192.168.1.11  →   203.0.113.11
192.168.1.12  →   203.0.113.12

Each device has its own public IP
```

**Use Cases:**
- Web servers
- Mail servers
- Public-facing services

**Configuration Example (iptables):**
```bash
# Static NAT: 192.168.1.10 ↔ 203.0.113.10
sudo iptables -t nat -A PREROUTING -d 203.0.113.10 -j DNAT --to-destination 192.168.1.10
sudo iptables -t nat -A POSTROUTING -s 192.168.1.10 -j SNAT --to-source 203.0.113.10
```

### 2. Dynamic NAT (Pool)

**Multiple private IPs share a pool of public IPs.**

```
Private IPs          Public IP Pool
192.168.1.10   ↗     203.0.113.10
192.168.1.11   →     203.0.113.11  (assigned dynamically)
192.168.1.12   ↘     203.0.113.12

First-come, first-served from pool
```

**Use Case:**
- Medium-sized companies
- More devices than public IPs

### 3. PAT/NAT Overload (Most Common)

**PAT = Port Address Translation (aka NAT Overload)**

**Many private IPs share ONE public IP using different ports.**

```
This is what your home router does!

Internal                    External (NAT Table)
192.168.1.10:54321    →    203.0.113.5:10001
192.168.1.10:54322    →    203.0.113.5:10002
192.168.1.11:34567    →    203.0.113.5:10003
192.168.1.12:45678    →    203.0.113.5:10004

All use same public IP (203.0.113.5)
Different ports distinguish connections
```

**NAT Table Example:**
```
Inside Local        Inside Global         Outside
192.168.1.10:54321  203.0.113.5:10001    8.8.8.8:53
192.168.1.11:34567  203.0.113.5:10002    1.1.1.1:53
192.168.1.12:45678  203.0.113.5:10003    142.250.185.46:80
```

---

## 🔓 Port Forwarding

**Allowing external connections to reach internal services.**

### The Problem:
```
You run a web server at 192.168.1.100:80
Internet can only see 203.0.113.5
How do external users reach your server?

Answer: Port Forwarding!
```

### How It Works:
```
Internet User
    ↓
Requests: 203.0.113.5:8080
    ↓
Router (Port Forward Rule):
    "Traffic to 203.0.113.5:8080 → 192.168.1.100:80"
    ↓
Web Server: 192.168.1.100:80 receives request
```

### Configuration Example (iptables):
```bash
# Forward external port 8080 to internal 192.168.1.100:80
sudo iptables -t nat -A PREROUTING -p tcp --dport 8080 -j DNAT --to-destination 192.168.1.100:80
sudo iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
```

### Common Port Forwards:
```
External Port  →  Internal IP:Port       Service
─────────────────────────────────────────────────
22             →  192.168.1.100:22      SSH
80             →  192.168.1.100:80      HTTP
443            →  192.168.1.100:443     HTTPS
3389           →  192.168.1.101:3389    RDP
25565          →  192.168.1.102:25565   Minecraft
```

---

## 🐳 NAT in Docker

Docker uses NAT extensively for container networking.

### Docker NAT Flow:
```
Container (172.17.0.2)
    ↓
Docker bridge (docker0: 172.17.0.1)
    ↓
NAT Translation
    ↓
Host interface (eth0: 192.168.1.100)
    ↓
Internet
```

### View Docker NAT Rules:
```bash
# Docker creates these automatically
sudo iptables -t nat -L -n -v | grep docker

# Example output:
Chain POSTROUTING (policy ACCEPT)
MASQUERADE  all  --  172.17.0.0/16  0.0.0.0/0

# This means: NAT all traffic from containers
```

### Docker Port Publishing:
```bash
# Run container with port forward
docker run -d -p 8080:80 nginx

# Creates NAT rule:
# External 8080 → Container 80

# View the rule
sudo iptables -t nat -L -n | grep 8080

# Output:
DNAT  tcp  --  0.0.0.0/0  0.0.0.0/0  tcp dpt:8080 to:172.17.0.2:80
```

---

## ☁️ NAT in Cloud Environments

### AWS NAT Gateway:
```
Private Subnet (10.0.1.0/24)
    ↓
NAT Gateway (10.0.2.5 with Elastic IP)
    ↓
Internet Gateway
    ↓
Internet

Private instances access internet via NAT Gateway
Internet cannot initiate connections to private instances
```

### Azure NAT:
```
VM in private subnet
    ↓
Azure NAT Gateway
    ↓
Public IP
    ↓
Internet
```

---

## 🛠️ Configuring NAT on Linux

### Enable IP Forwarding:
```bash
# Temporary
sudo sysctl -w net.ipv4.ip_forward=1

# Permanent
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Basic NAT (Masquerade):
```bash
# NAT all traffic from 192.168.1.0/24 going out eth0
sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# View NAT rules
sudo iptables -t nat -L -n -v
```

### SNAT (Source NAT):
```bash
# Specify exact source IP
sudo iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 203.0.113.5
```

### DNAT (Destination NAT):
```bash
# Port forward
sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.100:80
```

### Save NAT Rules:
```bash
# Ubuntu/Debian
sudo apt install iptables-persistent
sudo netfilter-persistent save

# RHEL/CentOS
sudo service iptables save
```

---

## 🔍 Viewing NAT Connections

### Active NAT Translations:
```bash
# View connection tracking
sudo conntrack -L

# Filter by protocol
sudo conntrack -L -p tcp

# Example output:
tcp  6 431999 ESTABLISHED src=192.168.1.10 dst=8.8.8.8 
     sport=54321 dport=53 src=8.8.8.8 dst=203.0.113.5 
     sport=53 dport=12345 [ASSURED]

Shows both original and translated addresses
```

### NAT Statistics:
```bash
# View NAT table
sudo iptables -t nat -L -n -v

# Output shows:
# - Packet counts
# - Byte counts
# - Rules hit
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Accessing Private Servers
```bash
# Problem: Web app in private subnet can't reach internet

# Solution: NAT Gateway
# Configure NAT on gateway server:
sudo iptables -t nat -A POSTROUTING -s 10.0.1.0/24 -o eth0 -j MASQUERADE

# Web app can now reach internet (for updates, APIs, etc.)
```

### Scenario 2: Exposing Container Services
```bash
# Container running on port 3000
# Want to access via host port 80

docker run -d -p 80:3000 myapp

# Docker creates NAT rule automatically
# External:80 → Container:3000
```

### Scenario 3: Home Lab Behind Router
```bash
# Running services at home
# Need external access

# Port forwards on router:
External 2222 → Internal 192.168.1.100:22   (SSH)
External 8080 → Internal 192.168.1.100:80   (Web)
External 8443 → Internal 192.168.1.100:443  (HTTPS)
```

### Scenario 4: Kubernetes NodePort
```bash
# Kubernetes NodePort service
# Creates NAT from node IP:NodePort → Pod IP:ContainerPort

# Example:
kubectl expose deployment myapp --type=NodePort --port=80

# Creates: NodeIP:30080 → PodIP:80
# NAT handled by kube-proxy
```

---

## ⚠️ NAT Limitations

### 1. Breaks End-to-End Connectivity
```
❌ External users can't initiate connections
❌ Some protocols don't work well (FTP, SIP)
❌ P2P applications struggle
```

### 2. Complicates Troubleshooting
```
❌ Logs show translated IPs, not original
❌ Hard to track which internal device
```

### 3. Performance Overhead
```
❌ Router must track every connection
❌ Additional processing delay
❌ Memory for connection table
```

### 4. Port Exhaustion
```
❌ Max ~65,000 ports per public IP
❌ High-traffic scenarios can run out
```

---

## 🎯 Quick Check: Do You Understand?

1. **What problem does NAT solve?**
   <details>
   <summary>Answer</summary>
   IPv4 address exhaustion. Allows multiple private IPs to share one or more public IPs.
   </details>

2. **What's the most common type of NAT?**
   <details>
   <summary>Answer</summary>
   PAT (Port Address Translation / NAT Overload) - many private IPs share one public IP using different ports.
   </details>

3. **What is port forwarding?**
   <details>
   <summary>Answer</summary>
   Directing incoming traffic on a specific port to an internal IP and port. Allows external access to internal services.
   </details>

4. **How does Docker use NAT?**
   <details>
   <summary>Answer</summary>
   Docker uses NAT to allow containers (private IPs) to access the internet through the host's IP. Port publishing creates port forwards.
   </details>

---

## 📝 Key Takeaways

✅ NAT translates private IPs to public IPs  
✅ Solves IPv4 address exhaustion  
✅ **PAT** = many devices share one public IP (different ports)  
✅ **Port forwarding** = allow external access to internal services  
✅ Docker and Kubernetes use NAT extensively  
✅ Enable with `iptables -t nat` and IP forwarding  
✅ `conntrack` shows active NAT connections  
✅ NAT is transparent to applications (mostly)  

---

## 🚀 Next Steps

You understand NAT! Next, let's learn about network protocols - TCP, UDP, HTTP, DNS, and more.

**Next lesson:** `12-protocols.md`

---

## 💡 Pro Tip

**Quick NAT check on Linux:**
```bash
# 1. Is IP forwarding enabled?
cat /proc/sys/net/ipv4/ip_forward
# Should be: 1

# 2. View NAT rules
sudo iptables -t nat -L -n -v

# 3. Check active connections
sudo conntrack -L | grep <IP>

# 4. Test connectivity
# From inside network, try to reach internet
ping 8.8.8.8
```

NAT issues are common in container/cloud environments! 🎯
