# 10 - Switching and MAC Addresses

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What MAC addresses are
- How switches work vs hubs
- MAC address tables
- ARP (Address Resolution Protocol)
- VLANs (Virtual LANs)

---

## 🔖 What is a MAC Address?

**MAC (Media Access Control) Address = Hardware address burned into your network card.**

**Think of it like a serial number:**
```
Social Security Number → Unique to you (doesn't change)
MAC Address → Unique to NIC (doesn't change*)

*Can be spoofed, but that's the hardware default
```

### MAC Address Format:
```
00:1A:2B:3C:4D:5E
││ ││ ││ ││ ││ ││
││ ││ ││ ││ ││ └└─ Device-specific
││ ││ ││ └└─────── Device-specific
││ ││ └└─────────── Device-specific
└└─└└──────────────── Manufacturer (OUI)

48 bits = 6 bytes = 12 hex digits
Separated by colons (:) or dashes (-)
```

### Example MAC Addresses:
```
00:0C:29:3F:4A:5B   (VMware virtual NIC)
00:1A:2B:3C:4D:5E   (Physical NIC)
FF:FF:FF:FF:FF:FF   (Broadcast - send to all)
```

### View Your MAC Address:
```bash
# Linux
ip link show eth0

# Output:
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    link/ether 00:0c:29:3f:4a:5b brd ff:ff:ff:ff:ff:ff

# Or shorter
ip link show eth0 | grep link/ether

# Old way
ifconfig eth0 | grep ether
```

---

## 🔀 Hub vs Switch: The Difference

### Hub (Dumb Device - Layer 1)
```
        [HUB]
       /  |  \
      /   |   \
    PC1  PC2  PC3

PC1 sends data to PC2:
1. Hub receives on port 1
2. Hub BROADCASTS to ALL ports
3. PC2 AND PC3 receive it
4. PC3 ignores it (not for me)
```

**Problems:**
```
❌ Everyone sees all traffic (security risk)
❌ Wastes bandwidth
❌ Collisions (only one can talk at a time)
❌ Slow!
```

### Switch (Smart Device - Layer 2)
```
        [SWITCH]
       /  |  \
      /   |   \
    PC1  PC2  PC3
    
PC1 sends data to PC2:
1. Switch receives on port 1
2. Switch checks MAC address table
3. Switch sends ONLY to port 2
4. PC3 never sees the traffic
```

**Benefits:**
```
✅ Efficient (sends only where needed)
✅ Secure (devices only see their traffic)
✅ Full-duplex (send and receive simultaneously)
✅ Fast!
```

---

## 📚 How Switches Learn: MAC Address Table

**Switches build a table mapping MAC addresses to ports.**

### Learning Process:

**Step 1: Switch starts empty**
```
Port  MAC Address
───────────────────
1     ?
2     ?
3     ?
```

**Step 2: PC1 (MAC: AA:AA:AA:AA:AA:AA) sends frame**
```
Frame arrives on Port 1
Source MAC: AA:AA:AA:AA:AA:AA

Switch learns:
Port  MAC Address
───────────────────
1     AA:AA:AA:AA:AA:AA  ← Learned!
2     ?
3     ?
```

**Step 3: PC2 (MAC: BB:BB:BB:BB:BB:BB) replies**
```
Frame arrives on Port 2
Source MAC: BB:BB:BB:BB:BB:BB

Switch learns:
Port  MAC Address
───────────────────
1     AA:AA:AA:AA:AA:AA
2     BB:BB:BB:BB:BB:BB  ← Learned!
3     ?
```

**Step 4: PC1 sends to PC2 again**
```
Destination MAC: BB:BB:BB:BB:BB:BB

Switch checks table:
"BB:BB:BB:BB:BB:BB is on Port 2"
Sends ONLY to Port 2 ✅
```

### Viewing Switch MAC Table (On Managed Switches):
```bash
# Cisco switch
show mac address-table

# Output:
Vlan    Mac Address       Type    Ports
----    -----------       ----    -----
1       aa:aa:aa:aa:aa:aa DYNAMIC Gi0/1
1       bb:bb:bb:bb:bb:bb DYNAMIC Gi0/2
1       cc:cc:cc:cc:cc:cc DYNAMIC Gi0/3
```

### MAC Table Aging:
```
Entries expire after ~5 minutes of inactivity
Why? To handle device moves and disconnects

Old entry removed → Next packet floods → Re-learns
```

---

## 🔍 ARP: Address Resolution Protocol

**The bridge between Layer 2 (MAC) and Layer 3 (IP)**

### The Problem:
```
You know destination IP: 192.168.1.100
But Ethernet frames need MAC addresses!
How do you find the MAC for that IP?

Answer: ARP!
```

### ARP Process:

**Step 1: ARP Request (Broadcast)**
```
Your PC: "Who has IP 192.168.1.100? Tell 192.168.1.50"

Broadcast frame:
Destination MAC: FF:FF:FF:FF:FF:FF (everyone!)
Source MAC: Your MAC
Message: "Who has 192.168.1.100?"
```

**Step 2: ARP Reply (Unicast)**
```
192.168.1.100: "I'm at MAC AA:BB:CC:DD:EE:FF"

Unicast frame:
Destination MAC: Your MAC
Source MAC: AA:BB:CC:DD:EE:FF
Message: "192.168.1.100 is at AA:BB:CC:DD:EE:FF"
```

**Step 3: Cache the Result**
```
Your PC stores in ARP cache:
192.168.1.100 → AA:BB:CC:DD:EE:FF

Next time: No ARP needed, use cached MAC!
```

### View ARP Cache:
```bash
# Linux
ip neigh show
# or
arp -a

# Output:
192.168.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
192.168.1.100 dev eth0 lladdr 11:22:33:44:55:66 STALE
192.168.1.50 dev eth0 lladdr 77:88:99:aa:bb:cc DELAY

States:
REACHABLE - Valid, recently confirmed
STALE - May be outdated
DELAY - Checking if still valid
FAILED - Unreachable
```

### ARP Commands:
```bash
# View ARP table
ip neigh show

# Delete specific entry
sudo ip neigh del 192.168.1.100 dev eth0

# Flush all ARP cache
sudo ip neigh flush all

# Add static ARP entry
sudo ip neigh add 192.168.1.100 lladdr aa:bb:cc:dd:ee:ff dev eth0
```

### ARP in Action:
```bash
# Watch ARP in real-time
sudo tcpdump -i eth0 arp

# Send ARP request manually
arping -c 1 192.168.1.100

# Output:
ARPING 192.168.1.100 from 192.168.1.50 eth0
Unicast reply from 192.168.1.100 [AA:BB:CC:DD:EE:FF]  1.234ms
```

---

## 🔄 Complete Communication Flow (IP to MAC)

### Sending packet to 192.168.1.100:

```
┌─────────────────────────────────────────┐
│ Application Layer                       │
│ "Send HTTP request to 192.168.1.100"    │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Network Layer (IP)                      │
│ Destination IP: 192.168.1.100           │
│ Source IP: 192.168.1.50                 │
└─────────────────┬───────────────────────┘
                  ↓
      "What's the MAC for 192.168.1.100?"
                  ↓
┌─────────────────────────────────────────┐
│ ARP Check Cache                         │
│ - Found? Use it!                        │
│ - Not found? Send ARP request           │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Data Link Layer (Ethernet)              │
│ Destination MAC: AA:BB:CC:DD:EE:FF      │
│ Source MAC: Your MAC                    │
│ Contains: IP packet                     │
└─────────────────┬───────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│ Physical Layer                          │
│ Electrical signals on wire              │
└─────────────────────────────────────────┘
```

---

## 🏢 VLANs: Virtual LANs

**One physical switch acting as multiple virtual switches.**

### Without VLANs:
```
        [Switch]
       /   |    \
      /    |     \
   Sales  HR   Engineering

Problem: Everyone on same broadcast domain!
```

### With VLANs:
```
        [Switch]
       /    |     \
      /     |      \
  VLAN10  VLAN20  VLAN30
  Sales    HR    Engineering

Each VLAN = Separate network
Can't communicate without router
```

### VLAN Benefits:
```
✅ Security (isolate departments)
✅ Reduce broadcast traffic
✅ Flexibility (reorganize without rewiring)
✅ Better performance
```

### VLAN Configuration Example:
```bash
# Create VLAN interface on Linux
sudo ip link add link eth0 name eth0.10 type vlan id 10

# Assign IP to VLAN interface
sudo ip addr add 192.168.10.1/24 dev eth0.10

# Bring it up
sudo ip link set eth0.10 up

# View VLAN interfaces
ip -d link show eth0.10
```

---

## 🔧 Switching Types

### 1. Store-and-Forward
```
Process:
1. Receive entire frame
2. Check for errors (FCS)
3. Forward if valid

Pros: Error-free
Cons: Higher latency
```

### 2. Cut-Through
```
Process:
1. Read destination MAC (first 6 bytes)
2. Start forwarding immediately

Pros: Lower latency
Cons: May forward bad frames
```

### 3. Fragment-Free
```
Hybrid approach:
1. Read first 64 bytes
2. Check for collisions
3. Forward

Balance of speed and reliability
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Troubleshooting Connectivity
```bash
# Can't reach server at 192.168.1.100

# Step 1: Check ARP
ip neigh show | grep 192.168.1.100

# No entry? Try to reach it
ping 192.168.1.100

# Check again
ip neigh show | grep 192.168.1.100

# Still nothing? Layer 2 problem!
```

### Scenario 2: Docker Networking
```bash
# Docker creates a virtual switch (bridge)
ip link show docker0

# Output:
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    link/ether 02:42:9e:7f:3a:8d

# Containers connect to this bridge
# Switch learns container MAC addresses
```

### Scenario 3: MAC Address Conflicts
```bash
# Two devices with same MAC (rare but happens)

# Symptoms:
# - Intermittent connectivity
# - Traffic going to wrong device

# Find duplicate MACs
arp-scan --interface=eth0 --localnet | sort -k2

# Look for duplicate MACs in second column
```

### Scenario 4: Network Segmentation
```bash
# Using VLANs to isolate environments

# Production: VLAN 10 (10.0.10.0/24)
# Staging: VLAN 20 (10.0.20.0/24)
# Development: VLAN 30 (10.0.30.0/24)

# Each VLAN isolated at Layer 2
# Requires router to communicate between VLANs
```

---

## 🔍 Practical Switching Commands

### Linux Bridge (Virtual Switch):
```bash
# Create bridge
sudo ip link add br0 type bridge

# Add interfaces to bridge
sudo ip link set eth0 master br0
sudo ip link set eth1 master br0

# Bring bridge up
sudo ip link set br0 up

# View bridge details
ip link show master br0

# Show bridge MAC table
bridge fdb show br br0
```

### Analyze Network Traffic:
```bash
# Capture Layer 2 traffic
sudo tcpdump -i eth0 -e

# Output shows MAC addresses:
00:0c:29:3f:4a:5b > aa:bb:cc:dd:ee:ff, ethertype IPv4...

# See ARP traffic
sudo tcpdump -i eth0 arp -n

# Monitor specific MAC
sudo tcpdump -i eth0 ether host aa:bb:cc:dd:ee:ff
```

---

## 📊 Switch vs Router Comparison

```
Feature           Switch (Layer 2)       Router (Layer 3)
─────────────────────────────────────────────────────────
Uses              MAC addresses          IP addresses
Forwards          Frames                 Packets
Connects          Devices in same LAN    Different networks
Broadcast Domain  Single                 Separates domains
Learning          MAC address table      Routing table
Speed             Very fast              Slower (more processing)
Intelligence      MAC-based              IP-based, routing
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is a MAC address?**
   <details>
   <summary>Answer</summary>
   A unique hardware address (48-bit) assigned to network interface cards. Format: XX:XX:XX:XX:XX:XX
   </details>

2. **How does a switch learn MAC addresses?**
   <details>
   <summary>Answer</summary>
   By reading the source MAC address of incoming frames and associating it with the port it came from.
   </details>

3. **What is ARP used for?**
   <details>
   <summary>Answer</summary>
   To resolve IP addresses to MAC addresses. Asks "Who has this IP?" and gets back "I do, my MAC is..."
   </details>

4. **What's the broadcast MAC address?**
   <details>
   <summary>Answer</summary>
   FF:FF:FF:FF:FF:FF - sends to all devices on the network
   </details>

5. **What's the difference between a hub and a switch?**
   <details>
   <summary>Answer</summary>
   Hub broadcasts to all ports (dumb). Switch forwards only to destination port (smart).
   </details>

---

## 📝 Key Takeaways

✅ MAC addresses are hardware addresses (48-bit, Layer 2)  
✅ Switches use MAC addresses to forward frames intelligently  
✅ Switches learn by reading source MAC addresses  
✅ ARP resolves IP addresses to MAC addresses  
✅ ARP cache stores IP-to-MAC mappings temporarily  
✅ VLANs create virtual networks on one physical switch  
✅ `ip neigh` shows ARP table  
✅ Switches are Layer 2, Routers are Layer 3  

---

## 🚀 Next Steps

You understand Layer 2 switching! Next, let's learn about NAT (Network Address Translation) - how private IPs access the internet.

**Next lesson:** `11-nat.md`

---

## 💡 Pro Tip

**Quick ARP troubleshooting:**
```bash
# 1. Check ARP cache
ip neigh show

# 2. Clear stale entries
sudo ip neigh flush all

# 3. Test connectivity
ping <IP>

# 4. Verify new ARP entry
ip neigh show | grep <IP>
```

ARP issues cause 50% of "I can't connect" problems! 🎯
