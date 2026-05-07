
## Simple Explanation
A WAN connects an organization's separate locations across distance. This is NOT always the internet —private carrier networks, dedicated fiber, and encrypted tunnels are all valid WAN options. The right choice depends on cost, performance, and business need.

## Why It Exists
Businesses centralize services — phone systems, databases, payment systems, email — in a data center or corporate HQ. Every branch office needs a reliable path back to those services. That path is the WAN.

## LAN vs WAN

LAN = everything inside one location (private)
WAN = everything connecting separate locations

The internet is ONE type of WAN — not the only one.

## WAN Technologies — Evolution

### 1. Leased Lines (Legacy)
A dedicated physical circuit rented from a carrier. Only your traffic. Guaranteed performance.

  [Branch] ────dedicated cable──── [HQ]

- T1 = 1.54 Mbps
- T3 = 43 Mbps
- Private and reliable but painfully slow and expensive
- Connecting many sites = many separate lines = nightmare
- Still exists in the wild

---

### 2. MPLS — Multiprotocol Label Switching

Each site connects once to the carrier's private network. The carrier handles all routing between
your locations internally.

  [Dallas HQ] ──┐
  [Phoenix]  ───┼──► [MPLS Cloud] ← carrier routes here
  [Seattle]  ───┘
  [Data Center]─┘

How it works:
- Traffic enters the carrier network via CE router (Customer Edge — your router)
- Carrier receives it at PE router (Provider Edge — carrier's router)
- Carrier applies a LABEL to your traffic
- Label logically isolates your traffic from other customers on the same physical infrastructure
- Feels like a private dedicated network

Key properties:
- NOT the internet — private carrier network
- Layer 2.5 — label sits between L2 and L3 headers
- Built-in QoS — voice and critical traffic prioritized
- Expensive but reliable and predictable

CE and PE Router:
  [Your Site]──[CE Router]──[PE Router]──[MPLS Cloud]
                ^ yours        ^ carrier's

Why MPLS is dying:
- Expensive
- Cloud apps (AWS, Azure) don't live in your data center — so private paths back to your DC matter less
- SD-WAN offers similar quality at lower cost

---

### 3. Metro Ethernet (Metro E)

High-speed fiber connection between major sites within a metropolitan area.

  [Corporate HQ] ══ fiber ══ [Data Center]
        1 Gbps or 10 Gbps dedicated

- Layer 2 — feels like extending a giant switch between two buildings
- Best for: HQ ↔ Data Center, or DC ↔ DC
- Too expensive for every branch office
- NOT a replacement for MPLS — different use case

Three flavors:

| Type   | Description              | Model          |
|--------|--------------------------|----------------|
| E-Line | Two sites, one connection| Point-to-point |
| E-LAN  | All sites connected      | Full mesh      |
| E-Tree | Branches connect to hub  | Hub and spoke  |

---

### 4. Internet + Site-to-Site VPN

Regular business internet connection with encrypted
tunnel between sites.

  [Branch]──[Internet]──encrypted tunnel──[Data Center]

- Much cheaper than MPLS
- Traffic encrypted so it's safe crossing public internet
- Unreliable — packets get no special treatment
- Congestion, latency, and jitter affect voice calls
- No built-in QoS — critical traffic competes with
  everything else
- The cheap option that often causes headaches

---

### 5. QoS — Quality of Service

Not a WAN technology — a traffic prioritization mechanism used across WAN solutions.

  Voice calls  ──► Priority lane — always first
  Payroll data ──► High priority
  Email        ──► Normal
  Streaming    ──► Last in line

MPLS handled QoS natively. Internet VPNs traditionally could not. SD-WAN brings it back over regular internet.

---

### 6. SD-WAN — Software Defined WAN

Uses regular internet connections but adds intelligence on top. The modern MPLS replacement.

- Smart path selection — picks best route automatically
- Built-in QoS — prioritizes critical traffic
- Optimizes connections directly to cloud apps
- Much cheaper than MPLS
- Solves the problems of basic internet VPNs

Why SD-WAN is winning:
- Traffic increasingly goes to cloud (AWS, Azure) not back to the data center
- No need for expensive private circuits when intelligent internet routing achieves the same result

---

## WAN Technology Comparison

| Technology      | Private? | Speed      | Cost     | Best For              |
|-----------------|----------|------------|----------|-----------------------|
| Leased Line     | Yes      | Very slow  | High     | Legacy point-to-point |
| MPLS            | Yes      | Varies     | High     | Many branch offices   |
| Metro Ethernet  | Yes      | 1-10 Gbps  | High     | HQ ↔ Data Center      |
| Internet VPN    | No*      | Varies     | Low      | Budget branches       |
| SD-WAN          | No*      | Varies     | Low-Med  | Modern cloud-first    |

 *encrypted but not private carrier network

## Real-World Design Logic

One building, one site:
  → LAN only, no WAN needed

Corporate HQ ↔ Data Center (same city):
  → Metro Ethernet (fast, dedicated fiber)

HQ + many branch offices across the country:
  → MPLS (private, QoS, one connection per site)
  → or SD-WAN (cheaper, cloud-optimized)

Small branch on a budget:
  → Internet + Site-to-Site VPN

## Exam Traps
1. WAN ≠ internet. MPLS and Metro E are WANs that never touch the public internet.
2. MPLS is Layer 2.5 — the label sits between the Layer 2 and Layer 3 headers. Not purely either.
3. Metro E is NOT replacing MPLS. They serve different purposes. SD-WAN is replacing MPLS.
4. Site-to-site VPN encrypts traffic — MPLS does NOT necessarily encrypt. MPLS uses label isolation, not encryption, to keep traffic private.
5. CE router = yours. PE router = carrier's. Know which side of the boundary each sits on.

## Recall Questions
1. What is the difference between a WAN and the internet?
2. A company has 15 branch offices across the country.They want private connectivity with QoS for voice traffic. Which WAN technology fits best and why?
3. What does MPLS label switching actually do? Why is it called Layer 2.5?
4. What is the difference between E-Line, E-LAN, and E-Tree?
5. Why is SD-WAN replacing MPLS? Name two reasons.
6. What problem does site-to-site VPN solve? What problem does it NOT solve?