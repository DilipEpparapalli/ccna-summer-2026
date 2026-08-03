## Mission Briefing

Jeremy drags the bunker simulator into overtime because Castle Rysen's fallout shelter trunks are ready for faster convergence. The switches start in legacy PVST mode, but the VLAN instances must be present before you can compare timers and port roles. Your drill is to restore the shelter VLANs, migrate the triangle to Rapid PVST, prove the alternate path reacts quickly, and then blueprint a matching MST region.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Restore VLANs 10, 20, 30, and 40 on Fallout-SW1, Fallout-SW2, and Fallout-SW6.
- Identify the current IEEE 802.1D PVST behavior, including timers, root election, and the blocked redundant link on Fallout-SW2.
- Migrate all three switches to Rapid PVST and validate the faster failover behavior.
- Define an MST region plan that groups the shelter VLANs into efficient instances and confirm every switch agrees on the mapping.

---

## Task 0 - Audit the Legacy Timers

Verify how the network is currently converging so you can measure the improvement once Rapid PVST takes over. Start by restoring the VLAN instances on all three switches, then audit from the Fallout-SW2 console.

**Steps:**

1. On Fallout-SW1, Fallout-SW2, and Fallout-SW6, create VLANs 10, 20, 30, and 40 with the following names: vlan 10: Shelter-Operations, vlan 20: Shelter-Logistics, vlan 30: Shelter-Medical, and vlan 40: Shelter-Comms.
2. On Fallout-SW2, review the spanning-tree summary and VLAN 10 details. Fallout-SW1 starts as the root because it has the lowest configured bridge priority.
3. Document that Fallout-SW2 uses `Ethernet0/1` as the root forwarding port and keeps `Ethernet0/2` as the alternate blocking path.

---

## Task 1 - Enable Rapid PVST Across the Shelter

Shift every switch into rapid per-VLAN spanning tree so the redundant paths remember their alternate status.

**Steps:**

1. On Fallout-SW1, Fallout-SW2, and Fallout-SW6, activate Rapid PVST mode.
2. Give the switches a few seconds to rebuild the VLAN STP instances. Immediately after changing modes, IOS may briefly report that an instance does not exist.
3. Verify the new mode and confirm Fallout-SW1 remains the root for VLANs 10 through 40.
4. On Fallout-SW2, shut down `Ethernet0/1` to force traffic onto `Ethernet0/2`, then restore `Ethernet0/1` and verify the steady-state roles return.

---

## Task 2 - Group the Shelter VLANs with MST

Design an MST region so the switch stack can scale beyond a few VLANs without burning extra CPU. Begin on Fallout-SW1, then replicate the exact same region on Fallout-SW2 and Fallout-SW6.

**Steps:**

1. Define the MST region name `RYSEN-CORE` and revision `5`.
2. Map VLANs 10 and 20 into MST instance 1, and VLANs 30 and 40 into MST instance 2.
3. Confirm every switch reports the same MST configuration. If `show spanning-tree mst` briefly says no MST information is available, wait a few seconds and run the command again.

---

## Completion Check

- VLANs 10, 20, 30, and 40 exist on all three switches before STP mode testing.
- Spanning-tree summaries on all switches show `rapid-pvst` before shifting to `mst`, and Fallout-SW1 remains the preferred root for the shelter VLANs.
- Fallout-SW2 demonstrates rapid failover: `Ethernet0/2` becomes root forwarding while `Ethernet0/1` is shut, then returns to alternate blocking after `Ethernet0/1` comes back.
- MST configuration outputs confirm the region name, revision, and VLAN-to-instance mapping match across every switch.