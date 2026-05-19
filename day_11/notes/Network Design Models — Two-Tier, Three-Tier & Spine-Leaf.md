
## 1. Why Design Models Exist

Plugging switches together without a plan works — until it doesn't.
The moment a business depends on a network, organic growth becomes a liability.

**The organic growth trap:**
- Router → switch → another switch → daisy-chained chaos
- One cable kicked = half the network down
- No redundancy, no recovery plan, no future-proofing

Design models give you a **blueprint before you touch hardware.**

---

## 2. SOHO Model (Not Really a Model)

**Small Office / Home Office** — one router, one or two switches.

Fine for home labs and tiny setups. The moment the business depends
on the network, you've outgrown it.

---

## 3. Three-Tier Architecture

Cisco's foundational enterprise design model. Three layers, every time.

```
                    [ CORE LAYER ]
                    /             \
        [Dist-SW-A] ----------- [Dist-SW-B]    ← DISTRIBUTION LAYER
          / | \                   / | \
         /  |  \                 /  |  \
    [Acc-1][Acc-2][Acc-3]  [Acc-4][Acc-5][Acc-6]  ← ACCESS LAYER
      |                              |
  [End devices]                [End devices]
  PCs, phones,                 PCs, phones,
  printers, WAPs               printers, WAPs
```

### Layer Breakdown

| Layer | Job | What lives here |
|-------|-----|-----------------|
| Access | Where devices access the network | PCs, printers, IP phones, WAPs |
| Distribution | Consolidation + redundancy point | Servers (DNS, DHCP), uplinks from access switches |
| Core | Connects multiple buildings / distribution blocks | High-speed backbone switches |

### The Critical Redundancy Rule

Each access switch connects to **both** distribution switches — not one.

```
     [Dist-SW-A] -------- [Dist-SW-B]
       /  |  \              /  |  \
      /   |   \            /   |   \
 [Acc-1][Acc-2][Acc-3]
```

If Dist-SW-A dies → traffic reroutes through Dist-SW-B automatically.
Two cables to the **same** switch is NOT redundancy. Two cables to **two different** switches is.

---

## 4. Two-Tier (Collapsed Core)

Used when you have **one or two buildings.**

The core layer is collapsed into the distribution layer — one less tier to manage.

```
     [Dist-SW-A] -------- [Dist-SW-B]
       /  |  \              /  |  \
      /   |   \            /   |   \
 [Acc-1][Acc-2][Acc-3]
```

Same diagram as three-tier distribution — just no separate core switches above it.

**Rule of thumb:**
- 1–2 buildings → Two-tier (collapsed core)
- 3+ buildings → Three-tier (add dedicated core)

---

## 5. Why Core Layer Exists — The Full Mesh Problem

Without a core, buildings connect directly to each other (full mesh).

| Buildings | Links needed (full mesh) |
|-----------|--------------------------|
| 2 | 1 |
| 3 | 3 |
| 4 | 6 |
| 8 | 28 |

Full mesh = exponential cable growth + management nightmare.

**Core layer fix:** every building connects to the core. Not to each other.

```
        Building A          Building B
        [Dist block] \    / [Dist block]
                      \  /
                   [CORE]
                      /  \
        [Dist block] /    \ [Dist block]
        Building C          Building D
```

Clean. Scalable. Every new building adds exactly two links (to each core switch).

---

## 6. Spine and Leaf (Data Center)

A different architecture built for **data centers** — not campus networks.

```
  [Spine-1] ----------- [Spine-2]
   / | \ \             / / | \
  /  |  \ \           / /  |  \
[L1][L2][L3][L4]  [L5][L6][L7][L8]
                ↑
          Leaf switches
     (servers connect here)
```

- Every **leaf** connects to every **spine**
- No leaf connects directly to another leaf
- Predictable, low-latency paths between any two servers
- Built for massive east-west traffic (server-to-server)

---

## 7. MDF vs IDF

| Term | Meaning | Where |
|------|---------|-------|
| MDF | Main Distribution Facility | Core/distribution switches, router, ISP connections |
| IDF | Intermediate Distribution Facility | Per-zone/per-floor access layer gear |

In a large building: one MDF + multiple IDFs connected back via fiber.
Ethernet maxes at **100 meters** — fiber bridges the distance between MDF and IDFs.

---

## 8. Real-World Tips

- **Never max out switch ports** — leave 20–30% free. When a switch fails,
  you need spare capacity on another switch to absorb the load temporarily.
- **Intentional design at switch #3** — when you hit your third switch,
  stop and apply the two-tier model. Don't let the network sprawl.
- **Fiber between MDF and IDFs** — copper won't make the distance in large buildings.
- **Redundant uplinks, not redundant cables to the same device** —
  two cables to one switch protects against cable failure only.
  Two cables to two switches protects against device failure too.

---

## 9. Exam Traps

1. **Two cables to the same switch ≠ redundancy.**
   True redundancy requires two separate distribution switches.

2. **Collapsed core = two-tier**, not a broken three-tier.
   It's a deliberate choice for smaller environments.

3. **Spine and leaf is data center only** — don't apply it to campus design questions.

4. **Access layer = end devices only.** Servers live at distribution, not access.

5. **Core layer is about scale, not speed alone.**
   You add core when connecting 3+ buildings — not just because traffic is heavy.

---

## 10. Recall Questions

1. What are the three tiers and what does each one do?
2. Harvey has 4 switches daisy-chained. One gets unplugged — half the network dies.
   What design principle did he violate?
3. You have 5 buildings. Why does full mesh fail and what do you use instead?
4. What is a collapsed core and when do you use it?
5. Each access switch connects to both distribution switches — why?
6. What is the difference between MDF and IDF?
7. Why does spine-and-leaf exist and where does it live?
