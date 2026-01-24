---
render_with_liquid: false
---
# 03 - Binary Basics

## 🎯 Learning Objectives
By the end of this lesson, you'll understand:
- What binary is and why it's used
- How to read binary numbers
- How binary relates to IP addresses
- Why binary matters for subnetting

---

## 🤔 Why Binary? (The Simple Answer)

**Computers only understand two states: ON and OFF**

Think about it:
- A light switch: ON or OFF
- A circuit: OPEN or CLOSED
- Electricity: FLOWING or NOT FLOWING

```
ON  = 1
OFF = 0

That's binary!
```

---

## 🔢 Decimal vs Binary

### What You Already Know: Decimal (Base 10)

We use 10 digits: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9

```
The number 247 means:
  2     4     7
  ↓     ↓     ↓
2×100 + 4×10 + 7×1 = 247

Position value: 100  10  1
```

### What Computers Use: Binary (Base 2)

Only 2 digits: 0 and 1

```
The number 101 (binary) means:
  1    0    1
  ↓    ↓    ↓
1×4 + 0×2 + 1×1 = 5 (decimal)

Position value: 4   2   1
```

---

## 📊 Binary Place Values

Just like decimal has ones, tens, hundreds... binary has powers of 2:

```
Position:    8th  7th  6th  5th  4th  3rd  2nd  1st
            ─────────────────────────────────────────
Value:       128   64   32   16    8    4    2    1
            ─────────────────────────────────────────
Binary:        ?    ?    ?    ?    ?    ?    ?    ?
```

**Each position is worth 2× the previous position!**

---

## 🎓 Converting Binary to Decimal

### Example 1: Simple Binary Number

Convert `1011` to decimal:

```
Position value:  8    4    2    1
Binary:          1    0    1    1
                 ↓    ↓    ↓    ↓
Calculation:    8 +  0 +  2 +  1  = 11

Answer: 1011 (binary) = 11 (decimal)
```

### Example 2: Larger Number

Convert `11010110` to decimal:

```
Position:   128  64   32   16    8    4    2    1
Binary:       1   1    0    1    0    1    1    0
              ↓   ↓    ↓    ↓    ↓    ↓    ↓    ↓
Calc:       128+ 64+  0 + 16+  0 +  4 +  2 +  0  = 214

Answer: 11010110 (binary) = 214 (decimal)
```

### Quick Method:
```
Only add the positions where you see a "1"
Ignore positions where you see a "0"
```

---

## 🔄 Converting Decimal to Binary

### Method: Repeated Division by 2

Convert 13 to binary:

```
13 ÷ 2 = 6  remainder 1  ← Rightmost bit
 6 ÷ 2 = 3  remainder 0
 3 ÷ 2 = 1  remainder 1
 1 ÷ 2 = 0  remainder 1  ← Leftmost bit

Read remainders from bottom to top: 1101

Answer: 13 (decimal) = 1101 (binary)
```

### Visual Representation:
```
128  64  32  16   8   4   2   1
  0   0   0   0   1   1   0   1
                  ↑   ↑       ↑
                  8 + 4 + 0 + 1 = 13
```

---

## 📱 Common Binary Patterns (Memorize These!)

These patterns appear constantly in networking:

```
Decimal  →  Binary      (Why it matters)
─────────────────────────────────────────
0        →  00000000    (Minimum value)
128      →  10000000    (Half of 255)
192      →  11000000    (Common subnet mask)
224      →  11100000    (Subnet mask)
240      →  11110000    (Subnet mask)
248      →  11111000    (Subnet mask)
252      →  11111100    (Subnet mask)
254      →  11111110    (Subnet mask)
255      →  11111111    (Maximum value, all bits ON)
```

---

## 🌐 Binary and IP Addresses

### IP Address Structure

An IPv4 address has **4 octets** (4 sets of 8 bits):

```
Decimal:     192   .   168   .    1    .   10
             ↓         ↓         ↓         ↓
Binary:   11000000.10101000.00000001.00001010
          └──8 bits─┘└─8 bits─┘└─8 bits─┘└─8 bits─┘
          
Total: 32 bits (4 octets × 8 bits)
```

### Why 0-255 Range?

With 8 bits, you can represent:
```
Minimum: 00000000 = 0
Maximum: 11111111 = 255

2^8 = 256 possible values (0 through 255)
```

---

## 🔍 Practical Example: Understanding an IP Address

Let's decode `192.168.1.10`:

### Octet 1: 192
```
128  64  32  16   8   4   2   1
  1   1   0   0   0   0   0   0
  ↓   ↓
128+ 64 = 192
```

### Octet 2: 168
```
128  64  32  16   8   4   2   1
  1   0   1   0   1   0   0   0
  ↓       ↓       ↓
128+    32+     8 = 168
```

### Octet 3: 1
```
128  64  32  16   8   4   2   1
  0   0   0   0   0   0   0   1
                              ↓
                              1
```

### Octet 4: 10
```
128  64  32  16   8   4   2   1
  0   0   0   0   1   0   1   0
                  ↓       ↓
                  8   +   2 = 10
```

### Full IP in Binary:
```
192      .168      .1        .10
11000000.10101000.00000001.00001010
```

---

## 🎯 Why This Matters for Subnetting

Subnetting is all about manipulating bits. Here's a preview:

### Network vs Host Portion

```
IP:           192.168.1.10
Binary:       11000000.10101000.00000001.00001010

Subnet Mask:  255.255.255.0
Binary:       11111111.11111111.11111111.00000000
              └─────Network Bits─────┘└─Host Bits┘

Network:      192.168.1.0    (Hosts = 0s)
Hosts:        192.168.1.1 through 192.168.1.254
Broadcast:    192.168.1.255  (All Host Bits = 1s)
```

**We'll dive deep into this in the subnetting lesson!**

---

## 🧮 Binary Math Basics

### AND Operation (Used in Subnetting)

```
Rule: Result is 1 only if BOTH are 1

  1 AND 1 = 1
  1 AND 0 = 0
  0 AND 1 = 0
  0 AND 0 = 0
```

### Example: Finding Network Address

```
IP Address:    192.168.1.10
Binary:        11000000.10101000.00000001.00001010

Subnet Mask:   255.255.255.0
Binary:        11111111.11111111.11111111.00000000

AND Operation:
               11000000.10101000.00000001.00001010
            AND 11111111.11111111.11111111.00000000
               ────────────────────────────────────
Network:       11000000.10101000.00000001.00000000
Decimal:       192      168      1        0
```

**This is how routers determine if two IPs are on the same network!**

---

## 📝 Practice Exercises

### Exercise 1: Binary to Decimal

Convert these to decimal:
```
1. 00001010 = ?
2. 11111111 = ?
3. 10000001 = ?
4. 11000000 = ?
```

<details>
<summary>Click for answers</summary>

```
1. 00001010 = 10   (8 + 2)
2. 11111111 = 255  (128+64+32+16+8+4+2+1)
3. 10000001 = 129  (128 + 1)
4. 11000000 = 192  (128 + 64)
```
</details>

### Exercise 2: Decimal to Binary

Convert these to binary (8 bits):
```
1. 16  = ?
2. 255 = ?
3. 1   = ?
4. 240 = ?
```

<details>
<summary>Click for answers</summary>

```
1. 16  = 00010000
2. 255 = 11111111
3. 1   = 00000001
4. 240 = 11110000
```
</details>

### Exercise 3: IP Address to Binary

Convert this IP to binary:
```
10.0.0.1 = ?
```

<details>
<summary>Click for answer</summary>

```
10.0.0.1 = 00001010.00000000.00000000.00000001
```
</details>

---

## 🎲 Binary Patterns Recognition

Learn to recognize these patterns instantly:

### Consecutive 1s (From Left)
```
10000000 = 128   (1 bit ON)
11000000 = 192   (2 bits ON)
11100000 = 224   (3 bits ON)
11110000 = 240   (4 bits ON)
11111000 = 248   (5 bits ON)
11111100 = 252   (6 bits ON)
11111110 = 254   (7 bits ON)
11111111 = 255   (8 bits ON)
```

**These are your subnet masks!**

### Powers of 2
```
00000001 = 1    (2^0)
00000010 = 2    (2^1)
00000100 = 4    (2^2)
00001000 = 8    (2^3)
00010000 = 16   (2^4)
00100000 = 32   (2^5)
01000000 = 64   (2^6)
10000000 = 128  (2^7)
```

---

## 💼 Why This Matters for DevOps

### Scenario 1: Troubleshooting IP Conflicts
```
Question: "Are 192.168.1.50 and 192.168.2.50 on the same network?"

Convert to binary:
192.168.1.50  = 11000000.10101000.00000001.00110010
192.168.2.50  = 11000000.10101000.00000010.00110010
                                        ↑
                                    Different octet!

Answer: NO (assuming /24 subnet mask)
```

### Scenario 2: Calculating Available IPs
```
Subnet: 192.168.1.0/24

/24 means first 24 bits are network, last 8 are hosts
8 bits for hosts = 2^8 = 256 addresses
Minus network (192.168.1.0) and broadcast (192.168.1.255)
= 254 usable IP addresses
```

### Scenario 3: Designing Container Networks
```
Docker network: 172.17.0.0/16

/16 = 16 bits for hosts
2^16 = 65,536 addresses
Plenty of room for thousands of containers!
```

---

## 🎯 Quick Reference Table

```
Bits  Decimal  Binary      Power of 2
────────────────────────────────────
1     128      10000000    2^7
2     192      11000000    2^7 + 2^6
3     224      11100000    2^7 + 2^6 + 2^5
4     240      11110000    2^7 + 2^6 + 2^5 + 2^4
5     248      11111000    ...
6     252      11111100
7     254      11111110
8     255      11111111    2^8 - 1
```

---

## 🎯 Quick Check: Do You Understand?

1. **Why do computers use binary?**
   <details>
   <summary>Answer</summary>
   Computers only understand two states: ON (1) and OFF (0). Binary perfectly represents this.
   </details>

2. **What's the decimal value of 10101010?**
   <details>
   <summary>Answer</summary>
   128 + 32 + 8 + 2 = 170
   </details>

3. **How many different values can 8 bits represent?**
   <details>
   <summary>Answer</summary>
   2^8 = 256 values (0-255)
   </details>

4. **Why is 255 the maximum value for an IP octet?**
   <details>
   <summary>Answer</summary>
   An octet is 8 bits. All 8 bits set to 1 (11111111) equals 255.
   </details>

---

## 🧠 Mental Math Shortcuts

### Quick Conversion Tricks:

**128 or higher?** → First bit is 1
```
192 > 128 → Starts with 1: 1???????
```

**Even or odd?** → Look at last bit
```
Even: Last bit = 0 (10, 12, 14)
Odd:  Last bit = 1 (11, 13, 15)
```

**Recognize multiples of 2:**
```
2, 4, 8, 16, 32, 64, 128 → One bit set
```

---

## 📝 Key Takeaways

✅ Binary uses only 0 and 1  
✅ Each position is a power of 2 (1, 2, 4, 8, 16, 32, 64, 128)  
✅ 8 bits (1 byte) can represent 0-255  
✅ IP addresses are 32 bits (4 octets of 8 bits each)  
✅ Binary is essential for understanding subnetting  
✅ AND operation finds network addresses  
✅ Common patterns (192, 224, 255) are subnet masks  

---

## 🚀 Next Steps

You now have the binary foundation needed for IP addressing and subnetting! Next, let's learn about the physical components of networks.

**Next lesson:** `04-network-hardware.md`

---

## 💡 Pro Tip

**Don't try to memorize everything!**

Focus on:
- Converting common numbers (192, 168, 255, 0, 10)
- Recognizing patterns
- Understanding the AND operation

The rest will come with practice! 🎉
