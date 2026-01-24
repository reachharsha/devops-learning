---
render_with_liquid: false
---
# 04 - Network Hardware

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- Physical components of networks
- What cables, switches, and routers do
- Difference between hubs, switches, and routers
- When to use each device

---

## 🔌 Network Cables: The Physical Connection

### 1. Ethernet Cable (Most Common)

```
┌─────────────────────────────────┐
│  [RJ-45 Connector]~~~~~~~~~~~~~~│  ← Twisted pair cable
└─────────────────────────────────┘
```

**Types:**
- **Cat5e** - 1 Gbps, up to 100 meters
- **Cat6** - 10 Gbps, up to 55 meters  
- **Cat6a** - 10 Gbps, up to 100 meters
- **Cat7/8** - Higher speeds, specialized use

**Real-World Analogy:** Like a phone cable but thicker, connects computers to network devices.

### 2. Fiber Optic Cable

```
┌─────────────────────────────────┐
│  [Fiber Connector]═══════════   │  ← Light signals
└─────────────────────────────────┘
```

**Characteristics:**
- Uses light instead of electricity
- Much faster (100 Gbps+)
- Longer distances (kilometers)
- More expensive
- Used in data centers and ISP backbones

### 3. Wireless (WiFi)

```
    📡 Router
   /  |  \
  /   |   \
 💻  📱  🖥️
```

**No physical cable!** Uses radio waves.

---

## 🔀 Network Devices: The Traffic Managers

### 1. Network Interface Card (NIC)

```
Computer Motherboard
┌──────────────────┐
│                  │
│  [NIC Card]      │ ← Has a unique MAC address
│      └─[Port]    │ ← Where cable plugs in
└──────────────────┘
```

**What it does:**
- Connects your computer to the network
- Has a unique hardware address (MAC address)
- Converts data to electrical signals

**Example MAC address:** `00:1A:2B:3C:4D:5E`

**DevOps Note:** 
```bash
# View your NIC on Linux
ip link show
# or
ifconfig
```

---

## 🔁 Hub (Obsolete - But Good to Understand)

```
        [HUB]
       /  |  \
      /   |   \
    PC1  PC2  PC3
```

**How it works:**
1. PC1 sends data to the hub
2. Hub broadcasts to ALL ports
3. PC2 and PC3 receive it (even if not intended for them)

**Problems:**
❌ No intelligence - broadcasts everything  
❌ Wastes bandwidth  
❌ Security risk (everyone sees all traffic)  
❌ Collisions happen often  

**Status:** Replaced by switches. You won't see these in modern networks.

---

## 🔀 Switch (Layer 2 Device)

```
        [SWITCH]
       /  |  \
      /   |   \
    PC1  PC2  PC3
    
MAC Table:
Port 1: MAC-A (PC1)
Port 2: MAC-B (PC2)
Port 3: MAC-C (PC3)
```

**How it works:**
1. PC1 sends data for PC2
2. Switch checks its MAC address table
3. Sends data ONLY to port 2 (PC2)
4. PC3 doesn't receive it

**Advantages:**
✅ Intelligent - learns which MAC is on which port  
✅ Efficient - sends data only where needed  
✅ Faster - no collisions  
✅ More secure - devices only see their own traffic  

**Real-World Analogy:** 
A smart mail sorter that knows which apartment each person lives in and delivers mail directly.

### Switch Types:

**Unmanaged Switch:**
```
- Plug and play
- No configuration needed
- Good for home/small office
- Example: 8-port Netgear switch
```

**Managed Switch:**
```
- Configurable (VLANs, QoS, monitoring)
- Can prioritize traffic
- Better security
- Used in enterprise/data centers
```

---

## 🌐 Router (Layer 3 Device)

```
     Home Network              Internet            Google
    ┌──────────────┐          ┌──────┐          ┌──────┐
    │ 💻 📱 🖥️    │          │      │          │      │
    │              │          │      │          │      │
    │  [Router]    │◄────────►│ ISP  │◄────────►│Server│
    │ 192.168.1.1  │          │      │          │      │
    └──────────────┘          └──────┘          └──────┘
    Private IPs              Public Internet
```

**What it does:**
- **Connects different networks** (your home to the internet)
- **Routes packets** to correct destinations
- **Provides NAT** (Network Address Translation)
- **Has firewall** capabilities
- **Assigns IP addresses** (DHCP)

**Router vs Switch:**
```
Switch: Connects devices WITHIN a network (uses MAC addresses)
Router: Connects BETWEEN networks (uses IP addresses)
```

### Router Functions:

**1. Routing Table:**
```
Destination      Gateway        Interface
0.0.0.0/0        ISP Router     eth0        (Default route - internet)
192.168.1.0/24   0.0.0.0        eth1        (Local network)
10.0.0.0/8       10.1.1.1       eth2        (Another network)
```

**2. NAT (Network Address Translation):**
```
Inside:  192.168.1.50:54321  ──┐
Inside:  192.168.1.51:54322  ──┤
                                ├──► Router ──► 203.0.113.5:8080 (Public IP)
Inside:  192.168.1.52:54323  ──┤
Inside:  192.168.1.53:54324  ──┘

Multiple private IPs → One public IP
```

**3. DHCP Server:**
```
New device connects:
"I need an IP address!"
        ↓
Router responds:
"Use 192.168.1.100, I'm your gateway at 192.168.1.1"
```

**Real-World Analogy:**
A router is like a border checkpoint between cities (networks), deciding which traffic can pass and how.

---

## 🔥 Firewall (Security Device)

```
    Internet                    Your Network
       │                             │
   [Firewall]                        │
    Block ↓   Allow →                │
       ❌────────✅─────────► [Web Server]
```

**What it does:**
- Filters traffic based on rules
- Blocks malicious connections
- Allows legitimate traffic
- Can be hardware or software

**Types:**

**1. Hardware Firewall:**
```
Internet → [Firewall Appliance] → Network
Examples: Cisco ASA, Palo Alto, pfSense
```

**2. Software Firewall:**
```
On each computer
Examples: iptables, ufw, Windows Firewall
```

**DevOps Example:**
```bash
# Block incoming SSH except from specific IP
sudo ufw allow from 203.0.113.5 to any port 22
sudo ufw deny 22
```

---

## 🔌 Modem (Modulator-Demodulator)

```
[Phone/Cable Line] ──► [Modem] ──► [Router] ──► [Your Devices]
    ISP Signal         Converts      Routes       Use Internet
                      to Ethernet
```

**What it does:**
- Converts ISP signal (cable/DSL/fiber) to Ethernet
- Provides internet connection
- Usually combined with router nowadays

**Modem vs Router:**
```
Modem:  Brings internet INTO your home
Router: Distributes internet to your devices
```

---

## 📡 Access Point (AP)

```
        [Wired Network]
              │
         [Switch]
              │
        [Access Point]  ← Converts wired to wireless
           📡
         /  |  \
        💻 📱 🖥️
```

**What it does:**
- Creates WiFi from wired connection
- Extends wireless coverage
- Multiple devices can connect

**Home Router vs Access Point:**
```
Home Router = Router + Switch + Access Point (all-in-one)
Access Point = ONLY provides WiFi (needs separate router)
```

---

## 🏢 Putting It All Together: Network Topology

### Small Office Network:
```
              Internet
                 │
            [Modem/Router]
            (NAT, DHCP, Firewall)
                 │
            [Switch] ────── [Access Point]
            /  |  \              📡
           /   |   \           /  |  \
       PC1   PC2  Server    💻  📱  💻
```

### Data Center Network:
```
                  Internet
                     │
              [Edge Router]
              (Firewall, NAT)
                     │
           [Core Switch Layer]
              /           \
    [Distribution          [Distribution
     Switch 1]              Switch 2]
        │                      │
    [Access               [Access
     Switches]             Switches]
    /  |  \               /  |  \
  Srv1 Srv2 Srv3       Srv4 Srv5 Srv6
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Deploying Applications
```
You're deploying a web app:
- Server connected to switch (Layer 2)
- Switch connected to router (Layer 3)
- Router has firewall rules
- Load balancer in front

Need to understand: How packets flow through each device
```

### Scenario 2: Troubleshooting Connectivity
```
Problem: "Can't reach database server"

Check each layer:
1. Cable plugged in? (Physical)
2. Switch port active? (Data Link)
3. Router routing correctly? (Network)
4. Firewall blocking? (Security)
```

### Scenario 3: Container Networking
```
Docker creates virtual network devices:
- Virtual switches (docker0 bridge)
- Virtual NICs (veth pairs)
- Virtual routers (for overlay networks)

Same concepts, virtualized!
```

---

## 🔍 Viewing Network Hardware on Linux

### See Network Interfaces:
```bash
# Modern command
ip link show

# Output:
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
3: wlan0: <BROADCAST,MULTICAST> mtu 1500
```

### Check Cable Connection:
```bash
# Is cable plugged in?
ethtool eth0 | grep "Link detected"

# Output:
Link detected: yes  ✅
# or
Link detected: no   ❌
```

### View Hardware Info:
```bash
# Detailed NIC information
lshw -class network

# MAC address
ip link show eth0 | grep link/ether
```

---

## 🎯 Quick Comparison Table

```
Device      Layer  Function                Uses
────────────────────────────────────────────────────
NIC         L1/L2  Connects PC to network  MAC address
Hub         L1     Broadcasts to all       None (obsolete)
Switch      L2     Smart forwarding        MAC addresses
Router      L3     Connects networks       IP addresses
Firewall    L3-L7  Filters traffic         Rules/policies
Access Point L2    Wireless access         MAC addresses
Modem       L1/L2  Internet connection     ISP signals
```

---

## 📝 Practical Examples

### Example 1: Home Network
```
Internet
   │
[Modem/Router Combo]  ← One device doing multiple jobs
   │
   ├─ Wired: Desktop PC
   ├─ Wired: Smart TV
   └─ WiFi: Phones, Laptops
```

### Example 2: Small Business
```
Internet
   │
[Router/Firewall]
   │
[Managed Switch]
   ├─ Server 1
   ├─ Server 2
   ├─ [Access Point] → WiFi for employees
   └─ [Workstations]
```

### Example 3: Data Center
```
Internet
   │
[Edge Routers] (Redundant)
   │
[Core Switches] (High-speed)
   │
[Top-of-Rack Switches]
   │
[Servers/VMs/Containers]
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between a switch and a router?**
   <details>
   <summary>Answer</summary>
   Switch connects devices within a network using MAC addresses (Layer 2). Router connects different networks using IP addresses (Layer 3).
   </details>

2. **Why are hubs obsolete?**
   <details>
   <summary>Answer</summary>
   They broadcast all traffic to all ports, wasting bandwidth and creating security issues. Switches are smarter and more efficient.
   </details>

3. **What does a modem do?**
   <details>
   <summary>Answer</summary>
   Converts your ISP's signal (cable/DSL/fiber) into Ethernet that your router can use.
   </details>

4. **What's a MAC address?**
   <details>
   <summary>Answer</summary>
   A unique hardware address assigned to every network interface card (NIC). Example: 00:1A:2B:3C:4D:5E
   </details>

---

## 📝 Key Takeaways

✅ **NICs** connect devices to networks (have MAC addresses)  
✅ **Switches** forward traffic intelligently within a network (Layer 2)  
✅ **Routers** connect different networks and route packets (Layer 3)  
✅ **Firewalls** filter traffic for security  
✅ **Modems** bring internet into your location  
✅ **Access Points** provide WiFi connectivity  
✅ Each device has a specific role in the network  
✅ Modern devices often combine multiple functions  

---

## 🚀 Next Steps

Now you understand the physical components! Next, let's learn about network interfaces and how Linux names them.

**Next lesson:** `05-network-interfaces.md`

---

## 💡 Pro Tip

When troubleshooting networks:
1. Start at the physical layer (cables, lights)
2. Move up to switches and routers
3. Check routing and firewalls last

**"Always check if it's plugged in first!"** 🔌😄
