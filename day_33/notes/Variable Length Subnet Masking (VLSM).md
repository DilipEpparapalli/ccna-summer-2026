## Simple Explanation
VLSM means using different subnet masks for different segments within the same network
design. Instead of forcing every subnet to be the same size, you size each one to match
its actual host requirement. The subnetting process itself does not change — you just
apply it multiple times in one scenario.

## Why It Exists
Real networks have segments of wildly different sizes. A staff LAN might need 50 hosts.
A point-to-point WAN link between two routers needs exactly 2. Giving both the same
subnet mask wastes enormous amounts of address space. VLSM lets you carve a single
address block efficiently so every subnet is sized for its actual need.

## How It Works

**The one rule that prevents disasters: always allocate largest subnet first.**

Large subnets require large contiguous blocks. If you allocate small subnets first, you
fragment the address space and may not be able to fit the larger networks later.

**The process — same three steps as always, repeated per subnet:**

1. How many hosts are needed? → determine required host bits
2. Build the mask → subtract host bits from 32
3. Find the increment → place value of first host bit → map the ranges

**Worked example — carving 192.168.10.0/24**

Requirements:
- 2 networks × 50 hosts
- 2 networks × 20 hosts
- 2 WAN point-to-point links × 2 hosts

---

**Step 1 — 50-host networks → /26 (increment 64)**

6 host bits → 2⁶ = 64 addresses, 62 usable hosts

```
Network 1:  192.168.10.0/26    → 192.168.10.0   – 192.168.10.63
Network 2:  192.168.10.64/26   → 192.168.10.64  – 192.168.10.127
```

**Step 2 — 20-host networks → /27 (increment 32)**

5 host bits → 2⁵ = 32 addresses, 30 usable hosts

```
Network 3:  192.168.10.128/27  → 192.168.10.128 – 192.168.10.159
Network 4:  192.168.10.160/27  → 192.168.10.160 – 192.168.10.191
```

**Step 3 — WAN point-to-point links → /30 (increment 4)**

2 host bits → 2² = 4 addresses, 2 usable hosts — perfect for router-to-router links

```
Network 5:  192.168.10.192/30  → 192.168.10.192 – 192.168.10.195
Network 6:  192.168.10.196/30  → 192.168.10.196 – 192.168.10.199
```

**Remaining unallocated space: 192.168.10.200 – 192.168.10.255**
This is intentional — VLSM preserves unused space for future growth.

---

**Full allocation map:**

```
192.168.10.0/24  (256 addresses)
├── 192.168.10.0/26    [Network 1 — 50 hosts]   .0   – .63
├── 192.168.10.64/26   [Network 2 — 50 hosts]   .64  – .127
├── 192.168.10.128/27  [Network 3 — 20 hosts]   .128 – .159
├── 192.168.10.160/27  [Network 4 — 20 hosts]   .160 – .191
├── 192.168.10.192/30  [Network 5 — WAN link]   .192 – .195
├── 192.168.10.196/30  [Network 6 — WAN link]   .196 – .199
└── 192.168.10.200–255 [Unallocated — 56 addresses available]
```

## Key Terms

| Term | Meaning |
|------|---------|
| VLSM | Using different subnet masks for different subnets within one design |
| /30 | The standard mask for point-to-point WAN links — 2 usable hosts |
| Host bits | Bits borrowed from the right side of the mask to create host addresses |
| Increment | Block size for a given mask — place value of the first host bit |
| Contiguous block | A range of addresses that hasn't been fragmented by prior allocations |

## Real-World Connection
In an enterprise network, a single /24 might serve an entire branch office. That branch
has a staff LAN, a guest WiFi segment, a server VLAN, management interfaces, and WAN
uplinks. Each of those is a different size. VLSM lets the network engineer allocate
exactly what each segment needs from the same /24 pool — no separate address blocks
required, no waste, and room left for future segments.

Point-to-point WAN links are almost universally /30 in real deployments for exactly
this reason — two router interfaces, two usable addresses, nothing wasted.

## Exam Traps

1. **Allocating small subnets before large ones will paint you into a corner.** The exam
   may give you a scenario where someone made this mistake and ask you to identify why
   the design failed or what address space is unavailable.

2. **A /30 gives you 4 addresses, 2 usable.** Don't confuse "2 host bits" with
   "2 addresses total." Network and broadcast still consume one each.

3. **Each subnet in a VLSM design must not overlap with any other.** After allocating,
   verify that the end address of one subnet is strictly less than the start of the next.
   Overlapping ranges are a silent misconfiguration — no error fires until traffic breaks.

## Recall Questions

1. You have 172.16.0.0/16. You need subnets for 500 hosts, 100 hosts, 50 hosts, and a
   WAN link. What masks do you use, and in what order do you allocate?
2. Why does allocating largest-first prevent design failures?
3. A /30 WAN link — what are the network address, two usable host addresses, and
   broadcast for the block starting at 10.0.0.8?
4. You've allocated four /27 subnets from 192.168.1.0/24. What is the next available
   address for a new subnet?
