## Topology

```
[Cafe-PC]                                        [Fallout-SRV]
192.168.1.50                                     192.168.3.100
     |                                                 |
[Cafe-RT1]──────── WAN 192.168.2.0/30 ────────[Fallout-RT1]
 E0/0: 192.168.1.1                              E0/0: 192.168.3.1
 E0/1: 192.168.2.1                              E0/1: 192.168.2.2
 E0/2: 216.0.5.2 (ISP)
```
![[ccna-summer-2026/day_25/lab/Network_Diagram_1.png]]
---

## Objective

Read and decode a live routing table on Cafe-RT1 — identifying route codes, administrative distance, metric, next-hop, and exit interface. Demonstrate how AD overrides metric when two routes compete for the same prefix. Observe variably subnetted entries and local host routes using temporary loopback interfaces.

---

## Task 0 — Decode the Routing Table

### Baseline routing table — Cafe-RT1

```
Gateway of last resort is 216.0.5.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 216.0.5.1
      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/30 is directly connected, Ethernet0/1
L        192.168.2.1/32 is directly connected, Ethernet0/1
D     192.168.3.0/24 [90/409600] via 192.168.2.2, 00:00:25, Ethernet0/1
      216.0.5.0/24 is variably subnetted, 2 subnets, 2 masks
C        216.0.5.0/30 is directly connected, Ethernet0/2
L        216.0.5.2/32 is directly connected, Ethernet0/2
```

### EIGRP route decoded — field by field

```
D     192.168.3.0/24 [90/409600] via 192.168.2.2, 00:00:25, Ethernet0/1
│     │              │  │         │               │          │
│     │              │  │         │               │          └─ Exit interface
│     │              │  │         │               └─ Route age (how long known)
│     │              │  │         └─ Next hop IP
│     │              │  └─ Metric (path quality — lower is better)
│     │              └─ Administrative Distance (believability — lower is better)
│     └─ Destination network and prefix length
└─ Learned via EIGRP (D = DUAL algorithm)
```

| Field | Value | Meaning |
|-------|-------|---------|
| Code | `D` | Learned via EIGRP |
| Destination | `192.168.3.0/24` | The Fallout Shelter LAN |
| AD | `90` | EIGRP's believability score |
| Metric | `409600` | EIGRP's composite path quality score |
| Next hop | `192.168.2.2` | Send packet to Fallout-RT1's WAN interface |
| Age | `00:00:25` | Router has known this route for 25 seconds |
| Exit interface | `Ethernet0/1` | Packet leaves via the WAN link |

---

## Task 1 — Compare Administrative Distance and Metric

### Step 1 — Add a competing static route to the same /24

```
Cafe-RT1# conf t
Cafe-RT1(config)# ip route 192.168.3.0 255.255.255.0 192.168.2.2
```

### Step 2 — Observe the routing table

```
D     192.168.3.0/24 [90/409600] via 192.168.2.2, 00:03:03, Ethernet0/1
S     192.168.3.0/24 [1/0] via 192.168.2.2
```

Both routes exist in the router's knowledge base. Only **one** gets installed as active. The static route wins because AD 1 < AD 90.

> The EIGRP route disappears from the active routing table — but EIGRP has not forgotten it. It is held internally and will reappear the moment the static route is removed.

### Step 3 — Remove the static route and verify EIGRP returns

```
Cafe-RT1(config)# no ip route 192.168.3.0 255.255.255.0
```

After removal:
```
D     192.168.3.0/24 [90/409600] via 192.168.2.2, 00:00:05, Ethernet0/1
```

EIGRP route immediately reinstalled. Age reset to near zero — confirms it was held in EIGRP's topology table the entire time, not re-learned from scratch.

### Route selection summary

```
Same destination — two competing routes:

  Static:  192.168.3.0/24 [1/0]      ← AD 1  (wins)
  EIGRP:   192.168.3.0/24 [90/409600] ← AD 90 (loses, hidden but not forgotten)

Decision layer used: Administrative Distance
Metric not consulted — AD settled it first
```

---

## Task 2 — Variably Subnetted and Host Routes

### Create two loopback interfaces within 10.0.0.0/8

```
Cafe-RT1(config)# interface loopback 1
Cafe-RT1(config-if)# ip address 10.0.0.1 255.255.255.0

Cafe-RT1(config)# interface loopback 2
Cafe-RT1(config-if)# ip address 10.0.1.1 255.255.255.0
```

Both loopbacks came up immediately — loopback interfaces are always up/up as long as they exist.

### Routing table after loopbacks added

```
      10.0.0.0/8 is variably subnetted, 4 subnets, 2 masks
C        10.0.0.0/24 is directly connected, Loopback1
L        10.0.0.1/32 is directly connected, Loopback1
C        10.0.1.0/24 is directly connected, Loopback2
L        10.0.1.1/32 is directly connected, Loopback2
```

### What each line means

| Entry | Type | Meaning |
|-------|------|---------|
| `10.0.0.0/8 is variably subnetted` | Header | IOS acknowledging the classful block is broken into subnets of different sizes |
| `C 10.0.0.0/24` | Connected | Loopback1's network — traffic for this subnet exits here |
| `L 10.0.0.1/32` | Local | Router's own IP on Loopback1 — process locally, don't forward |
| `C 10.0.1.0/24` | Connected | Loopback2's network |
| `L 10.0.1.1/32` | Local | Router's own IP on Loopback2 |

**"Variably subnetted"** means the router has multiple subnets of different sizes carved out of the same classful block (`10.0.0.0/8`). Here: two /24s and two /32s — hence "2 masks."

The `/32 local routes are bookkeeping entries**, not extra remote networks. They exist so the router can identify traffic addressed to its own interface IPs and process it locally instead of forwarding it.

### Remove loopbacks and restore baseline

```
Cafe-RT1(config)# no interface loopback1
Cafe-RT1(config)# no interface loopback2
```

Both interfaces went administratively down immediately. Routing table returned to original state.

---

## What Each Task Confirmed

| Task | What it proved |
|------|---------------|
| Task 0 — Decode routing table | Every field of a route entry has a specific meaning — code, prefix, AD, metric, next-hop, age, exit interface |
| Task 1 — AD comparison | Static (AD 1) beats EIGRP (AD 90) for same prefix; EIGRP route disappears but is retained internally |
| Task 1 — EIGRP restoration | Removing static route instantly reinstalls EIGRP route — proves it was never forgotten |
| Task 2 — Loopback routes | Variably subnetted header and /32 local routes are IOS bookkeeping, not extra remote networks |

---

## Key Lessons

**The three-layer route selection process:**
```
1. Most specific prefix wins (longest match)
       ↓ tie?
2. Lowest Administrative Distance wins
       ↓ tie?
3. Lowest Metric wins
```

**Administrative Distance reference:**

| Source | AD |
|--------|----|
| Connected | 0 |
| Static | 1 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |

**The hidden route trap:**
When a static route beats EIGRP for the same prefix, EIGRP's route vanishes from the routing table but lives on in EIGRP's topology table. Remove the static route — it reappears instantly. The router never forgot it.

---

## Mistakes Made

**Wrong subnet mask on static route (`255.255.255.255` instead of `255.255.255.0`)**

Typed `ip route 192.168.3.0 255.255.255.255` which created a /32 host route to the network address itself — not a /24 network route. The routing table showed both the EIGRP /24 and the static /32 coexisting, since they were different prefixes. The `no` command to remove it also had to match the /32 exactly.

**Lesson:** `no ip route` must match the exact network and mask that was originally entered. A /24 `no` command will not remove a /32 entry and vice versa.

**Tried to assign a network address as an interface IP**

Attempts like `ip address 10.0.1.0 255.255.255.0` were rejected with `Bad mask`. The `.0` last octet is the network address for a /24 — not a valid host address. IOS rejects it. Valid host addresses for `10.0.1.0/24` run from `10.0.1.1` through `10.0.1.254`.

**Lesson:** Interface IPs must be host addresses, not network or broadcast addresses.

**Overlap error on second loopback**

Attempting `10.0.0.2/24` on Loopback2 failed with `overlaps with Loopback1` because both addresses fall inside `10.0.0.0/24`, which Loopback1 already owned. Two interfaces cannot share the same subnet.

**Lesson:** Each interface must be in a unique, non-overlapping subnet.
