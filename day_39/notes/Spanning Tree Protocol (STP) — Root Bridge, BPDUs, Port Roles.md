## Simple Explanation
STP solves the loop problem using a formula switches can run automatically: first, they elect one switch as the "center of the universe" (the Root Bridge). Then every other switch figures out its best path to that center. Finally, on any redundant link, one side keeps forwarding traffic and the other side blocks — like leaving one door open and one door shut in a two-door loop. The loop breaks, but the backup path is still physically there, ready to open if needed.

## Why It Exists
Humans can glance at a network diagram and instantly spot which link to block. Switches can't — they need a repeatable, automatic formula they can all run independently and agree on without a human intervening. That formula requires: (1) agreeing on a single reference point all paths are measured from (the Root Bridge), (2) a way to continuously communicate and detect failures (BPDUs), and (3) a consistent rule for deciding which ports forward and which ports block (port roles, driven by path cost).

## How It Works

### 1. Root Bridge Election
- Every switch has a **Bridge ID** = Priority + MAC address.
- Default priority on all switches: **32,768** (tied by default).
- Since priorities tie out of the box, the **MAC address breaks the tie** — lowest MAC wins.
- Older switches have lower MAC addresses, so **by default, the oldest switch in the network becomes the Root Bridge.**
- This is intentional, not accidental: if the newest switch always won, plugging in any new switch anywhere in the network would trigger a full topology recalculation every time — causing unnecessary outages and churn. An older, stable default root keeps the network calm unless something actually changes.
- In practice, engineers **manually lower the priority** on the switch they want as root (typically the core switch) rather than leaving it to chance.

### 2. BPDUs (Bridge Protocol Data Units)
- Switches exchange BPDUs as a continuous heartbeat — **every 2 seconds**, on every link.
- BPDUs are how the Root Bridge election happens in the first place.
- BPDUs are also the ongoing failure-detection mechanism: if a switch stops receiving BPDUs on a link, it assumes that link is dead and begins the process of unblocking a backup path to restore connectivity.

### 3. Port Cost and Port Roles
- Once the Root Bridge is elected, every other switch calculates its **best path to the root** using **port cost** — a number based on link speed.
- Faster link = lower cost = more attractive path. (Exam values: **100 Mbps = cost 19**, **1 Gbps = cost 4**.)
- Every port then gets one of three role labels:

| Port Role | Definition |
|-----------|------------|
| **Root Port** | The single port on a non-root switch that offers the best (lowest-cost) path to the Root Bridge. Every non-root switch has exactly one. |
| **Designated Port** | A port actively forwarding traffic. The Root Bridge's ports are *always* all designated. |
| **Blocked Port** | The redundant path that does not forward data frames, held in reserve to prevent a loop. |

**The critical trap:** on a redundant link between two switches, only ONE end blocks — not both. The same physical cable can show as **Designated** (forwarding) on one switch's `show spanning-tree` output, and **Blocked** on the other switch's output, simultaneously. Both are correct at the same time. Blocking is a per-port decision each switch makes about its own end, not a property of the cable itself.

```
        Port1 (Designated) ──────────── Port1 (Root Port)   ← forwarding, both ends
SW-A (Root Bridge)                                    SW-B
        Port2 (Designated) ──────────── Port2 (Blocked)     ← SAME cable, different labels!
```

A "blocked" port is not electrically disconnected — the switch still listens for BPDUs on it (so it knows the instant the primary path dies), it just refuses to forward data frames through it. Only one side needs to hold the door shut to break a loop; blocking both sides would be redundant and pointless, since one blocked side is already sufficient to stop the loop from forming.

## Key Terms
| Term | Meaning |
|------|---------|
| Root Bridge | The single switch elected as the reference point that all other switches calculate their best path toward |
| Bridge ID | Priority + MAC address; used to elect the Root Bridge — lowest Bridge ID wins |
| Priority | Configurable value (default 32,768) that is the primary tiebreaker in root bridge election; lowering it manually forces a switch to win |
| BPDU (Bridge Protocol Data Unit) | Heartbeat message sent every 2 seconds between switches; drives root bridge election and link failure detection |
| Port Cost | A value assigned per port based on link speed, used to calculate the best path to the Root Bridge; lower cost wins (100 Mbps = 19, 1 Gbps = 4) |
| Root Port | The one port per non-root switch with the lowest-cost path to the Root Bridge |
| Designated Port | A forwarding port; all Root Bridge ports are designated by default |
| Blocked Port | A port that does not forward data frames, held in reserve on a redundant link to prevent a loop |

## Real-World Connection
**Pi 5 home lab:** if you were running enterprise-grade managed switches with STP enabled, your oldest switch would silently become the root bridge by default — possibly not the one you'd actually want handling that role if it's your slowest or least central device. On a real network you'd manually adjust priority on whichever switch is your most central, most reliable one.

**Castle Rysen scenario:** imagine Cafe-SW1, Fallout-SW1, and a newer switch added later, all interconnected. Without manual priority tuning, the oldest switch — maybe a beat-up relic from an earlier era of the outpost — becomes the root bridge by default, potentially routing all traffic through the least reliable point in the network. Manually setting priority on the actual core switch avoids that trap.

**Enterprise scale:** this is exactly why real production networks insist on manually setting priority on the core switch — leaving root bridge election to chance in a building with dozens of switches means you have no control over which one becomes the hub, and you won't find out it's a problem until something breaks at 2am.

## Exam Traps
- **The oldest switch becomes root bridge by default** — because it has the lowest MAC address, and priorities tie at default (32,768). This is *intentional* for stability, not a flaw, and is a favorite "why" question on the exam.
- **Lowering priority wins the election.** Remember: *lower* number wins, both for priority and for the overall Bridge ID. Don't second-guess this into "higher wins" under exam pressure.
- **A single redundant link can show different port states on each end** — Designated on the root bridge side, Blocked on the other. This is correct and expected, not an inconsistency. Reading `show spanning-tree` output on only one switch will not tell you the full picture of the link.
- **Blocked does not mean disconnected.** The port is still physically up and still listening for BPDUs — it just isn't forwarding data frames. This distinction matters for understanding failover speed.

## Commands (if applicable)
_Not yet covered — this lesson remains conceptual. Commands for viewing STP state (`show spanning-tree`) and manually setting priority to control root bridge election will be added once covered in upcoming lessons._

## Recall Questions
1. Why does the default STP configuration cause the *oldest* switch in a network to become the root bridge, and why is that actually a deliberate design choice rather than an oversight?
2. On a redundant link between two switches, can the same physical cable be labeled "Designated" on one end and "Blocked" on the other at the same time? Why or why not?
3. What is a BPDU actually doing on an ongoing basis (not just during the initial election), and what triggers a blocked port to become active?
4. Between a 100 Mbps link and a 1 Gbps link, which has the lower port cost, and why does lower cost win when a switch calculates its best path to the root bridge?
