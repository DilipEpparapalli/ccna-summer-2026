## Simple Explanation
Before a port starts forwarding traffic, classic STP walks it through four sequential states — Blocking, Listening, Learning, then Forwarding — taking up to 50 seconds in a failover. That slowness is deliberate, not a bug: each stage depends on the previous one finishing first. On Cisco switches, this entire process runs *separately for every VLAN* (called PVST), which means the same physical cable can be actively forwarding one VLAN's traffic while blocked for another VLAN's traffic — two independent elections, one wire. And instead of leaving the "who's in charge" election to chance (default MAC-address luck), engineers explicitly configure root bridge priority using commands like `root primary` and `root secondary`.

## Why It Exists
STP can't just flip a port straight to "forwarding" the instant it comes up — it doesn't yet know the topology around it: who the Root Bridge is, whether this port will end up as Root/Designated (forwarding) or Blocked, and what devices are even reachable through it. Skipping ahead risks building a MAC address table off information that turns out to be wrong once the topology settles, or briefly creating a loop before decisions have propagated. The sequential states exist to guarantee the network has fully agreed on the topology *before* any port is trusted to move real traffic.

Per-VLAN STP (PVST) exists because a single spanning tree calculation for an entire network wastes bandwidth — a link blocked for all traffic is a link doing nothing. Running an independent calculation per VLAN means a link blocked for one VLAN can still be actively forwarding a different VLAN's traffic, using redundant links you already have instead of leaving them idle.

Manual root bridge configuration exists because the *default* election (lowest MAC address wins) has zero regard for which switch is actually best positioned, fastest, or most reliable — it could easily hand control of your network to some old switch in a wiring closet.

## How It Works

### The Four Port States (Classic STP)
| State | Duration | What's Happening |
|-------|----------|-------------------|
| **Blocking** | Up to 20 sec | Port is up but not forwarding; STP suspects it might create a loop. Only receiving BPDUs. |
| **Listening** | 15 sec | Actively collecting BPDUs to determine: who's the Root Bridge, and what role will this port play (forwarding or blocked)? Still not forwarding user traffic. |
| **Learning** | 15 sec | Topology/port roles are now settled from Listening; the switch begins building its MAC address table based on traffic it observes. Still not forwarding user traffic yet. |
| **Forwarding** | — | Port actively forwards traffic. |

- **Listening and Learning must happen in that order, not in parallel.** During Listening, the switch is still determining whether this port will even end up forwarding — its role in the topology isn't decided yet. If MAC-learning started before that's settled, the switch could build its address table based on a port that's about to be blocked, producing incorrect forwarding data. Learning can only be trusted once Listening has finished and the topology is locked in.
- Listening + Learning alone = **30 seconds minimum** before a freshly-activated port does anything useful.
- Including a Blocking period during failover, full recovery can take **up to 50 seconds** — considered a real operational problem on modern networks, which is why Rapid STP (RSTP) exists (covered next lesson).

### PVST — Per-VLAN Spanning Tree
- Running `show spanning-tree` on a Cisco switch shows **one STP instance per VLAN**, not one for the whole switch.
- Each VLAN runs its own **fully independent** STP calculation — its own Root Bridge election, its own BPDUs, its own port role decisions. These calculations don't share information; VLAN 10's election has no effect on VLAN 20's election.
- Because each VLAN's election is independent, **VLAN 10 can have a different Root Bridge than VLAN 20** on the very same set of switches.
- Practical result: on a single physical trunk link between two switches, that link can be **forwarding for VLAN 10 and simultaneously blocked for VLAN 20** — not because the wire has a split state, but because two entirely separate decisions (one per VLAN) both happen to use that same physical wire to carry their own VLAN's traffic.
- This is how PVST achieves load balancing: rather than adding more physical links, existing redundant links get used more efficiently — active for some VLANs, standing by for others.
- Cisco encodes the VLAN into the priority value: default priority 32,768 + VLAN ID. VLAN 1 → 32,769. VLAN 10 → 32,778. (This is why default priority values look "off" instead of a clean 32,768 across the board.)

### Configuring the Root Bridge
Leaving root bridge election to default MAC-address luck is not acceptable in production — it hands control of the network to whichever switch happens to have the oldest MAC address, regardless of whether that switch is actually well-positioned or reliable.

**Option 1 — Manual priority:**
```
spanning-tree vlan 1 priority 4096
```
- Lower number wins the election.
- Cisco requires priority values in **increments of 4,096**, because part of the priority field's bits are reserved to encode the VLAN ID for PVST.

**Option 2 — Root primary / root secondary (recommended):**
```
spanning-tree vlan 1 root primary
spanning-tree vlan 1 root secondary
```
- `root primary` **checks the network first** to see what priorities already exist, then sets this switch's priority a couple of increments *below* the current lowest — guaranteeing it wins, without needing to know or hardcode what other priorities are already set. Hardcoding a fixed value (e.g., always using 4096) risks losing the election if some other switch was already manually set lower.
- `root secondary` does the same check, but sets itself just *above* primary (still below everyone else). If the primary root bridge fails, the secondary automatically takes over — instead of the election falling back to whatever random switch wins by default MAC address.
- Multiple VLANs can be configured in one command:
```
spanning-tree vlan 1 10 20 root primary
```

## Key Terms
| Term | Meaning |
|------|---------|
| Blocking | Port state where STP suspects a possible loop; not forwarding, only receiving BPDUs |
| Listening | Port state where the switch gathers BPDUs to determine Root Bridge and its own port role |
| Learning | Port state where the switch builds its MAC address table, now that topology/port roles are settled |
| Forwarding | Port state where user traffic actively moves through the port |
| PVST (Per-VLAN Spanning Tree) | Cisco's implementation running one independent STP instance per VLAN, enabling different root bridges and load balancing across VLANs |
| root primary | Command that dynamically sets a switch's priority below the current lowest detected on the network, guaranteeing it wins root bridge election |
| root secondary | Command that sets a switch's priority just above the primary root, so it automatically takes over if the primary fails |

## Real-World Connection
**Pi 5 home lab:** if you ran multiple VLANs across your managed switch with redundant trunk links, PVST-style behavior means one VLAN's traffic could be actively using a link that's sitting blocked for a different VLAN — effectively splitting load across links you already own rather than leaving one idle as pure backup.

**Castle Rysen scenario / NetworkChuck Coffee setup:** with three VLANs across the outpost's switches, PVST lets each VLAN elect its own most-efficient root bridge and spread traffic across the available redundant trunks — rather than one single spanning tree forcing all VLAN traffic through the same narrow active path while everything else sits idle.

**Enterprise campus network:** the real-world tip is blunt about this — production networks always explicitly configure root primary/secondary on their most capable, well-connected switches. Leaving it to chance risks a random closet switch becoming the backbone of the entire network, with no easy warning until performance degrades.

## Exam Traps
- **Listening and Learning are each 15 seconds; total minimum delay before forwarding is 30 seconds** — but full failover recovery (including Blocking) can be up to 50 seconds. Know both numbers and which scenario each applies to.
- **PVST priority values are VLAN-offset from 32,768**, not a flat default — VLAN 1 shows 32,769, VLAN 10 shows 32,778, etc. Don't be thrown by "odd" default priority numbers in `show` output.
- **Manual priority changes must be in increments of 4,096** — this isn't arbitrary; it's because Cisco borrows bits from the priority field to encode the VLAN ID for PVST.
- **`root primary` doesn't hardcode a fixed priority value** — it dynamically checks the network and sets itself relative to whatever the current lowest priority is. Don't assume it always sets a specific fixed number.
- **Each VLAN's STP calculation is fully independent** — a link can simultaneously be forwarding for one VLAN and blocked for another on the exact same physical cable. This is a favorite scenario-question setup.

## Commands (if applicable)
```
spanning-tree vlan 1 priority 4096          ! Manually set priority (increments of 4096 only)
spanning-tree vlan 1 root primary           ! Dynamically become root bridge for VLAN 1
spanning-tree vlan 1 root secondary         ! Dynamically become backup root bridge for VLAN 1
spanning-tree vlan 1 10 20 root primary     ! Apply root primary across multiple VLANs at once
show spanning-tree                          ! View STP state (one instance per VLAN on Cisco switches)
```

## Recall Questions
1. Why must a port pass through Listening before Learning, rather than doing both simultaneously?
2. On a single physical trunk cable between two switches, how can that link be forwarding traffic for one VLAN while blocked for another VLAN at the same time?
3. Why does `root primary` dynamically check existing priorities on the network instead of just setting a fixed hardcoded value like 4096 every time?
4. Why does Cisco require manual priority changes to be in increments of 4,096 instead of allowing any arbitrary number?
