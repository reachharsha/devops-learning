# 17 - Network Monitoring and Performance Testing

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- How to monitor network traffic in real-time
- Network performance testing tools
- Bandwidth monitoring and analysis
- Troubleshooting network performance issues
- DevOps monitoring best practices

---

## 📊 Why Monitor Networks?

**Real-World Analogy:**
```
Highway Traffic Monitoring:
- Speed cameras → Measure throughput
- Traffic sensors → Monitor bandwidth usage
- Accident alerts → Detect packet loss
- GPS data → Analyze latency

Network monitoring does the same for data!
```

**What we monitor:**
```
✅ Bandwidth usage (how much data)
✅ Latency (how fast)
✅ Packet loss (reliability)
✅ Active connections (who's connected)
✅ Errors and drops (problems)
```

---

## 🔍 Real-Time Traffic Monitoring

### iftop: Interface TOP

**Shows real-time bandwidth usage per connection**

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iftop

# RHEL/CentOS
sudo yum install iftop
```

**Basic usage:**
```bash
# Monitor default interface
sudo iftop

# Monitor specific interface
sudo iftop -i eth0

# Don't resolve hostnames (faster)
sudo iftop -n

# Show port numbers
sudo iftop -P
```

**iftop output explained:**
```
                    19.1Mb        38.2Mb        57.4Mb       76.5Mb
                    └──────────────┴─────────────┴────────────┘
server1.example.com    => cdn.example.com        2.41Mb  1.50Mb  1.20Mb
                       <=                         850Kb   450Kb   380Kb
│                      │  │                       │       │       │
│                      │  │                       └───────┴───────┴─ Traffic (2s, 10s, 40s avg)
│                      │  └─ Remote host
│                      └─ Direction (=> sent, <= received)
└─ Local host

Bottom section:
TX:         3.2Mb      2.1Mb      1.8Mb      ← Transmitted
RX:         1.5Mb      950Kb      820Kb      ← Received
TOTAL:      4.7Mb      3.0Mb      2.6Mb      ← Combined
```

**iftop keyboard shortcuts:**
```
p - Toggle port display
n - Toggle name resolution
t - Toggle text interface
1/2/3 - Sort by column
< - Sort by source
> - Sort by destination
q - Quit
```

**Useful flags:**
```bash
# Show port numbers, no DNS resolution
sudo iftop -nP

# Show only traffic to/from specific host
sudo iftop -f 'host 192.168.1.100'

# Show only HTTP traffic (port 80)
sudo iftop -f 'port 80'

# Monitor specific subnet
sudo iftop -F 192.168.1.0/24
```

---

### nethogs: NET HOGs

**Shows bandwidth usage per process (who's using bandwidth)**

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install nethogs

# RHEL/CentOS
sudo yum install nethogs
```

**Usage:**
```bash
# Monitor all interfaces
sudo nethogs

# Monitor specific interface
sudo nethogs eth0

# Update every 1 second (default: 1s)
sudo nethogs -d 1
```

**nethogs output:**
```
NetHogs version 0.8.6

  PID USER     PROGRAM                DEV    SENT   RECEIVED
2341  user     sshd                   eth0   0.485   1.243 KB/sec
5678  www      nginx                  eth0   5.234   2.145 KB/sec
8901  mysql    mysqld                 eth0   0.123   0.089 KB/sec
1234  user     firefox                eth0   2.456   8.901 KB/sec
  ?   root     unknown TCP                   0.050   0.020 KB/sec

  TOTAL                                      8.348  12.398 KB/sec
```

**Use cases:**
```bash
# Find which process is consuming bandwidth
sudo nethogs

# Monitor during deployment
sudo nethogs eth0

# Identify unexpected traffic
sudo nethogs | grep -v '0.000'
```

---

### iptraf-ng: IP Traffic Monitor

**Interactive, colorful network monitor**

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iptraf-ng

# RHEL/CentOS
sudo yum install iptraf-ng
```

**Usage:**
```bash
# Start interactive menu
sudo iptraf-ng

# Monitor specific interface
sudo iptraf-ng -i eth0

# General statistics
sudo iptraf-ng -g
```

**Features:**
```
- IP traffic monitor (shows connections)
- General statistics (packets/bytes)
- Detailed interface statistics
- Protocol distribution
- LAN station monitor
```

---

### nload: Network Load

**Simple bandwidth graph**

**Installation:**
```bash
sudo apt install nload  # Ubuntu/Debian
sudo yum install nload  # RHEL/CentOS
```

**Usage:**
```bash
# Monitor default interface
nload

# Monitor specific interface
nload eth0

# Monitor multiple interfaces
nload eth0 eth1

# Change refresh interval (default: 100ms)
nload -t 500
```

**nload output:**
```
Device eth0 [192.168.1.100] (1/2):
================================================================================
Incoming:
                                         Curr: 1.25 MBit/s
    ####                                 Avg:  850.32 kBit/s
    ####                                 Min:  12.45 kBit/s
    ####                                 Max:  5.67 MBit/s
    ####                                 Ttl:  125.34 GByte
Outgoing:
                                         Curr: 524.34 kBit/s
        ##                               Avg:  312.56 kBit/s
        ##                               Min:  8.23 kBit/s
        ##                               Max:  2.34 MBit/s
        ##                               Ttl:  45.67 GByte
```

---

## 📈 Bandwidth Statistics

### vnstat: Network Statistics

**Logs bandwidth usage over time (hourly/daily/monthly)**

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install vnstat

# RHEL/CentOS
sudo yum install vnstat

# Start service
sudo systemctl start vnstat
sudo systemctl enable vnstat

# Initialize database
sudo vnstat -u -i eth0
```

**Basic usage:**
```bash
# Show summary
vnstat

# Hourly stats
vnstat -h

# Daily stats
vnstat -d

# Monthly stats
vnstat -m

# Live traffic
vnstat -l

# Specific interface
vnstat -i eth0
```

**vnstat output example:**
```
                      rx      /      tx      /     total    /   estimated
 eth0:
       Dec '23      5.23 GiB  /   2.45 GiB  /    7.68 GiB  /   10.50 GiB
       Jan '24     12.34 GiB  /   5.67 GiB  /   18.01 GiB
       Feb '24      8.90 GiB  /   4.23 GiB  /   13.13 GiB
------------------------+-------------+-------------+-------------
     yesterday      450 MiB  /    234 MiB  /     684 MiB
         today      523 MiB  /    289 MiB  /     812 MiB  /    1.20 GiB
------------------------+-------------+-------------+-------------
  last 7 days      3.45 GiB  /   1.67 GiB  /    5.12 GiB
```

**Useful commands:**
```bash
# Top 10 days by traffic
vnstat -d --limit 10

# Export to JSON
vnstat --json

# Reset statistics
sudo vnstat --reset -i eth0

# Create graph (requires vnstati)
vnstati -i eth0 -h -o hourly.png
vnstati -i eth0 -d -o daily.png
```

---

## 🏎️ Performance Testing

### iperf3: Network Speed Test

**Test bandwidth between two machines**

**Installation:**
```bash
# Ubuntu/Debian
sudo apt install iperf3

# RHEL/CentOS
sudo yum install iperf3
```

**Setup (requires 2 machines):**

**Server side:**
```bash
# Start iperf3 server
iperf3 -s

# Or specify port
iperf3 -s -p 5201
```

**Client side:**
```bash
# Test to server
iperf3 -c server-ip

# Test for 30 seconds
iperf3 -c server-ip -t 30

# Reverse (server sends to client)
iperf3 -c server-ip -R

# Bidirectional
iperf3 -c server-ip --bidir

# UDP test
iperf3 -c server-ip -u

# Parallel connections
iperf3 -c server-ip -P 4
```

**iperf3 output explained:**
```
Connecting to host 192.168.1.100, port 5201
[  5] local 192.168.1.50 port 54321 connected to 192.168.1.100 port 5201
[ ID] Interval           Transfer     Bitrate         Retr  Cwnd
[  5]   0.00-1.00   sec   112 MBytes   941 Mbits/sec    0    234 KBytes
[  5]   1.00-2.00   sec   113 MBytes   948 Mbits/sec    0    256 KBytes
[  5]   2.00-3.00   sec   111 MBytes   931 Mbits/sec    0    278 KBytes
...
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.10 GBytes   945 Mbits/sec    0             sender
[  5]   0.00-10.00  sec  1.10 GBytes   943 Mbits/sec                  receiver
                                           └─ Actual throughput

iperf Done.
```

**Practical tests:**
```bash
# Test max bandwidth
iperf3 -c server-ip

# Test with realistic packet size
iperf3 -c server-ip -M 1460

# Test over 60 seconds for stability
iperf3 -c server-ip -t 60

# Test both directions
iperf3 -c server-ip --bidir

# Save results to file
iperf3 -c server-ip --json > results.json
```

---

### ping: Latency Testing

**Measure round-trip time**

**Basic usage:**
```bash
# Default (continuous)
ping google.com

# Send 4 packets
ping -c 4 google.com

# Set packet size
ping -s 1000 google.com

# Set interval (default: 1s)
ping -i 0.2 google.com

# Flood ping (max speed, root only)
sudo ping -f google.com
```

**Analyzing ping output:**
```bash
ping -c 10 google.com

# Output:
PING google.com (142.250.185.46) 56(84) bytes of data.
64 bytes from lga34s34-in-f14.1e100.net (142.250.185.46): icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from lga34s34-in-f14.1e100.net (142.250.185.46): icmp_seq=2 ttl=117 time=13.1 ms
                                                                                │
                                                                                └─ Latency

--- google.com ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9013ms
                                    └─ Reliability
rtt min/avg/max/mdev = 11.234/12.567/14.890/1.123 ms
    │    │    │    └─ Standard deviation (jitter)
    │    │    └─ Maximum latency
    │    └─ Average latency
    └─ Minimum latency
```

**Latency interpretation:**
```
< 1 ms     - Same datacenter / localhost
1-10 ms    - Same city
10-50 ms   - Same country
50-100 ms  - Nearby country
100-200 ms - Different continent
> 200 ms   - Slow / far / congested

Packet loss:
0%       - Excellent
< 1%     - Good
1-5%     - Acceptable
> 5%     - Poor (investigate!)
```

---

### mtr: My TraceRoute

**Combines ping + traceroute with statistics**

**Installation:**
```bash
sudo apt install mtr  # Ubuntu/Debian
sudo yum install mtr  # RHEL/CentOS
```

**Usage:**
```bash
# Interactive mode
mtr google.com

# Report mode (10 cycles)
mtr -r -c 10 google.com

# Show both hostnames and IPs
mtr -b google.com

# No DNS resolution
mtr -n google.com

# CSV output
mtr --csv google.com

# JSON output
mtr --json google.com
```

**mtr output explained:**
```
                                 My traceroute  [v0.93]
server1 (192.168.1.50)                          Thu Jan 23 14:30:00 2026
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                           Packets               Pings
 Host                                    Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. _gateway (192.168.1.1)                0.0%    10    1.2   1.3   1.1   1.8   0.2
 2. 10.0.0.1                              0.0%    10    8.5   8.7   8.1   9.5   0.4
 3. 203.0.113.1                           0.0%    10   12.3  12.5  11.8  14.2   0.8
 4. ???                                  100.0%    10    0.0   0.0   0.0   0.0   0.0
 5. google.com (142.250.185.46)           0.0%    10   13.1  13.4  12.8  15.1   0.7
    │                                     │       │     │     │     │     │     │
    │                                     │       │     │     │     │     │     └─ Std deviation
    │                                     │       │     │     │     │     └─ Worst latency
    │                                     │       │     │     │     └─ Best latency
    │                                     │       │     │     └─ Average latency
    │                                     │       │     └─ Last ping
    │                                     │       └─ Packets sent
    │                                     └─ Packet loss %
    └─ Hop number
```

**Use cases:**
```bash
# Identify where latency increases
mtr target.com

# Find packet loss location
mtr -r -c 100 target.com

# Monitor over time
mtr --report-cycles 1000 target.com > mtr-log.txt
```

---

## 🔎 Packet Capture and Analysis

### tcpdump: Packet Sniffer

**Capture and analyze network packets**

**Basic capture:**
```bash
# Capture on default interface
sudo tcpdump

# Capture on specific interface
sudo tcpdump -i eth0

# Capture with more detail
sudo tcpdump -v

# Very verbose
sudo tcpdump -vv
```

**Save to file:**
```bash
# Save capture
sudo tcpdump -w capture.pcap

# Read from file
tcpdump -r capture.pcap

# Save and display
sudo tcpdump -w capture.pcap -v
```

**Filters:**
```bash
# Specific host
sudo tcpdump host 192.168.1.100

# Specific port
sudo tcpdump port 80

# Specific protocol
sudo tcpdump tcp
sudo tcpdump udp
sudo tcpdump icmp

# Source/destination
sudo tcpdump src 192.168.1.100
sudo tcpdump dst 192.168.1.100

# Combinations
sudo tcpdump 'tcp port 80 and host 192.168.1.100'

# HTTP traffic
sudo tcpdump 'tcp port 80 or tcp port 443'

# Exclude SSH (useful when connected via SSH)
sudo tcpdump 'not port 22'
```

**Advanced examples:**
```bash
# Capture HTTP POST requests
sudo tcpdump -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# Capture DNS queries
sudo tcpdump -i eth0 -n port 53

# Capture syn packets (new connections)
sudo tcpdump 'tcp[tcpflags] & (tcp-syn) != 0'

# Capture traffic between two hosts
sudo tcpdump 'host 192.168.1.50 and host 192.168.1.100'

# Capture to file, rotate every 100MB
sudo tcpdump -i eth0 -w capture.pcap -C 100
```

---

## 🔧 Connection Monitoring

### ss: Socket Statistics

**(Covered in 13-linux-network-commands.md, quick recap)**

**Active connections:**
```bash
# All TCP connections
ss -ta

# All listening ports
ss -tln

# Show process names
sudo ss -tlnp

# Show statistics
ss -s
```

**Watch connections in real-time:**
```bash
# Update every 1 second
watch -n 1 'ss -s'

# Monitor specific port
watch -n 1 'ss -tan | grep :80'
```

---

### netstat (deprecated but still useful)

```bash
# Active connections
netstat -an

# Listening ports
netstat -tln

# Show program names
sudo netstat -tlnp

# Statistics
netstat -s

# Routing table
netstat -r
```

---

## 📊 System Network Statistics

### /proc/net files

**Interface statistics:**
```bash
# Quick look at interface stats
cat /proc/net/dev

# Output:
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo: 12345678  123456    0    0    0     0          0         0 12345678  123456    0    0    0     0       0          0
  eth0: 987654321 1234567    0    0    0     0          0      1234 456789012 2345678    0    0    0     0       0          0
```

**Parse interface stats:**
```bash
#!/bin/bash
# Show RX/TX in human-readable format

while read line; do
  if [[ $line =~ ^[[:space:]]*(eth[0-9]|ens[0-9]+): ]]; then
    iface=$(echo $line | awk '{print $1}' | tr -d ':')
    rx_bytes=$(echo $line | awk '{print $2}')
    tx_bytes=$(echo $line | awk '{print $10}')
    echo "$iface - RX: $(numfmt --to=iec-i --suffix=B $rx_bytes), TX: $(numfmt --to=iec-i --suffix=B $tx_bytes)"
  fi
done < /proc/net/dev
```

---

## 🚀 DevOps Monitoring Best Practices

### What to Monitor:

**Infrastructure:**
```
✅ Bandwidth usage (current/peak/trends)
✅ Latency to critical services
✅ Packet loss
✅ Active connections
✅ Error rates (retransmissions, drops)
✅ DNS resolution time
```

**Application:**
```
✅ API response times
✅ Database connection pool
✅ Message queue depth
✅ Cache hit rates
✅ External service latency
```

### Monitoring Scripts:

**Bandwidth monitoring:**
```bash
#!/bin/bash
# monitor-bandwidth.sh

INTERFACE="eth0"
INTERVAL=5

echo "Monitoring $INTERFACE every ${INTERVAL}s"
echo "Press Ctrl+C to stop"
echo

while true; do
  RX_BEFORE=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
  TX_BEFORE=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
  
  sleep $INTERVAL
  
  RX_AFTER=$(cat /sys/class/net/$INTERFACE/statistics/rx_bytes)
  TX_AFTER=$(cat /sys/class/net/$INTERFACE/statistics/tx_bytes)
  
  RX_RATE=$(( ($RX_AFTER - $RX_BEFORE) / $INTERVAL ))
  TX_RATE=$(( ($TX_AFTER - $TX_BEFORE) / $INTERVAL ))
  
  echo "$(date +%H:%M:%S) - RX: $(numfmt --to=iec-i --suffix=B/s $RX_RATE), TX: $(numfmt --to=iec-i --suffix=B/s $TX_RATE)"
done
```

**Connection monitoring:**
```bash
#!/bin/bash
# monitor-connections.sh

echo "Monitoring TCP connections"
echo

while true; do
  ESTABLISHED=$(ss -tan | grep ESTAB | wc -l)
  LISTEN=$(ss -tln | wc -l)
  TIME_WAIT=$(ss -tan | grep TIME-WAIT | wc -l)
  
  echo "$(date +%H:%M:%S) - Established: $ESTABLISHED, Listening: $LISTEN, TIME_WAIT: $TIME_WAIT"
  
  sleep 5
done
```

**Alert on high latency:**
```bash
#!/bin/bash
# alert-latency.sh <target> <threshold-ms>

TARGET=$1
THRESHOLD=${2:-100}

while true; do
  LATENCY=$(ping -c 1 $TARGET | grep 'time=' | awk -F'time=' '{print $2}' | awk '{print $1}')
  
  if (( $(echo "$LATENCY > $THRESHOLD" | bc -l) )); then
    echo "ALERT: High latency to $TARGET: ${LATENCY}ms (threshold: ${THRESHOLD}ms)"
    # Send alert email/slack/etc
  fi
  
  sleep 60
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **Which tool shows bandwidth usage per process?**
   <details>
   <summary>Answer</summary>
   nethogs
   </details>

2. **How do you test bandwidth between two servers?**
   <details>
   <summary>Answer</summary>
   Use iperf3: Run `iperf3 -s` on server, `iperf3 -c server-ip` on client
   </details>

3. **What does mtr combine?**
   <details>
   <summary>Answer</summary>
   ping + traceroute with statistics
   </details>

4. **How do you capture HTTP traffic with tcpdump?**
   <details>
   <summary>Answer</summary>
   `sudo tcpdump -i eth0 'tcp port 80 or tcp port 443'`
   </details>

---

## 📝 Key Takeaways

✅ **iftop** - Real-time bandwidth per connection  
✅ **nethogs** - Bandwidth per process  
✅ **vnstat** - Historical bandwidth statistics  
✅ **iperf3** - Bandwidth testing between hosts  
✅ **mtr** - Traceroute with statistics  
✅ **tcpdump** - Packet capture and analysis  
✅ Monitor: bandwidth, latency, packet loss, connections  
✅ Use monitoring scripts for automation  
✅ Alert on thresholds (high latency, packet loss)  

---

## 🚀 Next Steps

Congratulations! You've completed **Phase 3: Practical Skills**! 🎉

You now know:
- All essential Linux networking commands
- DNS deep dive
- Firewall configuration
- Network monitoring and performance testing

**Next:** Phase 4 - DevOps-Specific Topics

**Next lesson:** `18-docker-networking.md`

---

## 💡 Pro Tip

**Create network health dashboard:**
```bash
#!/bin/bash
# network-health.sh

clear
while true; do
  tput cup 0 0
  echo "=== Network Health Dashboard ==="
  echo "Time: $(date)"
  echo
  echo "=== Interfaces ==="
  ip -br addr
  echo
  echo "=== Bandwidth (last 5 sec) ==="
  # Calculate bandwidth
  echo
  echo "=== Connections ==="
  echo "Established: $(ss -tan | grep ESTAB | wc -l)"
  echo "Listening: $(ss -tln | wc -l)"
  echo
  echo "=== Latency Tests ==="
  echo "Google DNS: $(ping -c 1 8.8.8.8 2>/dev/null | grep 'time=' | awk -F'time=' '{print $2}' | awk '{print $1}') ms"
  echo
  echo "Press Ctrl+C to exit"
  sleep 5
done
```

One-glance network status! 📊
