## Simple Explanation
Rolling out Spanning Tree across a real network isn't just "turn it on" — it's a five-step sequence, in order, because each step depends on the one before it being done correctly. Skip a step, do it out of order, or miss one switch, and you get a network that looks fine on the surface while quietly carrying a slow-convergence weak spot or an invisible loop on a specific VLAN.

## Why It Exists
STP's whole job is to prevent loops while keeping convergence fast — but that only works if every switch agrees on the same STP version, the same root bridge, and the same VLAN membership across every trunk. A checklist exists because these dependencies aren't obvious from a single switch's perspective: a switch can look perfectly configured on its own and still be the one weak link that drags the whole topology's failover time down, or the one missing VLAN that turns a redundant path into an active, undetected loop.

## How It Works — The Five Steps, In Order

**Step 1 — Enable Rapid PVST+ on every switch, no exceptions**
```
spanning-tree mode rapid-pvst
```
This has to happen first and everywhere, because Rapid PVST+ falls back to classic 802.1D behavior *per link* the moment it detects a neighbor sending classic STP BPDUs. One switch left on legacy STP doesn't break the election math — root/designated/blocked port selection still works — but every link touching that switch reverts to 30–50 second convergence timers while the rest of the network reconverges in under a second. The whole topology's effective failover speed becomes only as fast as its slowest switch.

**Step 2 — Choose and configure the root bridge**
Pick the switch with the most uplinks or the one closest to the router as root — it becomes the "center of the universe" that every other switch calculates its best path toward.

Manual priority, per VLAN (since PVST+ runs one independent STP instance *per VLAN*, priority must be set per instance, not once for the whole switch):
```
spanning-tree vlan 1,10,20,30,40 priority 4096
```
Or let Cisco calculate it:
```
spanning-tree vlan 1,10,20,30,40 root primary
spanning-tree vlan 1,10,20,30,40 root secondary
```
`root primary` sets priority to **24576** (or 4096 below the current lowest priority seen in the topology, if something is already lower). `root secondary` sets priority to **28672** — high enough to lose to the primary while it's up, low enough to beat every other switch if the primary fails. Lower number always wins the election.

**Step 3 — Verify consistent VLAN configuration across trunks**
```
show interfaces trunk
```
Confirm the allowed-VLAN list matches on both ends of every trunk link, on every switch — before moving on. This step exists precisely because a missing VLAN on a trunk is invisible under normal monitoring: STP builds a completely separate topology decision per VLAN, so if VLAN 10 never crosses a given trunk, the switches on either end have no way of knowing a redundant path exists for VLAN 10 at all. STP can't block a loop it doesn't know about. VLAN 1 can look perfectly healthy on the same link while VLAN 10 is actively looping.

**Step 4 — Enable PortFast and BPDU Guard on access ports**
```
interface range fa0/3-24
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable
```
PortFast skips the listening/learning delay for ports connected to a single end device. BPDU Guard is the backstop — if a PortFast port ever receives a BPDU, that's proof a switch (not an end device) got connected, and the port is immediately error-disabled before a loop can form. Deliberately done *after* Step 3 — no point speeding up access ports on a topology whose trunk VLAN membership hasn't been verified yet.

**Step 5 — Verify and save**
```
show spanning-tree
copy run start
```
Confirm the root bridge is who you expect, and that blocked ports make sense given the standard tiebreaker order (lowest path cost → lowest bridge ID → lowest port ID). A port blocked somewhere unexpected is a diagnostic clue, not just a status check — e.g., a Fast Ethernet port auto-negotiated down to 10 Mbps inflates its path cost, which can shift which port gets blocked. Save only after everything checks out, on every switch.

## Key Terms
| Term | Meaning |
|------|---------|
| Rapid PVST+ | Cisco's fast, per-VLAN spanning tree implementation — runs one independent STP instance per VLAN |
| Root primary / root secondary | Keywords that let the switch auto-calculate priority (24576 / 28672 by default) instead of setting a number manually |
| Priority (STP) | Part of the Bridge ID used in root election; lower number wins; must be set per-VLAN under PVST+ |
| Trunk VLAN consistency | The requirement that every trunk link carry the same allowed-VLAN set on both ends, so STP has visibility into every VLAN's topology |

## Real-World Connection
In a multi-closet enterprise campus, it's common for a new switch to get added to a trunk with a copy-pasted `switchport trunk allowed vlan` line that's missing a VLAN the rest of the network uses — the network looks fine until that specific VLAN develops a loop that standard "is STP okay?" monitoring never catches, because that monitoring is usually checking VLAN 1 or the management VLAN, not every VLAN individually. This is exactly why Step 3 exists as a deliberate, standalone checklist item rather than something assumed to be correct by default.

## Exam Traps
- Mixing STP versions across switches does **not** break root/designated/blocked port election — it silently degrades convergence speed on the links touching the legacy switch. Don't describe this as "STP breaks" — it's a timing problem, not a correctness problem.
- `spanning-tree vlan X priority Y` must list every VLAN that needs the setting — there is no switch-wide priority command under PVST+, since each VLAN runs its own instance.
- `root primary` sets the **lower** number (24576) and `root secondary` sets the **higher** number (28672) — a common trap is assuming "secondary" should get a more aggressive (lower) number since it sounds like a backup that should "try harder." It's the opposite: secondary must lose to primary while primary is alive.
- A missing VLAN on a trunk produces a loop that is invisible on every other VLAN — `show interfaces trunk` must be checked per-link, not assumed correct because the topology "looks" fine.

## Commands
```
! Step 1 - Rapid PVST+ everywhere
spanning-tree mode rapid-pvst

! Step 2 - Root bridge, manual priority (must list every VLAN)
spanning-tree vlan 1,10,20,30,40 priority 4096

! Step 2 - Root bridge, automatic keywords
spanning-tree vlan 1,10,20,30,40 root primary
spanning-tree vlan 1,10,20,30,40 root secondary

! Step 3 - Verify trunk VLAN consistency
show interfaces trunk

! Step 4 - PortFast + BPDU Guard on access ports
interface range fa0/3-24
 switchport mode access
 spanning-tree portfast
 spanning-tree bpduguard enable

! Step 5 - Verify and save
show spanning-tree
copy run start
```

## Recall Questions
1. Why does enabling Rapid PVST+ on every switch have to come before any other step, and what specifically happens on a link where one side is still running classic STP?
2. Why does `spanning-tree vlan X priority Y` require listing every VLAN number instead of one switch-wide command — and what are the actual default priority values `root primary` and `root secondary` assign?
3. If Step 4 (PortFast/BPDU Guard) were done before Step 3 (trunk VLAN verification) and a trunk was missing a VLAN, what would you see — or fail to see — on the network, and on which VLANs specifically?
