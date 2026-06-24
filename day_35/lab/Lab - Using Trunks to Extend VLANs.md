## Mission Briefing

Welcome back to the Castle Rysen bunker. The district shops have their VLANs defined, but right now each switch is an island. Jeremy wants you on the simulation floor to stitch those islands together so admin and patron segments can ride the same cabling without bleeding together. This drill keeps us ready to stretch the network across every fallout shelter while keeping the beans safe.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Prepare the inter-switch links so VLAN tags move cleanly between Cafe-SW1 and Cafe-SW2.
- Place the admin workstations in VLAN 10 with matching addressing across both switches.
- Shift the patron workstation into VLAN 20 to confirm the segmentation holds without routing.

---

## Task 0 - Fortify the Inter-Switch Lifeline

Harden the link between Cafe-SW1 and Cafe-SW2 so every VLAN can traverse the Ethernet0/1 and Ethernet0/2 uplinks without mismatches.

**Steps:**

1. From the console of Cafe-SW1, create VLANs 10 and 20 so the trunk has active VLANs to carry.
2. Place the uplink interfaces that face Cafe-SW2 into a mode that forwards tagged VLAN traffic.
3. Repeat the same VLAN and trunk configuration on Cafe-SW2 so both ends of the backbone speak the same tagged language.
4. Review each switch's trunk report to confirm Ethernet0/1 and Ethernet0/2 are trunking and that VLANs 10 and 20 are active on the trunks.

---

## Task 1 - Stretch VLAN 10 Across the Bunker

Anchor both workstation ports inside VLAN 10 and validate that the admin subnet spans the trunk between the switches.

**Steps:**

1. On Cafe-SW1, assign the Ethernet0/3 access port that feeds Cafe-Admin1 to VLAN 10.
2. Mirror that placement on Cafe-SW2 for the Ethernet0/3 port leading to Cafe-Client1 so both endpoints share the same broadcast domain.
3. Log in to Cafe-Admin1 and Cafe-Client1 as `tc`, configure their Linux `eth0` addresses, then validate they exchange successful test traffic.

---

## Task 2 - Prove the VLAN Boundary Holds

Move the patron workstation out of VLAN 10, place it in VLAN 20 with its own subnet, and confirm the lack of routing keeps the segments isolated.

**Steps:**

1. On Cafe-SW2, reassign the Ethernet0/3 port connected to Cafe-Client1 so it belongs to VLAN 20.
2. Update Cafe-Client1 to 10.0.18.34/27 with gateway 10.0.18.33 while leaving Cafe-Admin1 unchanged.
3. Test connectivity from Cafe-Admin1 toward Cafe-Client1 and observe the traffic block that proves the VLAN separation is working.

---

## Completion Check

- `show interface trunk` on both Cafe-SW1 and Cafe-SW2 lists Ethernet0/1 and Ethernet0/2 as trunking with VLANs 10 and 20 active. Cafe-SW2 may show the redundant Et0/2 trunk as `none` in the spanning-tree forwarding section.
- `show vlan brief` confirms Ethernet0/3 on Cafe-SW1 lives in VLAN 10 and Ethernet0/3 on Cafe-SW2 lives in VLAN 20.
- Pings succeed between Cafe-Admin1 (10.0.18.2/27) and Cafe-Client1 (10.0.18.3/27) while both are in VLAN 10, and fail once Cafe-Client1 moves to VLAN 20, demonstrating the segregation.