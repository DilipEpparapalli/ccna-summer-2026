## Simple Explanation
Class B and Class A subnetting use the exact same three-step process as Class C. What changes is the default mask and how much host space you have to work with. The only new challenge is staying calm when the increment crosses an octet boundary.

## Why It Exists
Most enterprise networks aren't Class C. ISPs, large organizations, and cloud providers work with Class A and B space. If you can only subnet within one octet, you're limited. Real-world design means confidently borrowing bits across octets and mapping ranges that span multiple third and fourth octet values.

## How It Works

### Default Masks by Class
| Class | Example     | Default Mask        | Host Octets |
| ----- | ----------- | ------------------- | ----------- |
| A     | 10.0.0.0    | /8 (255.0.0.0)      | 3 octets    |
| B     | 172.16.0.0  | /16 (255.255.0.0)   | 2 octets    |
| C     | 192.168.1.0 | /24 (255.255.255.0) | 1 octet     |

---

### Same Three Steps — Every Time

**Step 1 — Convert subnet requirement to bits needed**

| Requirement | Bits needed | Because |
|---|---|---|
| 100 subnets | 7 bits | 2^7 = 128 ≥ 100 |
| 500 subnets | 9 bits | 2^9 = 512 ≥ 500 |
| 1,000 subnets | 10 bits | 2^10 = 1,024 ≥ 1,000 |

**Step 2 — Add bits to default mask, find the increment**

The increment = decimal value of the lowest borrowed network bit, evaluated per octet independently.

**Class B Example — 100 subnets from 172.16.0.0/16**
```
Borrow 7 bits → /23 → 255.255.254.0

Third octet after borrowing 7 bits:
11111110 → value of lowest bit = 2 → increment = 2 (in third octet)
```

**Class B Example — 500 subnets from 172.20.0.0/16**
```
Borrow 9 bits → /25 → 255.255.255.128

Borrows all 8 bits of third octet + 1 bit of fourth octet:
Fourth octet: 10000000 → value of lowest bit = 128 → increment = 128 (in fourth octet)
```

**Class A Example — 1,000 subnets from 10.0.0.0/8**
```
Borrow 10 bits → /18 → 255.255.192.0

Third octet after borrowing 2 bits: 11000000 → lowest bit = 64 → increment = 64 (in third octet)
```

**Step 3 — Map the ranges using the increment**

**172.16.0.0/23 (increment = 2 in third octet)**
| Subnet | Network | Usable Range | Broadcast |
|---|---|---|---|
| 1 | 172.16.0.0 | .0.1 – .1.254 | 172.16.1.255 |
| 2 | 172.16.2.0 | .2.1 – .3.254 | 172.16.3.255 |
| 3 | 172.16.4.0 | .4.1 – .5.254 | 172.16.5.255 |

**172.20.0.0/25 (increment = 128 in fourth octet)**
| Subnet | Network | Broadcast |
|---|---|---|
| 1 | 172.20.0.0 | 172.20.0.127 |
| 2 | 172.20.0.128 | 172.20.0.255 |
| 3 | 172.20.1.0 | 172.20.1.127 ← rollover |

**10.0.0.0/18 (increment = 64 in third octet)**
| Subnet | Network | Broadcast |
|---|---|---|
| 1 | 10.0.0.0 | 10.0.63.255 |
| 2 | 10.0.64.0 | 10.0.127.255 |
| 3 | 10.0.128.0 | 10.0.191.255 |
| 4 | 10.0.192.0 | 10.0.255.255 |
| 5 | 10.1.0.0 | 10.1.63.255 ← rollover |

---

### The Octet Rollover Rule
When the increment is in the fourth octet and adding it hits 256 — it rolls over into the third octet.

```
172.20.0.128 + 128 = 172.20.0.256 → doesn't exist
                   → rolls to 172.20.1.0  ✓
```

This is not an exception. This is the rule.

---

### The Oreo Rule
> Only the **first** and **last** address in a subnet are reserved. Everything in the middle is usable.

An address ending in `.0` or `.255` is **not automatically reserved** — it depends on where the subnet boundaries fall. If it lands in the middle of a range, it's a valid host address. Your eyes will lie to you. Trust the math.

## Key Terms
| Term | Meaning |
|---|---|
| Octet boundary | The point where borrowing bits moves from one octet to the next |
| Rollover | When adding the increment to the fourth octet exceeds 255, it carries into the third octet |
| /23 | 255.255.254.0 — spans 2 blocks of the third octet |
| /25 | 255.255.255.128 — splits the fourth octet in half |
| /18 | 255.255.192.0 — 64-block increment in the third octet |

## Real-World Connection
Private Class A space (10.0.0.0/8) is used by nearly every large enterprise and cloud provider. AWS VPCs, Azure VNets, and corporate networks carve /18s, /20s, and /22s out of that space daily. When you're designing VLANs at Castle Rysen's scale or looking at your Pi lab's routing table, understanding where the subnet boundary actually sits — especially across octets — is what separates a working network from a mysterious one.

## Exam Traps
1. **".0 or .255 = reserved" is wrong** — only the network address and broadcast of *that specific subnet* are reserved. An address ending in .0 or .255 can be a valid host if it falls inside the usable range.
2. **The increment resets per octet** — if the lowest borrowed bit lands in the third octet with value 2, the increment is 2, not 2×256. Each octet is evaluated independently.
3. **Rollover catches people off guard** — when the fourth octet increment pushes past 255, the carry goes to the third octet. 172.20.0.128 + 128 = 172.20.1.0, not an error.

## Commands
Not applicable — this is a design and planning skill.

## Recall Questions
1. You need 200 subnets from 172.30.0.0/16. How many bits do you borrow? What is the new mask? What is the increment and which octet does it live in?
2. In a 172.20.0.0/25 scheme, what is the network address and broadcast of the third subnet?
3. Is 172.16.3.0 a valid host address in a /23 subnet starting at 172.16.2.0? Why?
4. You're subnetting 10.0.0.0/8 into /18 blocks. What is the fifth subnet's network address?
