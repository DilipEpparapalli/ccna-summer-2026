## Topology

```
Fallout-SW1 ──Et0/1────────Et0/1── Fallout-SW2
     │                                  │
   Et0/2                              Et0/2
     │                                  │
     └──────Et0/1── Fallout-SW6 ──Et0/2─┘
```
![[ccna-summer-2026/day_40/lab/Network_Diagram.png]]
- Fallout-SW1 Ethernet0/1 ↔ Fallout-SW2 Ethernet0/1
- Fallout-SW1 Ethernet0/2 ↔ Fallout-SW6 Ethernet0/1
- Fallout-SW2 Ethernet0/2 ↔ Fallout-SW6 Ethernet0/2
- Switch bridge identities: Fallout-SW1 = aabb.cc00.0100, Fallout-SW2 = aabb.cc00.0200, Fallout-SW6 = aabb.cc00.0300
- Starting condition: Fallout-SW1 pre-configured as root for VLANs 10–40 (priority base 4096 + VLAN ID), running legacy IEEE PVST

## Task-by-Task Walkthrough

### Task 0 — Audit the Legacy Timers
VLANs 10, 20, 30, and 40 were created and named on Fallout-SW1, with VTP domain set to `fallout`. Only Fallout-SW1's transcript shows explicit `vlan` creation commands — Fallout-SW2 and Fallout-SW6 both show the four VLANs already active without local creation commands in their own transcripts, confirming VTP propagated the VLAN database automatically once the domain name matched (or was adopted from the first advertisement, since both started with a blank domain name).

`show spanning-tree` on Fallout-SW2 confirmed the legacy baseline:
```
Root ID    Priority    4106
           Address     aabb.cc00.0100
           Cost        100
           Port        2 (Ethernet0/1)
Et0/1               Root FWD 100       128.2    P2p
Et0/2               Altn BLK 100       128.3    P2p
```
Protocol shown as `ieee` (classic PVST) — matches expected starting state: Fallout-SW1 is root, Fallout-SW2 reaches it via Ethernet0/1 (Root Port, forwarding), with Ethernet0/2 in Alternate Blocking.

### Task 1 — Enable Rapid PVST Across the Shelter
`spanning-tree mode rapid-pvst` was applied on all three switches (each triggered Cisco's standard warning about potential disruption during the mode change). Post-change verification confirmed protocol now reporting `rstp` on all three, with Fallout-SW1 remaining root for VLANs 10–40 — priorities were untouched by the mode change, so the election outcome didn't shift.

**Failover test on Fallout-SW2:**
- `interface ethernet0/1` → `shutdown` — this forced Ethernet0/2 to become the new path to root. Verification showed:
```
Root ID    Priority    4106
           Cost        200
           Port        3 (Ethernet0/2)
Et0/2               Root FWD 100       128.3    P2p
```
  The root path cost jumped to 200, reflecting the rerouted path through Fallout-SW6 (SW2→SW6 cost 100 + SW6→SW1 cost 100 = 200) — direct confirmation that traffic rerouted around the triangle rather than simply failing.
- `interface ethernet0/1` → `no shutdown` — restored the link. Verification confirmed reconvergence back to the original steady state: Ethernet0/1 Root FWD (cost 100), Ethernet0/2 back to Alternate Blocking.

### Task 2 — Group the Shelter VLANs with MST
Identical MST region configuration was applied on all three switches:
```
spanning-tree mode mst
spanning-tree mst configuration
 name RYSEN-CORE
 revision 5
 instance 1 vlan 10,20
 instance 2 vlan 30,40
 exit
```
`show spanning-tree mst configuration` confirmed matching output across all three switches — region name `RYSEN-CORE`, revision 5, and identical instance-to-VLAN mapping (Instance 1: VLANs 10,20; Instance 2: VLANs 30,40; Instance 0 (default/CIST): everything else).

`show spanning-tree mst` showed each switch computing root election **independently per instance** (MST0/CIST, MST1, MST2), each with its own "Root" line and port role table — Fallout-SW1 confirmed as root for all three instances.

**Observed anomaly — PVST boundary inconsistency:** On Fallout-SW1's and Fallout-SW2's own `show spanning-tree mst` output, both of their trunk ports (Et0/1, Et0/2 on SW1; Et0/2 on SW2) displayed as `BKN*` (broken) with a `Bound(PVST) *PVST_Inc` tag — meaning that switch, at the moment of that snapshot, believed its neighbor on that port was still speaking PVST/Rapid-PVST rather than MST, and refused to trust it into full forwarding. Fallout-SW6's final `show spanning-tree mst` output — captured after all three switches had completed the MST migration — showed **no** such inconsistency: clean `Root FWD` / `Altn BLK` roles with no `Bound(PVST)` tags. This strongly suggests SW1 and SW2's snapshots were simply captured mid-migration (before their neighbors had finished switching to MST), and the inconsistency likely cleared shortly after — but this wasn't directly re-verified on SW1 or SW2 after the full mesh had converged.

## Verification Outputs

**Fallout-SW6 — final MST state (captured last, after full convergence):**
```
##### MST0    Root: address aabb.cc00.0100 ... port Et0/1
Et0/1    Root FWD 2000000
Et0/2    Altn BLK 2000000

##### MST1 (VLANs 10,20)    Root: aabb.cc00.0100, port Et0/1
Et0/1    Root FWD 2000000
Et0/2    Altn BLK 2000000

##### MST2 (VLANs 30,40)    Root: aabb.cc00.0100, port Et0/1
Et0/1    Root FWD 2000000
Et0/2    Altn BLK 2000000
```
No `Bound(PVST)` flags — clean MST-native state, confirming the region fully converged by this point.

**Trunk state after MST activation (Fallout-SW2):**
```
Vlans allowed and active in management domain: Et0/1 10,20,30,40 | Et0/2 10,20,30,40
Vlans in spanning tree forwarding state and not pruned: Et0/1 10,20,30,40 | Et0/2 none
```

## Final State Summary
- All three switches migrated from legacy PVST (`ieee`) to Rapid PVST (`rstp`) successfully, with Fallout-SW1 remaining root throughout.
- Rapid PVST failover was directly demonstrated on Fallout-SW2: shutting down its Root Port forced traffic onto the previously-blocked alternate path (cost rose from 100 to 200, confirming the reroute through Fallout-SW6), and restoring the primary link returned the topology to its original steady state.
- MST region `RYSEN-CORE` (revision 5) was configured identically on all three switches, grouping VLANs 10/20 into Instance 1 and VLANs 30/40 into Instance 2.
- A transient `Bound(PVST) *PVST_Inc` boundary inconsistency appeared on Fallout-SW1 and Fallout-SW2 during the MST migration — expected behavior while switches are mid-transition to a shared MST region, but not explicitly re-verified as cleared on those two switches after Fallout-SW6 completed its own migration.
- Outstanding follow-up: re-run `show spanning-tree mst` on Fallout-SW1 and Fallout-SW2 to directly confirm the `Bound(PVST)` flags cleared once the full three-switch mesh was running MST consistently — mirrors the same "verify on all three, not just two" gap flagged in the previous lab.
