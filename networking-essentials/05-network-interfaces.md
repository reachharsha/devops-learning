---
render_with_liquid: false
---
# 05 - Network Interfaces

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What network interfaces are
- Linux interface naming conventions
- Loopback interface
- How to view and manage interfaces
- Virtual interfaces

---

## 🤔 What is a Network Interface?

**A network interface is the connection point between your computer and a network.**

**Think of it as a door:**
- Your computer is the house
- The network is the street
- The interface is the door connecting them

```
┌─────────────────────────┐
│   Your Computer         │
│                         │
│  [Network Interface] ◄──┼──► Network
│       (eth0)            │
└─────────────────────────┘
```

---

## 📛 Interface Naming: Old vs New

Linux has two naming schemes for network interfaces.

### Old Naming (Predictable but Simple)
```
eth0  - First Ethernet interface
eth1  - Second Ethernet interface
wlan0 - First wireless interface
wlan1 - Second wireless interface
lo    - Loopback (always)
```

**Problem:** Names could change on reboot! 😱

### New Naming (Predictable and Consistent)
```
eno1      - Onboard Ethernet port 1
ens33     - Ethernet PCI slot 33
enp3s0    - Ethernet PCI bus 3, slot 0
wlp2s0    - Wireless PCI bus 2, slot 0
lo        - Loopback (always)
```

**Benefit:** Names stay the same after reboot! 🎉

### Decoding New Names:
```
Format: <type><bus><slot>

en  = Ethernet
wl  = Wireless
o   = Onboard
s   = Slot
p   = PCI bus

Examples:
eno1   = Ethernet, Onboard, port 1
ens33  = Ethernet, Slot 33
enp3s0 = Ethernet, PCI bus 3, Slot 0
wlp2s0 = Wireless, PCI bus 2, Slot 0
```

---

## 🔄 The Loopback Interface (lo)

**The most important interface you'll never see!**

```
┌─────────────────────────┐
│   Your Computer         │
│                         │
│  Application 1          │
│       ↓                 │
│   [lo: 127.0.0.1] ←───┐ │  ← Traffic never leaves!
│       ↑               │ │
│  Application 2 ───────┘ │
│                         │
└─────────────────────────┘
```

### Loopback Details:
- **IP Address:** 127.0.0.1 (IPv4) or ::1 (IPv6)
- **Hostname:** localhost
- **Purpose:** Internal communication within the same computer
- **Speed:** Fastest possible (no physical hardware!)

### Why It Matters:
```bash
# Testing if network stack is working
ping 127.0.0.1

# Connecting to database on same server
mysql -h 127.0.0.1

# Docker containers accessing host
curl http://localhost:8080
```

**Real-World Analogy:** 
Sending a letter to yourself - it never leaves your house!

---

## 👀 Viewing Network Interfaces

### Method 1: `ip link show` (Modern)
```bash
ip link show

# Output:
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP
    link/ether 00:0c:29:3f:4a:5b brd ff:ff:ff:ff:ff:ff
```

**Breaking it down:**
```
1: lo:                     ← Interface name
<LOOPBACK,UP,LOWER_UP>     ← Status flags
mtu 65536                  ← Maximum Transmission Unit
state UNKNOWN              ← Interface state
link/loopback              ← Type
```

### Method 2: `ip addr show` (With IP addresses)
```bash
ip addr show

# Output:
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo     ← IPv4 address
    inet6 ::1/128 scope host           ← IPv6 address

2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 192.168.1.100/24 brd 192.168.1.255 scope global ens33
    inet6 fe80::20c:29ff:fe3f:4a5b/64 scope link
```

### Method 3: `ifconfig` (Old but still useful)
```bash
ifconfig

# Output:
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.100  netmask 255.255.255.0  broadcast 192.168.1.255
        ether 00:0c:29:3f:4a:5b  txqueuelen 1000  (Ethernet)
        RX packets 12345  bytes 8901234 (8.9 MB)
        TX packets 6789   bytes 4567890 (4.5 MB)
```

---

## 🏷️ Interface States

### UP vs DOWN
```
UP    ✅ - Interface is active and ready
DOWN  ❌ - Interface is disabled
```

### LOWER_UP
```
LOWER_UP ✅ - Physical link detected (cable plugged in)
NO-CARRIER ❌ - No cable connected
```

### Example Outputs:
```bash
# Interface up with cable connected
ens33: <BROADCAST,MULTICAST,UP,LOWER_UP>
       ✅ Interface enabled
       ✅ Cable connected

# Interface up but cable disconnected
ens33: <NO-CARRIER,BROADCAST,MULTICAST,UP>
       ✅ Interface enabled
       ❌ Cable NOT connected

# Interface disabled
ens33: <BROADCAST,MULTICAST>
       ❌ Interface disabled
```

---

## ⚙️ Managing Interfaces

### Bring Interface Up/Down:
```bash
# Bring interface up
sudo ip link set ens33 up

# Bring interface down
sudo ip link set ens33 down

# Old method (still works)
sudo ifconfig ens33 up
sudo ifconfig ens33 down
```

### Assign IP Address:
```bash
# Add IP address
sudo ip addr add 192.168.1.100/24 dev ens33

# Remove IP address
sudo ip addr del 192.168.1.100/24 dev ens33

# Old method
sudo ifconfig ens33 192.168.1.100 netmask 255.255.255.0
```

### Change MTU:
```bash
# Set MTU to 9000 (jumbo frames)
sudo ip link set ens33 mtu 9000
```

---

## 🎭 Types of Interfaces

### 1. Physical Interfaces
```
eth0, ens33, wlan0
↓
Connected to real hardware (NIC)
```

### 2. Virtual Interfaces

#### a) **Loopback (lo)**
```
For local communication
Always 127.0.0.1
```

#### b) **VLAN Interfaces**
```
ens33.10  - VLAN 10 on ens33
ens33.20  - VLAN 20 on ens33

One physical interface, multiple logical networks
```

#### c) **Bridge Interfaces**
```
br0  - Bridge connecting multiple interfaces
Used by VMs and containers
```

#### d) **Tunnel Interfaces**
```
tun0  - VPN tunnel
gre0  - GRE tunnel
```

#### e) **Docker Interfaces**
```
docker0       - Default Docker bridge
veth12345     - Virtual Ethernet pairs (container links)
```

---

## 🐳 Docker Network Interfaces

When you install Docker, new interfaces appear:

```bash
ip link show

# New interfaces:
docker0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 172.17.0.1/16
    
veth7a8b9c@if4: <BROADCAST,MULTICAST,UP,LOWER_UP>
    ← Virtual Ethernet pair connecting container
```

### Docker Network Flow:
```
Container
    ↓
veth (Virtual Ethernet in container)
    ↓
veth (Virtual Ethernet on host)
    ↓
docker0 (Bridge)
    ↓
ens33 (Physical interface)
    ↓
Internet
```

---

## 🔍 Detailed Interface Information

### View Specific Interface:
```bash
# Just one interface
ip addr show ens33

# Or
ifconfig ens33
```

### Get Statistics:
```bash
# Packets sent/received
ip -s link show ens33

# Output:
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    RX: bytes  packets  errors  dropped
        1048576  8192    0       0
    TX: bytes  packets  errors  dropped
        524288   4096    0       0
```

### Check Link Speed:
```bash
ethtool ens33 | grep Speed

# Output:
Speed: 1000Mb/s  ← Gigabit connection
```

---

## 📊 Interface Configuration Files

### RHEL/CentOS/Fedora:
```bash
/etc/sysconfig/network-scripts/ifcfg-ens33

# Example content:
TYPE=Ethernet
BOOTPROTO=static
NAME=ens33
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
```

### Ubuntu/Debian (Netplan):
```bash
/etc/netplan/01-netcfg.yaml

# Example content:
network:
  version: 2
  ethernets:
    ens33:
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

### Ubuntu/Debian (Old style):
```bash
/etc/network/interfaces

# Example:
auto ens33
iface ens33 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Server Not Reachable
```bash
# Check if interface is up
ip link show ens33

# Is it UP and LOWER_UP?
# If not:
sudo ip link set ens33 up
```

### Scenario 2: Container Networking Issues
```bash
# Check Docker bridge
ip addr show docker0

# Is it 172.17.0.1/16?
# Are veth pairs present?
```

### Scenario 3: Performance Troubleshooting
```bash
# Check for errors
ip -s link show ens33

# Look for:
# - errors: packets with errors
# - dropped: packets dropped
# - collisions: network collisions
```

### Scenario 4: Setting Up New Server
```bash
# 1. Identify interface
ip link show

# 2. Assign IP
sudo ip addr add 10.0.1.100/24 dev ens33

# 3. Bring it up
sudo ip link set ens33 up

# 4. Add default route
sudo ip route add default via 10.0.1.1
```

---

## 🛠️ Common Tasks

### Task 1: Find Your Interface Name
```bash
# List all interfaces
ip link show | grep -E '^[0-9]'

# Just Ethernet interfaces
ip link show | grep -E 'en[ops]'
```

### Task 2: Check IP Address
```bash
# All interfaces
ip addr show

# Specific interface
ip addr show ens33

# Just the IP
ip addr show ens33 | grep 'inet ' | awk '{print $2}'
```

### Task 3: Check if Cable is Connected
```bash
# Look for LOWER_UP
ip link show ens33 | grep LOWER_UP

# Or check carrier
cat /sys/class/net/ens33/carrier
# 1 = connected, 0 = not connected
```

### Task 4: Monitor Traffic in Real-Time
```bash
# Watch interface statistics
watch -n 1 'ip -s link show ens33'

# Or use iftop
sudo iftop -i ens33
```

---

## 🎯 Quick Check: Do You Understand?

1. **What is the loopback interface?**
   <details>
   <summary>Answer</summary>
   An internal interface (127.0.0.1) used for communication within the same computer. Traffic never leaves the system.
   </details>

2. **What does "LOWER_UP" mean?**
   <details>
   <summary>Answer</summary>
   The physical link is detected - a cable is connected to the interface.
   </details>

3. **What's the difference between eth0 and ens33?**
   <details>
   <summary>Answer</summary>
   eth0 is old naming (simple, sequential). ens33 is new predictable naming (based on hardware location).
   </details>

4. **How do you check if an interface has an IP address?**
   <details>
   <summary>Answer</summary>
   `ip addr show <interface>` or `ifconfig <interface>`
   </details>

---

## 📝 Key Takeaways

✅ Network interface = connection point to network  
✅ **lo** (127.0.0.1) = loopback for local communication  
✅ Old names: eth0, wlan0  
✅ New names: ens33, enp3s0, wlp2s0  
✅ **UP** = interface enabled, **LOWER_UP** = cable connected  
✅ `ip` command is modern replacement for `ifconfig`  
✅ Docker creates virtual interfaces (docker0, veth pairs)  
✅ Interface config files vary by Linux distribution  

---

## 🚀 Next Steps

You now understand network interfaces! Next, we'll dive into the OSI model - the framework that explains how all networking works.

**Next lesson:** `06-osi-model.md`

---

## 💡 Pro Tip

**Quick interface health check:**
```bash
# One command to see everything important
ip -br addr show

# Output:
lo       UNKNOWN  127.0.0.1/8 ::1/128
ens33    UP       192.168.1.100/24 fe80::20c:29ff:fe3f:4a5b/64

# -br = brief format
# Shows: name, state, IP addresses
```

Bookmark this command! 🔖
