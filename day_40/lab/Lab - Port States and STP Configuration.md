## Mission Briefing

Jeremy pulls you back into Castle Rysen's bunker simulator to make sure the four-VLAN fallout shelter design does not collapse under its own redundancy. The trunks are already cabled and configured, but the VLAN instances must be restored before Rapid PVST can make decisions. Your job is to read the topology's body language, promote the right core switch into command, and prove you can steer port states without breaking the lifeline for the survivors counting on this network.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Restore VLANs 10, 20, 30, and 40 on Fallout-SW1, Fallout-SW2, and Fallout-SW6 so Rapid PVST has VLAN instances to evaluate.
- Identify the current root bridge and observe which uplinks sit in forwarding versus blocking state.
- Reconfigure Fallout-SW1 so it becomes the spanning-tree root for every shelter VLAN.
- Tune Fallout-SW2's redundant trunk cost and confirm Rapid PVST returns the backup link to the alternate blocking role.

## Live Topology

- Fallout-SW1 `Ethernet0/1` connects to Fallout-SW2 `Ethernet0/1`.
- Fallout-SW1 `Ethernet0/2` connects to Fallout-SW6 `Ethernet0/1`.
- Fallout-SW2 `Ethernet0/2` connects to Fallout-SW6 `Ethernet0/2`.

---

## Task 0 - Map the Shelter's Spanning Tree

The live lab starts with trunk interfaces configured, but VLANs 10, 20, 30, and 40 are missing from the VLAN database. Create them on all three switches first, then inspect Rapid PVST.

**Steps:**

1. On Fallout-SW1, Fallout-SW2, and Fallout-SW6, create VLANs 10, 20, 30, and 40 with the following names: vlan 10: Shelter-Operations, vlan 20: Shelter-Logistics, vlan 30: Shelter-Medical, and vlan 40: Shelter-Comms.
2. From the Fallout-SW2 console, inspect spanning tree for VLAN 10.
3. Record that Fallout-SW6 starts as the root bridge and Fallout-SW2 reaches it through `Ethernet0/2`.
4. Repeat the check for VLANs 20, 30, and 40.

---

## Task 1 - Declare Fallout-SW1 the Root

Lower Fallout-SW1's bridge priority so it wins every root election and drives deterministic port roles across the triangle.

**Steps:**

1. On Fallout-SW1, reduce the spanning-tree bridge priority for VLANs 10, 20, 30, and 40.
2. Verify Fallout-SW1 reports itself as the root bridge for each VLAN.
3. From Fallout-SW2, confirm the root ID matches Fallout-SW1 and `Ethernet0/1` becomes the root forwarding port while `Ethernet0/2` becomes alternate blocking.

---

## Task 2 - Shape the Alternate Path and Observe Reconvergence

Increase the backup trunk's cost so you can predict which port parks in blocking, then bounce it and watch Rapid PVST reconverge.

**Steps:**

1. On Fallout-SW2, increase the spanning-tree path cost on Ethernet0/2 across VLANs 10, 20, 30, and 40. The live default cost is 100, so use 300 to keep Ethernet0/1 preferred.
2. Review the spanning-tree interface details and confirm Ethernet0/2 remains the alternate blocking path with the higher cost.
3. Temporarily disable and re-enable Ethernet0/2. Immediately after `no shutdown`, IOS may briefly report no spanning-tree information for the port; wait a few seconds and verify it returns to alternate blocking.

---

## Completion Check

- VLANs 10, 20, 30, and 40 exist on Fallout-SW1, Fallout-SW2, and Fallout-SW6.
- Fallout-SW2's spanning-tree reports list Fallout-SW1 as the root bridge for VLANs 10, 20, 30, and 40 with `Ethernet0/1` serving as the root port.
- `Ethernet0/2` on Fallout-SW2 shows path cost 300 and rests in the alternate blocking role once convergence finishes.
- During the shutdown/no-shutdown cycle, Rapid PVST reconverges and `Ethernet0/2` returns to its steady-state alternate blocking role.