## Mission Briefing

Jeremy has you back in the Castle Rysen command bunker with a new drill: turning a single router link into a lifeline for every VLAN the fallout shelters rely on. The admin baristas and patron kiosks now sit on different broadcast islands, and leadership wants proof you can bridge them without sacrificing segmentation. Time to snap that switch port into a trunk, carve subinterfaces on the router, and watch the packets move between worlds.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Convert the Cafe-SW1 uplink toward Cafe-RTR1 into a tagged trunk that carries VLANs 10 and 20.
- Build VLAN-aware subinterfaces on Cafe-RTR1 with the proper addressing to route between the admin and patron segments.
- Deliver dynamic addressing to both VLANs and confirm hosts can trade traffic through the router.

---

## Task 0 - Unlock the Switch-to-Router Trunk

Prepare the Cafe-SW1 interface facing Cafe-RTR1 so it forwards every VLAN the shelter depends on without stripping tags.

**Steps:**

1. From the Cafe-SW1 console, adjust Ethernet0/0 (the link into Cafe-RTR1) so it forwards VLAN traffic using tagged frames instead of remaining an access port.
2. Review the trunk status report. If it is empty at this point, continue with the router subinterface configuration and check again after the router side is tagging VLAN 10 and VLAN 20.
3. Confirm Ethernet0/0 lists VLANs 10 and 20 among the active IDs once the router side is ready.

---

## Task 1 - Carve Router-on-a-Stick Subinterfaces

Rebuild Cafe-RTR1's Ethernet0/0 interface into VLAN-specific channels so each subnet has a routed gateway.

**Steps:**

1. On Cafe-RTR1, clear the legacy address from Ethernet0/0 to make room for VLAN tagging on the link to Cafe-SW1.
2. Create logical interfaces Ethernet0/0.10 and Ethernet0/0.20, each tied to VLAN IDs 10 and 20 so the router recognizes tagged frames from the switch.
3. Assign gateway addresses 10.0.18.1/27 and 10.0.18.33/27 to the subinterfaces and verify the router's interface summary lists them as up/up.

---

## Task 2 - Serve DHCP and Prove Inter-VLAN Reachability

Stand up separate DHCP scopes for the admin and patron VLANs, then let the clients pull addresses and trade packets through Cafe-RTR1.

**Steps:**

1. On Cafe-RTR1, retire any single-scope pool still referencing the combined network.
2. Configure DHCP exclusions **before defining DHCP pools** to ensure reserved addresses are never leased.
3. Create DHCP pools `ADMIN-10` and `PATRON-20` for their respective /27 subnets.
4. Set Cafe-Admin1 and Cafe-Client1 to obtain their addresses automatically.
5. Verify cross-VLAN connectivity using ping through Cafe-RTR1.

---

## Completion Check

- Cafe-SW1 trunk shows VLANs 10 and 20 active.
- Cafe-RTR1 subinterfaces are up/up with correct /27 addressing.
- Both clients receive DHCP leases.
- Inter-VLAN ping succeeds via the router-on-a-stick configuration.