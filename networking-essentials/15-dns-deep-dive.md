# 15 - DNS Deep Dive

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- How DNS works in detail
- DNS record types and their uses
- DNS caching and TTL
- Troubleshooting DNS issues
- DNS in DevOps (service discovery, load balancing)

---

## 🤔 What is DNS? (Recap)

**DNS = Domain Name System**

```
Human-Friendly:  google.com
Computer Needs:  142.250.185.46

DNS translates between them!
```

**Real-World Analogy:**
```
Phone book:
Name: John Smith  →  Phone: 555-1234
DNS:
Domain: google.com  →  IP: 142.250.185.46
```

---

## 🏗️ DNS Hierarchy

DNS is structured like an upside-down tree:

```
                        . (Root)
                         │
        ┌────────────────┼────────────────┐
       .com             .org             .net
        │                │                │
    ┌───┼───┐         ┌──┼──┐         ┌──┼──┐
google amazon ms    wikip open      cloud stack
    │                 edia ource       flare over
   www                                   flow
```

### DNS Zones:
```
Root Zone:          .
Top-Level Domain:   .com, .org, .net, .io, .dev
Second-Level:       google.com, amazon.com
Subdomain:          www.google.com, mail.google.com
```

---

## 🔍 DNS Resolution Process (Step-by-Step)

### When you type `www.example.com` in browser:

```
Step 1: Browser checks its cache
  ├─ Found? Use it! ✅
  └─ Not found? Continue...

Step 2: Operating system checks /etc/hosts
  ├─ Found? Use it! ✅
  └─ Not found? Continue...

Step 3: OS checks DNS cache
  ├─ Found? Use it! ✅
  └─ Not found? Continue...

Step 4: Query local DNS resolver (usually ISP or 8.8.8.8)
  
Step 5: Recursive DNS server queries:
  
  a) Root server (.)
     Query: "Where is .com?"
     Response: "Ask the .com TLD servers at 192.5.6.30"
  
  b) TLD server (.com)
     Query: "Where is example.com?"
     Response: "Ask ns1.example.com at 93.184.216.34"
  
  c) Authoritative server (example.com)
     Query: "What's the IP for www.example.com?"
     Response: "www.example.com is 93.184.216.34"

Step 6: Recursive server caches result and returns to you

Step 7: Your computer caches result

Step 8: Browser connects to 93.184.216.34
```

### Visual Diagram:
```
Your Computer          Recursive DNS       Root DNS        TLD DNS       Authoritative DNS
(192.168.1.100)        (8.8.8.8)          (.)             (.com)        (ns1.example.com)
      │                    │                 │               │                 │
      │─ www.example.com? ─→                 │               │                 │
      │                    │─ Where is .com?─→               │                 │
      │                    │←─ Ask 192.5.6.30─               │                 │
      │                    │─ Where is example.com? ─────────→                 │
      │                    │←─ Ask ns1.example.com ──────────                  │
      │                    │─ IP for www.example.com? ───────────────────────→ │
      │                    │←─ 93.184.216.34 ─────────────────────────────────│
      │← 93.184.216.34 ───│                 │               │                 │
      │                    │                 │               │                 │
```

---

## 📋 DNS Record Types

### A Record (Address)
**Maps domain to IPv4 address**
```
example.com.  300  IN  A  93.184.216.34
│             │    │   │  │
│             │    │   │  └── IPv4 address
│             │    │   └───── Record type
│             │    └───────── Class (Internet)
│             └────────────── TTL (Time To Live in seconds)
└──────────────────────────── Domain name
```

**Usage:**
```bash
dig example.com A

# Multiple A records (round-robin load balancing)
example.com.  300  IN  A  93.184.216.34
example.com.  300  IN  A  93.184.216.35
example.com.  300  IN  A  93.184.216.36
```

### AAAA Record (IPv6 Address)
**Maps domain to IPv6 address**
```
example.com.  300  IN  AAAA  2606:2800:220:1:248:1893:25c8:1946
```

```bash
dig example.com AAAA
```

### CNAME Record (Canonical Name)
**Alias one domain to another**
```
www.example.com.  300  IN  CNAME  example.com.
blog.example.com. 300  IN  CNAME  wordpress-host.com.
```

**Usage:**
```
User accesses: www.example.com
DNS resolves: www.example.com → example.com → 93.184.216.34

Common use: CDN
cdn.mysite.com  IN  CNAME  mycdn.cloudfront.net.
```

**Important:** CNAME cannot coexist with other records for same name!
```
❌ example.com  IN  A      93.184.216.34
    example.com  IN  CNAME  other.com.     ← INVALID!

✅ www.example.com  IN  CNAME  example.com.
   example.com      IN  A      93.184.216.34
```

### MX Record (Mail Exchange)
**Specifies mail servers**
```
example.com.  300  IN  MX  10 mail1.example.com.
example.com.  300  IN  MX  20 mail2.example.com.
│                      │   │  │
│                      │   │  └── Mail server hostname
│                      │   └───── Priority (lower = preferred)
│                      └───────── Record type
└──────────────────────────────── Domain
```

**Lower priority number = higher priority:**
```
MX  10 mail1.example.com.  ← Try this first
MX  20 mail2.example.com.  ← Backup if first fails
```

```bash
dig example.com MX
```

### TXT Record (Text)
**Arbitrary text data - many uses**
```
example.com.  300  IN  TXT  "v=spf1 include:_spf.google.com ~all"
example.com.  300  IN  TXT  "google-site-verification=abc123..."
```

**Common uses:**
- SPF (email sender verification)
- DKIM (email authentication)
- Domain verification
- General notes

```bash
dig example.com TXT
```

### NS Record (Name Server)
**Specifies authoritative DNS servers**
```
example.com.  300  IN  NS  ns1.example.com.
example.com.  300  IN  NS  ns2.example.com.
```

```bash
dig example.com NS
```

### PTR Record (Pointer - Reverse DNS)
**Maps IP address to domain name**
```
34.216.184.93.in-addr.arpa.  300  IN  PTR  example.com.
```

**Usage:**
```bash
# Reverse lookup
dig -x 93.184.216.34

# Or
host 93.184.216.34
```

**Important for:**
- Email servers (many require valid reverse DNS)
- Logging (see hostnames instead of IPs)

### SRV Record (Service)
**Specifies location of services**
```
_service._protocol.domain.  TTL  IN  SRV  priority weight port target

Example:
_http._tcp.example.com.  300  IN  SRV  10 50 80 server1.example.com.
```

**Used by:**
- Active Directory
- XMPP/Jabber
- SIP
- Kubernetes service discovery

### SOA Record (Start of Authority)
**Defines zone parameters**
```
example.com.  IN  SOA  ns1.example.com. admin.example.com. (
                         2024012301  ; Serial
                         7200        ; Refresh
                         3600        ; Retry
                         1209600     ; Expire
                         300         ; Minimum TTL
                       )
```

---

## ⏱️ TTL: Time To Live

**How long DNS resolvers should cache a record**

```
example.com.  300  IN  A  93.184.216.34
              │
              └─ TTL = 300 seconds (5 minutes)
```

### How TTL Works:
```
1. DNS query resolves: example.com → 93.184.216.34 (TTL: 300)
2. Resolver caches result for 300 seconds
3. For next 5 minutes, returns cached answer (no new query)
4. After 300 seconds, cache expires
5. Next query triggers new DNS lookup
```

### Choosing TTL:
```
Short TTL (60-300 seconds):
✅ Quick propagation of changes
✅ Good for migrations, A/B testing
❌ More DNS queries (higher load)

Long TTL (3600-86400 seconds):
✅ Fewer DNS queries (less load, faster)
✅ Better performance
❌ Slow propagation of changes

Typical values:
300 (5 min)   - During migrations
3600 (1 hour) - Normal operation
86400 (1 day) - Rarely-changing records
```

---

## 🔧 DNS Commands Deep Dive

### dig (Domain Information Groper)

**Basic query:**
```bash
dig example.com

# Short answer only
dig +short example.com
# Output: 93.184.216.34

# Query specific type
dig example.com A
dig example.com AAAA
dig example.com MX
dig example.com TXT
dig example.com NS
```

**Using specific DNS server:**
```bash
# Query Google DNS
dig @8.8.8.8 example.com

# Query Cloudflare DNS
dig @1.1.1.1 example.com

# Query authoritative server
dig @ns1.example.com example.com
```

**Advanced queries:**
```bash
# Trace full DNS path
dig +trace example.com

# No recursion (query this server only)
dig +norecurs example.com

# Show query time
dig example.com | grep "Query time"

# Reverse DNS lookup
dig -x 93.184.216.34

# ANY query (all records)
dig example.com ANY

# Get DNSSEC info
dig +dnssec example.com
```

**Reading dig output:**
```bash
dig example.com

# Output explained:
;; QUESTION SECTION:
;example.com.                   IN      A
  └── What was asked

;; ANSWER SECTION:
example.com.            86400   IN      A       93.184.216.34
│                       │       │       │       │
└── Domain              └TTL    └Class  └Type   └IP

;; Query time: 23 msec
   └── How long query took

;; SERVER: 8.8.8.8#53(8.8.8.8)
   └── Which DNS server answered

;; WHEN: Thu Jan 23 14:30:00 UTC 2026
   └── When query was made

;; MSG SIZE  rcvd: 56
   └── Response size in bytes
```

### nslookup

```bash
# Simple query
nslookup example.com

# Use specific DNS server
nslookup example.com 8.8.8.8

# Interactive mode
nslookup
> set type=MX
> example.com
> exit

# Reverse lookup
nslookup 93.184.216.34
```

### host

```bash
# Simple query
host example.com

# Verbose
host -v example.com

# Specific record type
host -t MX example.com
host -t NS example.com

# Reverse lookup
host 93.184.216.34
```

---

## 🐛 DNS Troubleshooting

### Problem: Domain Not Resolving

**Step 1: Test with dig**
```bash
dig example.com

# Look for:
# - ANSWER SECTION (should have records)
# - status: NOERROR (not NXDOMAIN or SERVFAIL)
```

**Step 2: Check local DNS config**
```bash
cat /etc/resolv.conf

# Should show:
nameserver 8.8.8.8
nameserver 8.8.4.4
```

**Step 3: Test with different DNS server**
```bash
# Try Google DNS
dig @8.8.8.8 example.com

# Try Cloudflare DNS
dig @1.1.1.1 example.com

# If works with external DNS but not local:
# → Local DNS server issue
```

**Step 4: Check DNS server reachability**
```bash
# Can you reach DNS server?
ping -c 4 8.8.8.8

# Is DNS port open?
nc -zvu 8.8.8.8 53
```

**Step 5: Flush DNS cache**
```bash
# systemd-resolved
sudo systemd-resolve --flush-caches

# nscd
sudo systemctl restart nscd

# dnsmasq
sudo systemctl restart dnsmasq
```

### Problem: Wrong IP Returned

**Cached stale data:**
```bash
# Clear local cache
sudo systemd-resolve --flush-caches

# Check TTL
dig example.com | grep -A1 "ANSWER SECTION"
# Wait for TTL to expire

# Force new query (bypass cache)
dig +trace example.com
```

### Problem: Slow DNS Resolution

**Diagnose:**
```bash
# Measure query time
dig example.com | grep "Query time"

# Output:
;; Query time: 234 msec  ← Slow!

# Test different DNS servers
for dns in 8.8.8.8 1.1.1.1 208.67.222.222; do
  echo "Testing $dns:"
  dig @$dns example.com | grep "Query time"
done

# Trace full path
dig +trace example.com
```

**Solutions:**
```bash
# Use faster DNS server
echo "nameserver 1.1.1.1" | sudo tee /etc/resolv.conf

# Run local caching resolver (dnsmasq)
sudo apt install dnsmasq
```

### Problem: Intermittent DNS Failures

**Diagnose:**
```bash
# Continuous testing
while true; do
  dig +short example.com || echo "FAILED at $(date)"
  sleep 1
done

# Monitor DNS server health
watch -n 1 'dig @8.8.8.8 google.com | grep "Query time"'
```

---

## 🏢 DNS in DevOps

### Service Discovery

**Traditional:**
```
Application → Hardcoded IP: 10.0.1.50:3000
Problem: IP changes, app breaks!
```

**DNS-based:**
```
Application → DNS: api.internal.company.com
DNS returns: 10.0.1.50
IP changes? Just update DNS!
```

**Example: Internal DNS**
```bash
# /etc/hosts (simple, for few servers)
10.0.1.50   api.internal.company.com
10.0.1.51   db.internal.company.com

# Or run internal DNS server (BIND, dnsmasq)
```

### Load Balancing with DNS

**Round-Robin DNS:**
```
api.example.com.  300  IN  A  10.0.1.50
api.example.com.  300  IN  A  10.0.1.51
api.example.com.  300  IN  A  10.0.1.52

Each query returns all IPs, clients pick one (roughly balanced)
```

**Geographic DNS:**
```
Users in US:    api.example.com → 192.0.2.10 (US server)
Users in EU:    api.example.com → 198.51.100.10 (EU server)
Users in Asia:  api.example.com → 203.0.113.10 (Asia server)
```

### Kubernetes DNS

**Kubernetes creates DNS automatically:**
```
Service: myapp
Namespace: production

DNS names created:
- myapp.production.svc.cluster.local
- myapp.production.svc
- myapp.production
- myapp (from same namespace)
```

**Example:**
```bash
# From pod in same namespace
curl http://myapp:8080

# From different namespace
curl http://myapp.production.svc.cluster.local:8080
```

### Docker DNS

**Docker creates internal DNS:**
```bash
docker network create mynetwork
docker run -d --name web --network mynetwork nginx
docker run -d --name app --network mynetwork alpine sleep 3600

# From 'app' container:
docker exec app ping web
# Works! 'web' resolves to container IP
```

### DNS for Monitoring

**Health checks via DNS:**
```bash
# Check if service DNS is resolving
while true; do
  dig +short api.example.com > /dev/null 2>&1
  if [ $? -ne 0 ]; then
    echo "ALERT: DNS resolution failed!" | mail -s "DNS Alert" admin@example.com
  fi
  sleep 60
done
```

---

## 🎯 Quick Check: Do You Understand?

1. **What's the difference between A and CNAME records?**
   <details>
   <summary>Answer</summary>
   A record maps domain to IP address. CNAME is an alias pointing to another domain name.
   </details>

2. **What does TTL control?**
   <details>
   <summary>Answer</summary>
   How long DNS resolvers cache a record before querying again.
   </details>

3. **How do you test DNS resolution?**
   <details>
   <summary>Answer</summary>
   `dig example.com` or `nslookup example.com`
   </details>

4. **What's reverse DNS used for?**
   <details>
   <summary>Answer</summary>
   Map IP address back to hostname. PTR record. Important for email servers.
   </details>

---

## 📝 Key Takeaways

✅ DNS translates domain names to IP addresses  
✅ DNS hierarchy: Root → TLD → Domain → Subdomain  
✅ **A record** = IPv4, **AAAA** = IPv6, **CNAME** = alias  
✅ **MX** = mail servers, **TXT** = verification/SPF  
✅ **TTL** controls cache duration  
✅ Use `dig` for detailed DNS queries  
✅ `dig +trace` shows full resolution path  
✅ DNS used for service discovery in microservices  
✅ Always flush cache when troubleshooting  

---

## 🚀 Next Steps

You understand DNS thoroughly! Next, let's learn about firewalls and securing your network.

**Next lesson:** `16-firewall-basics.md`

---

## 💡 Pro Tip

**Create DNS troubleshooting script:**
```bash
#!/bin/bash
# dns-check.sh <domain>

DOMAIN=$1
echo "=== DNS Check for $DOMAIN ==="
echo
echo "1. A Record:"
dig +short $DOMAIN A
echo
echo "2. AAAA Record:"
dig +short $DOMAIN AAAA
echo
echo "3. MX Records:"
dig +short $DOMAIN MX
echo
echo "4. NS Records:"
dig +short $DOMAIN NS
echo
echo "5. Query Time:"
dig $DOMAIN | grep "Query time"
echo
echo "6. SOA Record:"
dig $DOMAIN SOA +short
```

Save time troubleshooting! 🎯
