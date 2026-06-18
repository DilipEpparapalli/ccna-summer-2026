## Simple Explanation
Real-world subnetting starts with business structure, not just host counts. You size
subnets for actual device density, infrastructure, and growth — then group them
contiguously so the addressing plan mirrors the business hierarchy and enables route
summarization later.

## Why It Exists
Exam subnetting gives you clean numbers. Real networks give you headcounts, growth plans,
and hidden infrastructure. An addressing plan that only counts humans will run out of
space the moment the business starts acting like a real business — cameras, IoT devices,
servers, VMs, wireless APs, and badge readers all silently consume IP addresses.

## How It Works

### Mindset Shift: Humans ≠ IP Addresses
A single person can carry 3–5 connected devices. Beyond personal devices, every site has:
- Servers and virtual machines
- IP cameras and physical security systems
- IoT and embedded devices (badge readers, door controllers, POS terminals)
- Wireless infrastructure (APs, controllers)
- Guest network segments
- Future growth and new services

**Never size a subnet by headcount alone. Size it by realistic device density plus
a growth buffer.**

### Choosing the Right Private Range
From RFC 1918 private address space:

| Range | Size | Usable hosts | Best for |
|-------|------|-------------|----------|
| 10.0.0.0/8 | Class A | ~16.7 million | Large enterprise |
| 172.16.0.0/12 | Class B | ~1 million | Mid-size org |
| 192.168.0.0/16 | Class C | ~65,000 | Small office / home lab |

Castle Rysen uses **10.0.0.0/8** — not because they need 16 million addresses today,
but because the design has room to scale without renumbering later. Renumbering a live
network is a travesty nobody enjoys.

### The Castle Rysen Design

**Business structure:**
- 1 Central Office (CO) — regional hub, ~200 humans but full infrastructure
- 30 Fallout Shelters (FS) — sub-regional hubs, ~50 humans each
- Each FS manages 50 District Shops (DS) — ~15 humans each

**Sizing with growth buffer:**

| Site type | Realistic device count | Mask chosen | Addresses |
|-----------|----------------------|-------------|-----------|
| Central Office | ~4,000 | /20 | 4,096 |
| Fallout Shelter | ~500 | /23 | 512 |
| District Shop | ~50 | /26 | 64 |

### The Allocation — Largest First

**Central Office:**
```
10.0.0.0/20     → 10.0.0.0  – 10.0.15.255   (4,096 addresses)
```

**Fallout Shelter 1 + its 50 District Shops (grouped contiguously):**
```
10.0.16.0/23    → 10.0.16.0 – 10.0.17.255   (FS 1 — 512 addresses)
10.0.18.0/26    → District Shop 1
10.0.18.64/26   → District Shop 2
10.0.18.128/26  → District Shop 3
10.0.18.192/26  → District Shop 4
...
(50 /26 blocks continuing through 10.0.31.255)
```

**Entire FS 1 region fits cleanly into 10.0.16.0 – 10.0.31.255**

**Fallout Shelter 2 starts at 10.0.32.0** — clean boundary, next region begins.

The pattern repeats for each of the 30 fallout shelters.

### Why Contiguous Grouping Matters — Route Summarization

Scattered allocation forces a router to carry one route per subnet:
```
10.0.16.0/23    ← FS 1
10.0.18.0/26    ← DS 1
10.0.18.64/26   ← DS 2
... (51 separate routing table entries per region)
```

Contiguous allocation lets the router represent the entire FS 1 region as **one route**:
```
10.0.16.0/19    ← entire FS 1 region (FS + all 50 shops)
```

One routing table entry instead of 51. Multiply that across 30 fallout shelters and the
efficiency gain is enormous. The addressing plan does two jobs at once: it fits the
business structure **and** makes routing efficient.

**The zip code analogy:** A mail sorting facility doesn't need every street address in
a city — it just knows "anything starting with 641 goes to this truck." Route
summarization works the same way. Contiguous blocks create that hierarchy.

## Key Terms

| Term | Meaning |
|------|---------|
| Route summarization | Representing many specific networks as one larger route in a routing table |
| Contiguous block | A range of addresses with no gaps — required for clean summarization |
| Growth buffer | Allocating more addresses than current headcount to accommodate future devices |
| RFC 1918 | The standard defining private IPv4 address ranges (10.x, 172.16.x, 192.168.x) |
| Renumbering | Reassigning IP addresses across a live network — expensive and disruptive |

## Real-World Connection
Enterprise network designers think in regions before they think in subnets. A large
retailer with hundreds of branches follows exactly this pattern — each region gets a
contiguous block, each branch gets a sized subnet within that block, and the WAN routing
table stays manageable because summarization collapses regional complexity into single
routes. The addressing plan is the foundation every other network service is built on.

## Exam Traps

1. **Headcount is not device count.** If an exam question says "50 users" and asks you
   to size a subnet, account for multiple devices per user plus infrastructure — don't
   just pick the mask that fits 50 hosts.

2. **Contiguous allocation is a prerequisite for summarization.** If subnets for a
   region are scattered across non-contiguous address space, you cannot summarize them
   into a single route without also capturing unrelated networks.

3. **Choosing too small a private range is a design failure, not just inefficiency.**
   A 192.168.0.0/16 base for an enterprise that grows to thousands of subnets will
   require a full renumber. Pick the range that fits the scale of the business, not
   the scale of today's spreadsheet.

## Recall Questions

1. A client says their office has 80 employees. What questions do you ask before sizing
   the subnet — and what realistic device count might you actually design for?
2. Why does grouping a fallout shelter and its district shops into one contiguous block
   matter for routing? What specific problem does it solve?
3. What mask would you use for a site needing ~500 addresses? Show the host bit math.
4. A network engineer allocates district shop subnets randomly across the 10.0.0.0/8
   space instead of grouping them by region. What breaks later, and why?
