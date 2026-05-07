
## Simple Explanation
Networks need to be designed so one failure doesn't take everything down. The way you connect switches matters as much as which switches you buy. Architecture is a business decision as much as a technical one.

## Why It Exists
Daisy-chained switches create single points of failure — one dead switch can kill connectivity for every device behind it. Layered architecture fixes this by isolating failures and distributing responsibility cleanly.

## The Problem — Daisy Chain

Switch A → Switch B → Switch C

If Switch B dies:
- All devices on Switch B go offline
- All devices on Switch C go offline (no path to network)
- Single point of failure cascades downstream

If Switch A dies:
- Entire network is dead

## The Fix — Two-Tier Architecture (Collapsed Core)

[Router/Firewall]
       |
[Distribution Switch]  ← Multilayer (L2 + L3)
   /    |    \
[Acc] [Acc] [Acc]  ← One switch per zone/floor
  |     |     |
Devices Devices Devices

### Access Layer (Tier 1)
- Where end devices connect (PCs, phones, printers,
  servers)
- Simple Layer 2 switching only
- Job: give devices access to the network

### Distribution Layer (Tier 2)
- Multilayer switch — handles both L2 frames and L3 packets
- Job: aggregate access switches + route between VLANs
- Fault isolation — one access switch dies, only its devices are affected
- In two-tier designs, also acts as the core backbone (this is called "collapsed core")

### Why a Multilayer Switch at Distribution?
A Layer 2 switch cannot route between VLANs.

Example — small office with three VLANs:
  VLAN 10 → Employee workstations
  VLAN 20 → IoT / guest devices
  VLAN 30 → Servers

Traffic from VLAN 10 → VLAN 30 needs Layer 3 routing. The distribution switch does this internally, in hardware, without sending traffic up to a router. Fast and clean.

## Three-Tier Architecture (Campus Scale)

[Router]
   |
[Core Layer]          ← Fast. Reliable. ONE JOB.
   |
[Distribution]        ← Smart. Routes VLANs. Policies.
   |
[Access]              ← Devices connect here.

### When Two-Tier Breaks Down
Multiple buildings → distribution switches start meshing with each other → cable nightmare → add a dedicated core layer to tie everything together.

### Core Layer — One Job Only
MOVE TRAFFIC FAST. That's it.
- No inter-VLAN routing
- No ACLs or policies
- No complexity
- Think: highway with no traffic lights

All the smart work stays at distribution. Core just moves packets between buildings at speed.

## Collapsed Core vs Full Three-Tier

|                  | Collapsed Core          | Three-Tier          |
|------------------|-------------------------|---------------------|
| Layers           | 2 (Access + Dist.)      | 3 (+ Core)          |
| Distribution role| Routing + backbone      | Routing only        |
| Best for         | Small / single site     | Campus / multi-site |
| Cost             | Lower                   | Higher              |

## The Business Reality
Redundancy costs money. Better switches, more links, more hardware = more budget.

Key question: **"What can this business afford to lose?"**
- Small office: collapsed core, acceptable risk
- Hospital or bank: full three-tier, dual uplinks, redundant core pairs

Architecture always matches risk tolerance + budget. Never a purely technical decision.

## Exam Traps
1. Core layer does NOT do inter-VLAN routing — that's distribution's job. Don't mix them up.
2. "Collapsed core" = two-tier model where distribution acts as the core backbone. Not a separate design — just a name for the two-tier model.
3. Redundancy ≠ just more cables. If the switch itself dies, extra cables on that switch don't save you. True redundancy means removing dependency on any single device entirely.

## Recall Questions
1. A switch in the middle of a daisy chain fails. Which devices lose connectivity and why?
2. What can a multilayer switch do that a Layer 2 switch cannot? Give a specific VLAN example.
3. What is the ONE job of the core layer? What should it never do?
4. Your company has one office building, 200 employees. Which architecture — and why?
5. What does "collapsed core" mean? Which layer is doing double duty?