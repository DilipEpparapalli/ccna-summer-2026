## Simple Explanation
Once a Root Bridge is elected, every switch runs the exact same three-step process to decide, port by port, whether it forwards traffic or blocks it. Step 1 guarantees every switch has one working path to the center of the network. Step 2 makes sure every cable segment has exactly one side actively forwarding. Step 3 blocks everything left over. Because Step 1 happens first, blocking in Step 3 can never accidentally cut a switch off — it can only remove genuinely extra, redundant paths.

## Why It Exists
Knowing STP "blocks redundant links" isn't enough — real networks (especially the three-tier Cisco hierarchy: access, distribution, core) are redundant everywhere, with multiple paths from nearly every switch to the top. Someone — or something — has to systematically decide *which* of those many paths stays active and which get blocked, consistently, the same way, on every switch, every time. The three-step algorithm is that consistent formula. It's also what guarantees the result is always safe: a fully connected, loop-free tree, never a network with an isolated switch.

## How It Works

### Step 1: Identify Root Ports
- Every non-root switch finds its single best path to the Root Bridge.
- That port becomes the switch's **Root Port** — always forwarding.
- Every non-root switch gets exactly one Root Port.
- Tiebreakers, applied **in this order** if the previous one ties:
  1. **Lowest cost to the Root Bridge** (based on cumulative link speed cost along the path: 100 Mbps = 19, 1 Gbps = 4)
  2. **Lowest Bridge ID** of the neighboring switch
  3. **Lowest port number** on the neighboring switch

### Step 2: Activate Designated Ports
- Every physical link (segment) in the network needs exactly one forwarding port on it.
- The **Root Bridge automatically wins Designated Port on every one of its own segments** — this is the "reward" for winning the root election. All Root Bridge ports are Designated Ports.
- For every other segment (a link between two non-root switches, or a switch and a non-root neighbor), the two switches compete for the Designated Port using the **same three tiebreakers as Step 1**: lowest cost to root, then lowest Bridge ID, then lowest port number.
- Whichever switch wins becomes the Designated Port for that segment (forwarding). The losing side's port moves on to Step 3.

### Step 3: Block All The Leftovers
- Any port that is **neither a Root Port nor a Designated Port** becomes a **Blocked Port**.
- Blocked ports are still electrically active and still receive BPDUs — they simply don't forward user traffic.
- In a fully redundant design, this can mean blocking roughly half of all links network-wide. That's expected, not a failure of the design.

### Why Blocking Is Always Safe
This is the core guarantee, and it comes purely from the **order of operations**:
- Step 1 runs first and guarantees **every single non-root switch already has one working path to the Root Bridge** before any blocking decision is ever made.
- Because that guarantee is locked in first, a port only becomes *eligible* to be blocked in Step 3 if it was never someone's only path — it can only be an extra, duplicate path layered on top of a connection that already exists.
- This is why mentally erasing every blocked link from a messy, fully-redundant topology always leaves behind a network that looks like a tree: full connectivity to every switch, zero loops — never an isolated switch. The safety isn't a coincidence; it's a direct consequence of guaranteeing reachability (Step 1) before ever removing anything (Step 3).

```
                    [ROOT BRIDGE]
                    All ports: Designated
                   /              \
          Designated            Designated
                 /                  \
          Root Port              Root Port
           SWITCH-A ─────────────── SWITCH-B
              (Designated)   (Blocked)
                 extra redundant link between A and B
```

## Key Terms
| Term | Meaning |
|------|---------|
| Root Port | The one port per non-root switch offering the best path to the Root Bridge; always forwarding |
| Designated Port | The one forwarding port on each network segment; Root Bridge ports are always designated |
| Blocked Port | A port that is neither a Root Port nor a Designated Port; electrically active but not forwarding traffic |
| Segment | A single physical link between two switches; every segment requires exactly one Designated Port |
| Cost | A value based on link speed used as the primary tiebreaker (100 Mbps = 19, 1 Gbps = 4); lower cost wins |
| Tiebreaker order | Cost → Bridge ID → Port number, applied identically in both Step 1 and Step 2 |

## Real-World Connection
**Three-tier hierarchy (access/distribution/core):** this is exactly the topology the algorithm is built for — many redundant paths from access switches up through distribution to the core. The three-step process is what keeps that redundancy safe instead of catastrophic, while still leaving every blocked link ready to take over instantly if something fails.

**Pi 5 home lab:** if you added a second managed switch and cross-connected it to your existing one with two cables, your switches would run through exactly these three steps automatically — electing a root, assigning root/designated ports, and blocking the extra path — without you touching a single command.

**Castle Rysen scenario:** picture the outpost's access-layer switches (like Cafe-SW1) each with two uplinks toward a distribution switch, which itself has two uplinks toward a core switch (Fallout-RT1 area). Redundant paths everywhere. The three-step algorithm runs independently on every switch and produces one clean, loop-free tree pulling toward whichever switch won the root election — with every blocked link standing by as an instant failover.

## Exam Traps
- **The tiebreaker order is fixed and identical for both Root Port and Designated Port selection:** cost → Bridge ID → port number. Don't mix up the order or assume they're different processes — they use the same three factors.
- **The Root Bridge always wins Designated Port on all of its own segments** — this is a "free win," not something it competes for. A common trick question tests whether you know the Root Bridge doesn't run the same competition as everyone else.
- **A port is only ever blocked if it is neither Root Port nor Designated Port** — never assume a port is blocked just because it's "redundant-looking" on a diagram; you have to actually run all three steps to know for sure.
- **Blocking roughly half the links in a fully redundant network is normal and expected**, not a misconfiguration. Don't second-guess a topology just because a large fraction of ports show blocked.

## Commands (if applicable)
_Not yet covered — this lesson remains conceptual. The next lesson covers configuring bridge priority to deliberately control root bridge election; commands (`show spanning-tree`, priority configuration) will be added once covered._

## Recall Questions
1. Walk through the three tiebreakers, in order, used to select both a Root Port and a Designated Port. Why is the order itself important?
2. Why does the Root Bridge never have to compete for Designated Port on its own segments?
3. What specifically guarantees that blocking roughly half the links in a fully redundant topology can never accidentally isolate a switch? Which step provides that guarantee, and why does the order it runs in matter?
