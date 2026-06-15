## Simple Explanation
Subnetting is taking one large network and slicing it into smaller pieces. You do this by stealing bits from the host portion of the subnet mask and converting them into network bits. More network bits = more subnets, but fewer hosts per subnet. That's always the tradeoff.

## Why It Exists
A flat network with hundreds of devices is inefficient, hard to manage, and a security risk. Real deployments — branch offices, kiosks, point-to-point WAN links — need networks sized to fit the actual requirement. Subnetting lets you carve IP space precisely instead of wasting it.

## How It Works

### The Core Idea
> "Subnetting is all about giving up hosts so you can have more networks."

In a subnet mask: **1 = network bit, 0 = host bit**. When you subnet, you flip host bits into network bits.

---

### The Three-Step Process

**Step 1 — Convert your subnet requirement to binary, count the bits needed**

How many subnets do you need? Convert that number to binary. You don't care about the exact pattern — you care how many bits it takes to represent it.

| Requirement | Binary | Bits needed |
|---|---|---|
| 25 subnets | 11001 | 5 bits |
| 60 subnets | 111100 | 6 bits |

Formula: **2^x ≥ requirement**, where x = subnet bits stolen.

---

**Step 2 — Reserve bits in the mask, find the increment**

Take a /24 (Class C) and steal x bits from the host side:

```
Original /24:
11111111.11111111.11111111.00000000  →  255.255.255.0

Steal 5 bits (for 25 subnets):
11111111.11111111.11111111.11111000  →  255.255.255.248  →  /29

Steal 6 bits (for 60 subnets):
11111111.11111111.11111111.11111100  →  255.255.255.252  →  /30
```

**The increment** = the decimal value of the lowest network bit (the last `1` in the mask).
- /29 → last network bit = **8** → increment is 8
- /30 → last network bit = **4** → increment is 4

---

**Step 3 — List the subnet ranges using the increment**

Start at the base network address, add the increment repeatedly. Each block is one subnet.

**Example: 192.168.10.0/29 (increment = 8)**

| Subnet | Network Address | Usable Hosts | Broadcast |
|---|---|---|---|
| 1 | 192.168.10.0 | .1 – .6 | .7 |
| 2 | 192.168.10.8 | .9 – .14 | .15 |
| 3 | 192.168.10.16 | .17 – .22 | .23 |
| ... | ... | ... | ... |

**Example: 216.5.10.0/30 (increment = 4)**

| Subnet | Network Address | Usable Hosts | Broadcast |
|---|---|---|---|
| 1 | 216.5.10.0 | .1 – .2 | .3 |
| 2 | 216.5.10.4 | .5 – .6 | .7 |
| 3 | 216.5.10.8 | .9 – .10 | .11 |
| ... | ... | ... | ... |

---

### The Formulas

| Formula | What it gives you |
|---|---|
| 2^x | Number of subnets (x = subnet bits stolen) |
| 2^y − 2 | Usable hosts per subnet (y = remaining host bits) |

Always subtract 2 from hosts — first address is the network address, last is broadcast.

## Key Terms
| Term | Meaning |
|------|---------|
| Subnet bits | Host bits stolen and converted to network bits |
| Increment | Decimal value of the lowest network bit; tells you where each subnet starts |
| Network address | First address in a subnet — not assignable to a host |
| Broadcast address | Last address in a subnet — not assignable to a host |
| /29 | 255.255.255.248 — 6 usable hosts per subnet |
| /30 | 255.255.255.252 — 2 usable hosts per subnet |

## Real-World Connection
A /30 is one of the most common masks in production networking. Any point-to-point link between two routers — a WAN connection, a site-to-site link — only needs 2 usable addresses. Using a /24 there would waste 252 addresses. On your Pi lab, if you ever set up a routed link between two VMs or containers, /30 is the right call.

## Exam Traps
1. **2^x vs 2^x − 2** — subnets use 2^x (no subtraction). Hosts use 2^y − 2 (subtract network + broadcast). Don't mix them up.
2. **The increment comes from the mask, not the requirement** — always derive it from the lowest network bit in binary, not from the number of subnets you asked for.
3. **First and last addresses are never usable** — network address and broadcast address are reserved in every subnet, no exceptions.

## Commands
Not applicable at this stage — subnetting is a design/planning skill before it's a CLI skill.

## Recall Questions
1. You need 10 subnets from 192.168.1.0/24. How many bits do you steal? What is the new mask? What is the increment?
2. What is the usable host range of the third subnet in a 192.168.1.0/29 scheme?
3. Why is /30 so common in real-world routing?
4. A /29 gives you how many usable hosts per subnet? Show the math.
