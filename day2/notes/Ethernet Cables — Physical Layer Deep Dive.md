
## Simple Explanation

An Ethernet cable carries data as electrical signals across copper wires. The way wires are arranged, twisted, and terminated determines speed, reliability, and which devices can talk to each other.

---

## What's Inside the Cable

```
Outer jacket (sheath)
└── 4 twisted pairs = 8 copper wires total
    ├── Pair 1: Orange + White Orange
    ├── Pair 2: Green + White Green
    ├── Pair 3: Blue + White Blue
    └── Pair 4: Brown + White Brown
```

Copper conducts electricity. Data travels as changing voltage levels — voltage shifts represent binary 1s and 0s across the wire.

---

## Why the Wires Are Twisted — Two Enemies

Twisting fights both enemies simultaneously:

```
Enemy 1 — EMI (Electromagnetic Interference)
  External devices emit electromagnetic fields
  Twisting the pairs cancels out the interference

Enemy 2 — Crosstalk
  Adjacent wire pairs interfering with each other
  Twisting reduces signal bleed between pairs
```

---

## UTP vs STP

```
UTP — Unshielded Twisted Pair
  Jacket + twisted pairs only
  Used in homes and offices
  Most common cable type

STP — Shielded Twisted Pair
  Extra metallic shield around the pairs
  Used in factories and high-EMI environments
  More expensive, heavier, harder to terminate
```

---

## Cable Categories

The Cat number refers to the quality of the **individual copper wires** inside the cable — not the cable as a whole. Higher category = tighter twists, better signal, faster speeds, less interference.

|Category|Speed|Standard|Common Use|
|---|---|---|---|
|Cat 3|10 Mbps|10BASE-T|Legacy (obsolete)|
|Cat 5|100 Mbps|100BASE-TX|Fast Ethernet|
|Cat 5e|1 Gbps|1000BASE-T|Most common today|
|Cat 6|1 Gbps|1000BASE-T|More reliable|
|Cat 6a|10 Gbps|10GBASE-T|High performance|
|Cat 8|40 Gbps|40GBASE-T|Data center|

---

## Ethernet Standard Naming Convention

Example: **1000BASE-T**

```
1000 = speed in Mbps (1000 Mbps = 1 Gbps)
BASE = baseband — uses full cable bandwidth
T    = twisted pair copper
       (F or X in this position = fiber)
```

---

## Evolution of Ethernet

```
Standard      Year  Pairs  Speed     Notes
─────────────────────────────────────────────────────
10BASE-T      1990  2      10 Mbps   Cat 3 wire
100BASE-TX    1995  2      100 Mbps  Cat 5, Fast Ethernet
1000BASE-T    1999  4      1 Gbps    Cat 5e, all pairs active
10GBASE-T     2006  4      10 Gbps   Cat 6a
40GBASE-T     2016  4      40 Gbps   Cat 8
```

Key evolution: 10/100 Ethernet used 2 pairs (4 wires). Gigabit uses all 4 pairs (8 wires) simultaneously, with each pair bidirectional — a massive speed increase.

---

## The RJ45 Connector

The 8-pin connector crimped onto the end of the cable. Clip facing down, pins facing you:

```
┌──────────────────────────┐
│ 1  2  3  4  5  6  7  8  │
└──────────────────────────┘
```

---

## T568B Pinout — Memorize This

The standard pinout for straight-through cables. Most common in North America.

```
Pin 1 — White Orange   TX+
Pin 2 — Orange         TX-
Pin 3 — White Green    RX+
Pin 4 — Blue
Pin 5 — White Blue
Pin 6 — Green          RX-  ← NOT white green
Pin 7 — White Brown
Pin 8 — Brown
```

Memory sequence: **White-Orange, Orange, White-Green, Blue, White-Blue, Green, White-Brown, Brown**

---

## T568A Pinout

Same as T568B but orange and green pairs swapped. Used in some government installations and for making crossover cables.

```
Pin 1 — White Green
Pin 2 — Green
Pin 3 — White Orange
Pin 4 — Blue
Pin 5 — White Blue
Pin 6 — Orange
Pin 7 — White Brown
Pin 8 — Brown
```

---

## Straight-Through vs Crossover Cables

### Why Pin Layout Matters

Devices are hardwired to transmit (TX) and receive (RX) on specific pins:

```
End devices (PC, server):    Switch:
TX on pins 1, 2              RX on pins 1, 2
RX on pins 3, 6              TX on pins 3, 6
```

Switches are designed to complement end devices. That's why straight-through works for PC → Switch.

### Straight-Through Cable (both ends T568B)

```
PC ─────────────────── Switch
Pin 1 TX ──── Pin 1 RX  ✅
Pin 2 TX ──── Pin 2 RX  ✅
Pin 3 RX ──── Pin 3 TX  ✅
Pin 6 RX ──── Pin 6 TX  ✅
```

Use for: PC → Switch, Server → Switch, PC → Router

### The Problem — Same Device to Same Device

```
PC ─────────────────── PC  (straight-through)
Pin 1 TX ──── Pin 1 TX  ❌ both transmitting
Pin 2 TX ──── Pin 2 TX  ❌ both transmitting
Pin 3 RX ──── Pin 3 RX  ❌ both listening, silence
Pin 6 RX ──── Pin 6 RX  ❌ both listening, silence
```

Nobody hears anything. Collision. No communication.

### Crossover Cable (one end T568B, other end T568A)

Swaps pins 1↔3 and 2↔6 so TX connects to RX:

```
PC ─────────────────── PC  (crossover)
Pin 1 TX ──── Pin 3 RX  ✅ talking to ears
Pin 2 TX ──── Pin 6 RX  ✅ talking to ears
Pin 3 RX ──── Pin 1 TX  ✅ listening to mouth
Pin 6 RX ──── Pin 2 TX  ✅ listening to mouth
```

Legacy crossover requirements (10/100 Mbps era):

- PC ↔ PC
- Switch ↔ Switch
- Router ↔ Router

---

## Auto-MDIX — Why Crossovers Are Mostly Obsolete

**Auto-MDIX** (Automatic Medium-Dependent Interface Crossover) is built into modern gigabit NICs and switches.

```
How it works:
1. Device detects what pins the other device is TX on
2. Automatically swaps its own TX/RX pins to match
3. Straight-through cable works for ANY connection type
```

Auto-MDIX is mandatory in the 1000BASE-T standard. In practice, crossover cables are rarely needed today.

**Exam trap:** If the exam asks what cable connects switch-to-switch — answer **crossover**. The exam tests theory, not modern Auto-MDIX behavior.

---

## Cable Length Limit

Maximum copper Ethernet run: **100 meters**

Beyond 100 meters:

- Electrical signal degrades
- Late collisions occur (a diagnostic sign of excessive cable length)
- Network becomes unreliable or fails entirely

Solution beyond 100 meters: use **fiber optic cable** — no equivalent copper distance limitation.

---

## Making the Cable — Step by Step

```
1. Strip ~1 inch of outer jacket
   Score carefully — never nick the copper wires inside

2. Untwist all 4 pairs completely

3. Arrange wires in T568B order, straighten flat

4. Trim evenly to ~0.5 inch

5. Insert into RJ45 connector with clip facing DOWN

6. Verify color order through the transparent connector head

7. Crimp firmly — ensure jacket sheath enters the connector
   body for strain relief

8. Repeat identical process on the other end
   (same T568B order for straight-through)

9. Test with a cable tester
   Pins 1–8 light sequentially and match both ends = pass
   Any mismatch or missing light = re-terminate and try again
```

---

## Common Mistakes

- Nicking copper wires while stripping the jacket
- Pin 6 = **Green**, not White Green — the most common termination error
- Wires not fully seated in the RJ45 before crimping
- Jacket sheath not inside the connector body (loses strain relief)
- Wires out of T568B order — cable fails or works intermittently

---

## Key Terms

|Term|Meaning|
|---|---|
|UTP|Unshielded Twisted Pair|
|STP|Shielded Twisted Pair|
|EMI|Electromagnetic Interference|
|Crosstalk|Signal bleed between adjacent wire pairs|
|RJ45|8-pin Ethernet connector|
|T568B|Standard North American pinout|
|T568A|Alternate pinout — used in crossover cables|
|TX|Transmit|
|RX|Receive|
|Auto-MDIX|Automatic pin-swap feature in modern NICs and switches|
|Cat 5e|Cable category supporting 1 Gbps (1000BASE-T)|
|Straight-through|Same pinout both ends — connects different device types|
|Crossover|Different pinouts each end — connects same device types|
|Late collision|Network error caused by cable exceeding 100 meters|

---

## Exam Traps

1. **Pin 6 is GREEN — not White Green.** White Green is pin 3. Swapping these kills your cable.
2. **Crossover = one end T568B, other end T568A.** Straight-through = T568B on both ends.
3. **Switch-to-switch requires a crossover cable** — even though Auto-MDIX handles it in practice. The exam tests the theory.
4. **100 meter limit applies to copper only.** Fiber has no equivalent copper distance limitation.
5. **Cat rating refers to the individual wire quality** inside the cable — not the cable jacket or assembly.
6. **10/100 Ethernet uses 2 pairs.** Gigabit uses all 4 pairs simultaneously, each pair bidirectional.

---

## Recall Questions

1. Name the two enemies that twisted pairs protect against. How does twisting fight each one?
2. What is the T568B pinout? Write all 8 colors in pin order from memory.
3. Why does a straight-through cable fail when connecting two PCs? Explain at the pin level.
4. What is Auto-MDIX? Why does it make crossover cables mostly obsolete?
5. What happens if you exceed 100 meters with a copper Ethernet cable?
6. What does 1000BASE-T mean? Break down each part of the name.
7. What is the difference between UTP and STP? When would you use each?
8. You connect two enterprise switches with a straight-through cable and it works fine. Why?