## Simple Explanation
Rapid STP (RSTP) uses the exact same algorithm as classic STP — same root bridge election, same port roles, same blocking logic — but it fixes the slowness. Blocked backup ports aren't left "dumb" anymore; they're continuously kept informed in the background, so when the primary link fails, they don't need to relearn the topology from scratch. Combined with much shorter failure-detection timers, failover drops from up to 50 seconds down to often under a second. Multiple Spanning Tree (MST) takes this further for large environments by letting many VLANs share a single spanning tree instance instead of each VLAN running its own.

## Why It Exists
Classic STP's 50-second failover was acceptable when networks were a convenience, not critical infrastructure. Modern networks can't tolerate that — a business losing connectivity for even a few seconds is a real incident. RSTP exists to close that gap using the same trusted algorithm, just executed faster and smarter. MST exists for a separate but related problem: PVST+/Rapid PVST+ runs a fully independent spanning tree instance per VLAN, which is powerful for load balancing at a small scale (a handful of VLANs) but becomes a real CPU and BPDU-traffic burden once an environment has dozens or hundreds of VLANs, each independently electing roots and exchanging heartbeats.

## How It Works

### What RSTP Changes
RSTP keeps the same root bridge election, same three-step port role logic, and same core purpose as classic STP. It improves exactly two things:

**1. Much shorter timers**
- Classic STP's max age timer: **20 seconds** before it acts on a suspected failure.
- BPDUs are sent every 2 seconds in both classic STP and RSTP.
- Missed-BPDU tolerance = max age timer ÷ BPDU interval:
  - Classic STP: 20 sec ÷ 2 sec = **10 missed BPDUs** tolerated before reacting.
  - RSTP: 6 sec ÷ 2 sec = **3 missed BPDUs** tolerated before reacting.
- In the best case — a cable is physically unplugged and the port state drops to "down" — RSTP doesn't even need to wait on missed BPDUs at all. It reacts immediately, enabling failover in under a second.

**2. Blocked ports become informed, not dumb — the Alternate Port**
- In classic STP, a blocked port is just "blocked." It has no pre-existing knowledge of whether it's actually safe to use, or what happens if it becomes active — that information has to be *discovered from scratch* after a failure, which is exactly why Listening (15 sec) and Learning (15 sec) exist and take so long.
- In RSTP, that same port is labeled an **Alternate Port**. Critically, it isn't idle while blocked — it continues exchanging BPDUs with its neighbor switch in the background the entire time it's blocked, continuously confirming it has a safe, loop-free path to the same root bridge.
- Because that discovery work already happened *before* the failure (not after), the switch doesn't need to rediscover the topology when the primary link dies — it already knows the Alternate Port is safe. It can transition straight to forwarding, skipping the need to relearn from zero.
- **This is the real mechanism behind the speed gain**: RSTP doesn't just do the same discovery process faster — it moves the discovery earlier, so none of it needs to happen reactively after a failure at all.

### Turning On RSTP (Cisco)
```
Switch(config)# spanning-tree mode rapid-pvst
```
- One command, applied on every switch in the environment.
- On Cisco gear this specifically enables **Rapid PVST+** — Rapid STP combined with Cisco's existing per-VLAN spanning tree model (PVST). You still get an independent instance and independent root bridge per VLAN, now with RSTP's speed improvements applied to each one.

### Multiple Spanning Tree (MST) — Solving Scale
- Rapid PVST+ still runs **one full independent STP instance per VLAN.** At small scale (a handful of VLANs) that's a strength — enabling per-VLAN load balancing. At large scale (50, 100, 200+ VLANs), it becomes a real cost: every VLAN independently generates its own BPDUs, runs its own root bridge election, and consumes its own CPU cycles for topology maintenance — multiplied by VLAN count.
- MST groups multiple VLANs into a shared spanning tree **instance**. Example: VLANs 10–50 share one instance, VLANs 51–100 share a separate instance.
- **What's gained:** dramatically reduced CPU load and BPDU overhead — you're no longer running as many separate, simultaneous topology calculations.
- **What's given up:** fine-grained, per-individual-VLAN independence. VLANs grouped into the same MST instance are forced to share one topology, one root bridge, and one set of blocked/forwarding decisions — they can no longer be tuned or load-balanced completely independently of each other the way they could under full PVST+. You still retain flexibility at the *instance* (group) level, just not at the individual VLAN level.

## Key Terms
| Term | Meaning |
|------|---------|
| RSTP (Rapid Spanning Tree Protocol) | Faster version of classic STP; same algorithm, shorter timers, and pre-informed backup ports |
| Alternate Port | RSTP's replacement for a classic Blocked Port; continuously exchanges BPDUs while blocked so it already knows it's safe to activate instantly on failure |
| Max Age Timer | How long a switch waits before acting on suspected BPDU loss; 20 sec in classic STP, 6 sec in RSTP |
| Rapid PVST+ | Cisco's implementation combining RSTP's speed with PVST's per-VLAN independent instances |
| MST (Multiple Spanning Tree) | Groups multiple VLANs into shared spanning tree instances, reducing per-VLAN overhead at scale |
| Instance (MST) | A single shared spanning tree calculation covering a defined group of VLANs |

## Real-World Connection
**Pi 5 home lab:** with only a handful of VLANs, Rapid PVST+ is more than sufficient — you get near-instant failover and still keep full per-VLAN flexibility without any real overhead concern.

**Castle Rysen / NetworkChuck Coffee scenario:** with three VLANs across the outpost, per-VLAN independence under Rapid PVST+ is a genuine asset, not a burden — small enough scale that the CPU/BPDU cost of one instance per VLAN is negligible.

**Enterprise campus network:** this is exactly where MST earns its place — a large campus with 100+ VLANs would grind under one full independent STP instance per VLAN. Grouping related VLANs into a handful of MST instances keeps convergence fast and CPU load manageable, trading away only the finest-grained per-VLAN tuning most environments don't actually need at that scale.

## Exam Traps
- **Know the exact timer math**: classic STP max age = 20 sec (10 missed BPDUs at 2-sec intervals); RSTP max age = 6 sec (3 missed BPDUs). Don't just memorize "RSTP is faster" — be ready to do the division.
- **RSTP uses the same algorithm as classic STP** — root bridge election, cost/Bridge ID/port-number tiebreakers, and port roles are unchanged. The exam may test whether you know RSTP is a speed improvement, not a different algorithm.
- **Alternate Port ≠ just a renamed Blocked Port.** The distinguishing fact is that it stays actively informed via ongoing BPDU exchange while blocked — that's what enables the near-instant failover, not just a shorter timer alone.
- **Rapid PVST+ is Cisco's default modern recommendation**, not plain RSTP or classic STP. The command is `spanning-tree mode rapid-pvst`.
- **MST doesn't eliminate per-VLAN flexibility, it reduces its granularity.** VLANs grouped into the same instance share one topology; VLANs in different instances can still differ. Don't treat MST as "all VLANs now share one identical topology, period."

## Commands (if applicable)
```
spanning-tree mode rapid-pvst      ! Enable Rapid PVST+ on a Cisco switch (do this on every switch)
show spanning-tree                 ! View current STP mode and per-VLAN instance state
```

## Recall Questions
1. Do the math: how many missed BPDUs does classic STP tolerate before reacting, versus RSTP? What timer and interval values produce those numbers?
2. What does an RSTP Alternate Port actually know, continuously, that a classic STP Blocked Port does not — and why is that the real reason failover can happen almost instantly?
3. What specific cost grows as VLAN count increases under Rapid PVST+, and what does MST trade away in order to reduce that cost?
