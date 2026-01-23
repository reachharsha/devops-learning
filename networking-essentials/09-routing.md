# 09 - Routing: How Packets Find Their Way

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What routing is and why it's needed
- How routing tables work
- Default gateway concept
- Static vs dynamic routing
- How to view and manipulate routes

---

## 🤔 What is Routing?

**Routing is the process of forwarding packets from source to destination across different networks.**

**Real-World Analogy:**
```
Sending a package across the country:

Your House → Local Post Office → Regional Hub → 
→ Destination Regional Hub → Destination Post Office → Friend's House

Each stop decides: "Where should this go next?"
That's routing!
```

### Without Routing:
```
❌ Can only talk to devices on same network
❌ No internet access
❌ Isolated networks
```

### With Routing:
```
✅ Connect to any network
✅ Access the internet
✅ Global communication
```

---

## 🗺️ The Routing Table

**Every device has a routing table that says: "For destination X, send packets to Y"**

### View Your Routing Table:
```bash
# Modern Linux
ip route show

# Output:
default via 192.168.1.1 dev eth0 proto dhcp metric 100
192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
```

**Breaking it down:**
```
default via 192.168.1.1 dev eth0
  │      │       │         │
  │      │       │         └── Use interface eth0
  │      │       └──────────── Send to this IP (gateway)
  │      └──────────────────── "via" means through this router
  └─────────────────────────── For all unknown destinations

192.168.1.0/24 dev eth0 proto kernel scope link
  │                │
  │                └──────────── Directly connected (same network)
  └───────────────────────────── Destination network
```

---

## 🚪 Default Gateway

**The router that handles traffic to destinations outside your local network.**

```
Your Computer (192.168.1.100)
      ↓
"Where is 8.8.8.8?"
      ↓
Routing Table: "Not on my network, send to default gateway"
      ↓
Default Gateway (192.168.1.1)
      ↓
Router forwards to Internet
```

### Configuring Default Gateway:
```bash
# Add default route
sudo ip route add default via 192.168.1.1 dev eth0

# Delete default route
sudo ip route del default

# Replace existing default
sudo ip route replace default via 192.168.1.1 dev eth0
```

---

## 📋 Routing Table Entries Explained

### Entry Format:
```
<destination> via <gateway> dev <interface> [options]
```

### Example Routing Table:
```bash
ip route show

# Output explained:
default via 192.168.1.1 dev eth0 
  → "For everything else, use router at 192.168.1.1"

192.168.1.0/24 dev eth0 proto kernel scope link src 192.168.1.100
  → "192.168.1.x is directly connected via eth0"

10.0.0.0/8 via 192.168.1.254 dev eth0
  → "For 10.x.x.x, send to router 192.168.1.254"
```

### Routing Decision Process:
```
Packet destination: 10.0.1.50

Router checks table (most specific first):
1. 10.0.1.50/32?        NO (no exact match)
2. 10.0.1.0/24?         NO
3. 10.0.0.0/16?         NO
4. 10.0.0.0/8?          YES! → Use this route
5. default?             (not reached)
```

---

## 🔄 How Routing Works: Step-by-Step

### Scenario: Accessing google.com (8.8.8.8)

```
Step 1: Application Layer
Browser: "Get me google.com"
DNS: "That's 8.8.8.8"

Step 2: Check Local Network
Computer checks: "Is 8.8.8.8 on my network (192.168.1.0/24)?"
Answer: NO

Step 3: Consult Routing Table
Computer checks table:
  - 192.168.1.0/24? NO
  - default? YES → Send to 192.168.1.1

Step 4: Send to Gateway
Packet sent to router at 192.168.1.1

Step 5: Router Routing
Router checks ITS routing table:
  - Local networks? NO
  - default? YES → Send to ISP router

Step 6: ISP Routing
ISP router has route to 8.8.8.0/24
Forwards packet closer to destination

Step 7: More Hops
Packet passes through multiple routers
Each router makes forwarding decision

Step 8: Arrival
Packet reaches 8.8.8.8 (Google DNS)
```

### Visual Representation:
```
You                Router 1           Router 2           Router 3           Google
(192.168.1.100) → (192.168.1.1)   → (ISP Router)     → (Backbone)       → (8.8.8.8)
                   Route via ISP      Route via next     Route to final      Destination!
                                      backbone hop       destination
```

---

## 🧭 Tracing the Route

### Using traceroute:
```bash
# Trace path to Google
traceroute 8.8.8.8

# Output:
 1  192.168.1.1 (192.168.1.1)        1.234 ms   ← Your router
 2  10.0.0.1 (10.0.0.1)             5.678 ms   ← ISP router
 3  72.14.204.1 (72.14.204.1)      12.345 ms   ← ISP backbone
 4  * * *                                       ← Router not responding
 5  108.170.250.1 (108.170.250.1)  15.678 ms
 6  8.8.8.8 (8.8.8.8)               18.901 ms   ← Google DNS!
```

**Each line = one "hop" (router)**

### Using mtr (better than traceroute):
```bash
# Install: sudo apt install mtr
mtr 8.8.8.8

# Shows real-time stats:
Host                Loss%  Snt  Last  Avg   Best  Wrst
1. 192.168.1.1        0.0%   10  1.2   1.3   1.1   1.5
2. 10.0.0.1           0.0%   10  5.6   5.8   5.5   6.2
3. 8.8.8.8            0.0%   10  18.9  19.1  18.5  20.3
```

---

## 📍 Static vs Dynamic Routing

### Static Routing

**Manually configured routes that don't change.**

```bash
# Add static route
sudo ip route add 10.0.0.0/8 via 192.168.1.254 dev eth0

# This route stays until you delete it or reboot
```

**Pros:**
- Simple
- Predictable
- No routing protocol overhead
- Secure (no routing updates)

**Cons:**
- Manual configuration
- Doesn't adapt to network changes
- Not scalable for large networks

**Use Cases:**
- Small networks
- Servers with fixed routes
- Point-to-point links

### Dynamic Routing

**Routes learned automatically via routing protocols.**

**Protocols:**
```
RIP (Routing Information Protocol)
- Simple, older
- Uses hop count

OSPF (Open Shortest Path First)
- Enterprise standard
- Faster convergence
- Link-state protocol

BGP (Border Gateway Protocol)
- Internet backbone
- Connects different autonomous systems
```

**Pros:**
- Automatic adaptation
- Scales to large networks
- Handles failures automatically

**Cons:**
- Complex configuration
- Requires routing protocol
- More overhead

**Use Cases:**
- Large enterprise networks
- ISP networks
- Data centers

---

## 🛠️ Managing Routes in Linux

### View Routes:
```bash
# All routes
ip route show

# IPv6 routes
ip -6 route show

# Specific destination
ip route show to 8.8.8.8

# Routing cache (older kernels)
route -n
```

### Add Static Route:
```bash
# Add route to specific network
sudo ip route add 10.0.0.0/8 via 192.168.1.254 dev eth0

# Add default route
sudo ip route add default via 192.168.1.1 dev eth0

# Add route with metric (priority)
sudo ip route add 10.0.0.0/8 via 192.168.1.254 metric 100
```

### Delete Route:
```bash
# Delete specific route
sudo ip route del 10.0.0.0/8

# Delete default route
sudo ip route del default

# Delete route via specific gateway
sudo ip route del default via 192.168.1.1
```

### Make Routes Persistent:

**Ubuntu (Netplan):**
```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: 0.0.0.0/0
          via: 192.168.1.1
        - to: 10.0.0.0/8
          via: 192.168.1.254
```

**RHEL/CentOS:**
```bash
# /etc/sysconfig/network-scripts/route-eth0
10.0.0.0/8 via 192.168.1.254 dev eth0
```

---

## 🔍 Routing Metrics

**When multiple routes exist to same destination, metric determines priority.**

```bash
ip route show

# Output:
default via 192.168.1.1 dev eth0 metric 100
default via 192.168.1.2 dev eth1 metric 200

Lower metric = higher priority
Router uses 192.168.1.1 first
```

### Route Selection Order:
```
1. Most specific route (longest prefix match)
   192.168.1.50/32 beats 192.168.1.0/24

2. Lower metric wins
   metric 100 beats metric 200

3. Administrative distance (for routing protocols)
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Troubleshooting Connectivity
```bash
# Can't reach database at 10.0.1.50

# Check routing table
ip route show

# Is there a route to 10.0.1.0/24?
ip route get 10.0.1.50

# Output:
10.0.1.50 via 192.168.1.254 dev eth0 src 192.168.1.100
# Shows path the packet will take
```

### Scenario 2: Multi-Homed Server
```bash
# Server with two network interfaces
# eth0: Internet-facing (203.0.113.5)
# eth1: Internal network (10.0.1.5)

# Route internal traffic via eth1
sudo ip route add 10.0.0.0/8 dev eth1

# Route everything else via eth0
sudo ip route add default via 203.0.113.1 dev eth0
```

### Scenario 3: Docker Network Routing
```bash
# Docker creates routes automatically
ip route show

# Output includes:
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1

# This allows host to reach containers
ping 172.17.0.2  # Reaches container
```

### Scenario 4: Kubernetes Pod Routing
```bash
# Each node needs routes to pod subnets on other nodes
# Node1: 10.244.0.0/24
# Node2: 10.244.1.0/24

# On Node1, route to Node2's pods:
ip route add 10.244.1.0/24 via <Node2-IP>
```

---

## 🧪 Testing Routing

### Check if Route Exists:
```bash
# Check route to specific destination
ip route get 8.8.8.8

# Output:
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100
    │        │               │          │
    │        │               │          └── Source IP used
    │        │               └──────────── Interface
    │        └──────────────────────────── Next hop (gateway)
    └───────────────────────────────────── Destination
```

### Test Connectivity:
```bash
# Can you reach the destination?
ping -c 4 8.8.8.8

# Trace the path
traceroute 8.8.8.8

# Test with specific interface
ping -I eth0 8.8.8.8
```

### Enable IP Forwarding (Make Linux a Router):
```bash
# Check current status
cat /proc/sys/net/ipv4/ip_forward
# 0 = disabled, 1 = enabled

# Enable temporarily
sudo sysctl -w net.ipv4.ip_forward=1

# Enable permanently
echo 'net.ipv4.ip_forward=1' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

---

## 📊 Common Routing Scenarios

### Scenario 1: Simple Home Network
```
Internet
   │
[Router: 192.168.1.1]
   │
LAN: 192.168.1.0/24
   │
Your PC: 192.168.1.100

Routing Table on PC:
default via 192.168.1.1 dev eth0
192.168.1.0/24 dev eth0 (directly connected)
```

### Scenario 2: Enterprise with Multiple Subnets
```
             [Core Router]
          /       |       \
        /         |         \
   Subnet A    Subnet B    Subnet C
  10.0.1.0/24 10.0.2.0/24 10.0.3.0/24

Routing on PC in Subnet A:
10.0.1.0/24 dev eth0 (direct)
10.0.2.0/24 via 10.0.1.1
10.0.3.0/24 via 10.0.1.1
default via 10.0.1.1
```

### Scenario 3: VPN Connection
```
Before VPN:
default via 192.168.1.1

After VPN:
10.8.0.0/24 dev tun0 (VPN subnet)
192.168.1.0/24 dev eth0 (local)
default via 10.8.0.1 dev tun0 (all traffic through VPN)
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is a default gateway?**
   <details>
   <summary>Answer</summary>
   The router that handles traffic to destinations not on your local network. The "last resort" route.
   </details>

2. **How do you view the routing table?**
   <details>
   <summary>Answer</summary>
   `ip route show` or `route -n`
   </details>

3. **What does "via" mean in a route entry?**
   <details>
   <summary>Answer</summary>
   The next-hop router (gateway) to send packets through.
   </details>

4. **What command shows the path packets take?**
   <details>
   <summary>Answer</summary>
   `traceroute <destination>` or `mtr <destination>`
   </details>

---

## 📝 Key Takeaways

✅ Routing forwards packets between networks  
✅ Every device has a routing table  
✅ Default gateway handles "unknown" destinations  
✅ Routes are checked from most specific to least  
✅ Lower metric = higher priority  
✅ `ip route show` displays routing table  
✅ `ip route get <IP>` shows path for specific destination  
✅ `traceroute` traces packet path hop-by-hop  
✅ Static routes = manual, Dynamic routes = automatic  

---

## 🚀 Next Steps

You understand routing! Next, let's learn about switching and how MAC addresses work on local networks.

**Next lesson:** `10-switching.md`

---

## 💡 Pro Tip

**Quick troubleshooting workflow:**
```bash
# 1. Can you reach the gateway?
ping 192.168.1.1

# 2. Can you reach external IP?
ping 8.8.8.8

# 3. Can you resolve DNS?
ping google.com

# 4. Check routes
ip route show

# 5. Trace the path
traceroute google.com
```

This eliminates 90% of network issues! 🎯
