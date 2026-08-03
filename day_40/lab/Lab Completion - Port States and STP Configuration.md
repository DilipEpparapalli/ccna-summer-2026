## Topology

```
Fallout-SW1 ──Et0/1────────Et0/1── Fallout-SW2
     │                                  │
   Et0/2                              Et0/2
     │                                  │
     └──────Et0/1── Fallout-SW6 ──Et0/2─┘
```

- Fallout-SW1 Ethernet0/1 ↔ Fallout-SW2 Ethernet0/1
- Fallout-SW1 Ethernet0/2 ↔ Fallout-SW6 Ethernet0/1
- Fallout-SW2 Ethernet0/2 ↔ Fallout-SW6 Ethernet0/2
- Trunks pre-configured on all three switches, VLANs 10/20/30/40 allowed
- VLANs: 10 Shelter-Operations, 20 Shelter-Logistics, 30 Shelter-Medical, 40 Shelter-Comms
- Switch bridge MAC identities observed: Fallout-SW1 = aabb.cc00.0100, Fallout-SW2 = aabb.cc00.0200, Fallout-SW6 = aabb.cc00.0300
![[ccna-summer-2026/day_40/lab/Network_Diagram.png]]
## Task-by-Task Walkthrough

### Task 0 — Map the Shelter's Spanning Tree
VLANs 10, 20, 30, and 40 were created with their named identities on Fallout-SW1. `show vlan brief` confirmed all four VLANs active. VTP domain was set to `fallout` on Fallout-SW1.

Notably, Fallout-SW2 and Fallout-SW6 both later showed **VTP Domain Name: fallout, Configuration Revision: 4, and identical MD5 digests** — despite the VLAN-creation commands only being pasted from Fallout-SW1. This indicates VTP (all three switches in Server mode, same domain) automatically propagated the VLAN database, rather than each VLAN needing to be manually re-typed on every switch.

Initial `show spanning-tree` output on Fallout-SW6 confirmed it as the starting root bridge for VLANs 10–40 (priority 28682/28692/28702/28712 = 28672 + VLAN ID), matching the lab's expected starting state. On Fallout-SW2, VLAN 10 showed Ethernet0/2 as Root FWD and Ethernet0/1 as Altn BLK — confirming the path to root ran through SW6.

### Task 1 — Declare Fallout-SW1 the Root
On Fallout-SW1:
```
spanning-tree vlan 10,20,30,40 priority 20480
```
This is a deviation from the lab guide's suggested value (24576) — see Deviations below.

Verification on Fallout-SW1 (`show spanning-tree vlan 10`):
```
Root ID    Priority    20490
           Address     aabb.cc00.0100
           This bridge is the root
```

Verification on Fallout-SW2 (`show spanning-tree vlan 10`):
```
Root ID    Priority    20490
           Address     aabb.cc00.0100
           Cost        100
           Port        2 (Ethernet0/1)
...
Et0/1               Root FWD 100       128.2    P2p
Et0/2               Altn BLK 100       128.3    P2p
```
Confirms Fallout-SW1 is now root for VLAN 10, and Fallout-SW2 correctly re-elected Ethernet0/1 as its Root Port, with Ethernet0/2 (the old path toward SW6) moving to Alternate Blocking.

### Task 2 — Shape the Alternate Path and Observe Reconvergence
On Fallout-SW2, interface Ethernet0/2:
```
spanning-tree vlan 10 cost 500
spanning-tree vlan 20 cost 500
spanning-tree vlan 30 cost 500
spanning-tree vlan 40 cost 500
```
Cost 500 was used instead of the lab guide's suggested 300 — see Deviations below.

Verification (`show spanning-tree interface ethernet 0/2 detail`) confirmed cost 500 applied across all four VLANs, with the port remaining in **alternate blocking** for each.

Ethernet0/2 was then bounced:
```
interface ethernet0/2
 shutdown
 no shutdown
```
Post-reconvergence verification confirmed Ethernet0/2 returned to **alternate blocking** with cost 500 retained on all four VLANs, and "Number of transitions to forwarding state: 0" — meaning the port never needed to pass through a forwarding state at all; RSTP simply reconfirmed it belonged in Alternate Blocking almost immediately. This is a direct, hands-on confirmation of the RSTP behavior covered in the prior lesson: an Alternate Port stays informed via ongoing BPDU exchange, so reconvergence after a bounce is fast, not a 30–50 second classic-STP-style rebuild.

## Verification Outputs

**Fallout-SW1 — final root state, VLAN 10:**
```
Root ID    Priority    20490
           Address     aabb.cc00.0100
           This bridge is the root
Bridge ID  Priority    20490  (priority 20480 sys-id-ext 10)
Et0/1               Desg FWD 100       128.2    P2p
Et0/2               Desg FWD 100       128.3    P2p
```

**Fallout-SW2 — final state, VLAN 10:**
```
Root ID    Priority    20490
           Address     aabb.cc00.0100
           Port        2 (Ethernet0/1)
Et0/1               Root FWD 100       128.2    P2p
Et0/2               Altn BLK 500       128.3    P2p
```

**Fallout-SW6 — baseline state captured during Task 0** (prior to Fallout-SW1's priority change): showed itself as root for VLANs 10–40 with all ports Designated Forwarding. No post-Task-1 verification pass was captured on Fallout-SW6 in this session — worth a quick `show spanning-tree vlan 10` on SW6 next time to directly confirm it re-elected Fallout-SW1 as root and adjusted its own port roles accordingly, rather than relying only on SW1/SW2's view of the topology.

## Final State Summary

- VLANs 10 (Shelter-Operations), 20 (Shelter-Logistics), 30 (Shelter-Medical), and 40 (Shelter-Comms) exist and are active on Fallout-SW1, Fallout-SW2, and Fallout-SW6, propagated via VTP from a single domain (`fallout`).
- Fallout-SW1 is now the Rapid PVST root bridge for VLANs 10, 20, 30, and 40 (priority 20480 + VLAN ID).
- Fallout-SW2 reaches root via Ethernet0/1 (Root Port, forwarding); Ethernet0/2 sits in Alternate Blocking with an artificially raised path cost of 500 on all four VLANs.
- A shutdown/no-shutdown cycle on Fallout-SW2's Ethernet0/2 confirmed Rapid PVST reconvergence: the port returned directly to Alternate Blocking without transitioning through Forwarding, demonstrating RSTP's fast, pre-informed failover behavior in practice.
- Outstanding follow-up: confirm Fallout-SW6's post-Task-1 spanning-tree state directly (not just inferred from SW1/SW2's perspective) to fully close out verification across all three switches.
