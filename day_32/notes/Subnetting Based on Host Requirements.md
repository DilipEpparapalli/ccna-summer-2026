## Simple Explanation
Instead of asking "how many subnets do I need?", you ask "how many hosts per subnet do I need?" You protect enough host bits to fit your devices, then let the remaining bits become subnet bits. Same three-step process — different starting point.

## Why It Exists
In real network design, requirements come in the form of device counts, not subnet counts. "This branch office needs to support 50 devices" is a host requirement. Knowing how to subnet from that starting point is how production IP plans actually get built.

## How It Works

### The Mental Shift
| Starting point | You're thinking about... | You're protecting... |
|---|---|---|
| Subnet requirement | How many network bits to borrow | Nothing — steal freely |
| **Host requirement** | **How many host bits to keep** | **The right side of the mask** |

> **"Save the host."** If the requirement is devices per subnet, protect the host bits first. Subnet with what's left.

---

### The Process

**Step 1 — Convert host requirement to bits needed**

How many hosts do you need? Find the smallest power of 2 that exceeds that number.

| Requirement | Bits needed | Because |
|---|---|---|
| 50 hosts | 6 bits | 2^6 = 64 > 50 |
| 500 hosts | 9 bits | 2^9 = 512 > 500 |
| 2,000 hosts | 11 bits | 2^11 = 2,048 > 2,000 |

Usable hosts = 2^y − 2 (subtract network address and broadcast).

**Step 2 — Preserve those host bits, build the mask**

Lock the required host bits on the right. Everything else becomes network bits.

**Example: 50 hosts from 172.16.10.0/24**
```
Need 6 host bits → preserve 6 zeros on the right
/24 default → 8 host bits available → steal 2 → /26

255.255.255.11000000 → 255.255.255.192 → /26
Increment = 64 (value of lowest network bit in fourth octet)
Usable hosts = 2^6 − 2 = 62
```

**Example: 500 hosts from 172.16.0.0/16**
```
Need 9 host bits → preserve 9 zeros on the right
/16 default → 16 host bits available → steal 7 → /23

255.255.11111110.00000000 → 255.255.254.0 → /23
Increment = 2 (in third octet)
Usable hosts = 2^9 − 2 = 510
```

**Example: 2,000 hosts from 10.0.0.0/8**
```
Need 11 host bits → preserve 11 zeros on the right
/8 default → 24 host bits available → steal 13 → /21

255.255.11111000.00000000 → 255.255.248.0 → /21
Increment = 8 (in third octet)
Usable hosts = 2^11 − 2 = 2,046
```

**Step 3 — Map the ranges using the increment**

**172.16.10.0/26 (increment = 64 in fourth octet)**
| Subnet | Network | Usable Range | Broadcast |
|---|---|---|---|
| 1 | 172.16.10.0 | .1 – .62 | .63 |
| 2 | 172.16.10.64 | .65 – .126 | .127 |
| 3 | 172.16.10.128 | .129 – .190 | .191 |
| 4 | 172.16.10.192 | .193 – .254 | .255 |

**172.16.0.0/23 (increment = 2 in third octet)**
| Subnet | Network | Broadcast |
|---|---|---|
| 1 | 172.16.0.0 | 172.16.1.255 |
| 2 | 172.16.2.0 | 172.16.3.255 |
| 3 | 172.16.4.0 | 172.16.5.255 |

**10.0.0.0/21 (increment = 8 in third octet)**
| Subnet | Network | Broadcast |
|---|---|---|
| 1 | 10.0.0.0 | 10.0.7.255 |
| 2 | 10.0.8.0 | 10.0.15.255 |
| 3 | 10.0.16.0 | 10.0.23.255 |

---

### Quick Mental Checklist
1. Is the requirement subnets or hosts?
2. If hosts → convert to binary, count bits needed
3. Preserve those bits on the right
4. Build the new mask from what's left
5. Find the increment → map the ranges

## Key Terms
| Term | Meaning |
|---|---|
| Host bits | The rightmost bits of a mask — define how many devices fit per subnet |
| "Save the host" | Mental cue: protect host bits first when requirement is device count |
| 2^y − 2 | Formula for usable hosts (y = host bits remaining) |
| Headroom | Extra address space left intentionally for future device growth |

## Real-World Connection
Production IP planning always starts with device counts. At a branch office, you'd inventory every device — workstations, phones, cameras, printers, APs — add 20–30% headroom for growth, then find the mask that fits. Building subnets too small forces a painful renumbering later. Subnetting from host requirements is how you avoid that.

## Exam Traps
1. **Don't subtract 2 from subnets, only from hosts** — 2^x subnets (no subtraction), 2^y − 2 usable hosts.
2. **The requirement tells you which direction to think** — "subnets needed" → borrow freely. "hosts per subnet" → protect first, subnet with the rest. Mixing these up is the most common mistake.
3. **Always add headroom mentally** — the exam gives you exact numbers, but in a real-world scenario question it may ask for the *minimum* mask that fits. Pick the mask where 2^y − 2 ≥ requirement, not just 2^y.

## Commands
Not applicable — this is a design and planning skill.

## Recall Questions
1. You need subnets that support at least 30 hosts each from 192.168.5.0/24. What mask do you use? What is the increment? How many usable hosts per subnet?
2. You need subnets supporting 100 hosts from 172.31.0.0/16. How many host bits do you preserve? What is the new prefix length?
3. What is the third subnet's network address and broadcast in a 172.16.10.0/26 scheme?
4. Why do you subtract 2 from the host count formula but not from the subnet count formula?
