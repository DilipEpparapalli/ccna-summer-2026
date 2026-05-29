
## Mission Briefing

The Castle Rysen diagram pit is yours today. Jeremy just wiped the auto-labels off **Switch6** so you can see every cable the old-school way. Two bunker workstations—**PC1** at `192.168.1.50/24` and **PC2** at `192.168.1.51/24`—share VLAN 10 on the access layer, and the uplink to **CoreSwitch** is already humming on VLAN 99. Your goal: generate real traffic, watch the CAM table learn live MAC addresses, and prove you can flush and repopulate entries without breaking a sweat.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Stage the endpoints so their Layer 2 identities are easy to spot from the switch.
- Trigger ARP-initiated traffic between PCs on the same VLAN to populate the CAM table.
- Inspect, clear, and repopulate the MAC address table to validate how the switch learns.

---

## Task 0 — Prep the Workstations

Verify the bunker PCs are addressed exactly as shown so any CAM table entries you see later are easy to recognize.

**Steps:**

1. Log into `PC1` with username `cisco` and password `cisco`, then verify its primary interface reports `192.168.1.50/24` with a default gateway of `192.168.1.1`.
2. Repeat the verification on `PC2` and confirm it shows `192.168.1.51/24` with the same default gateway.
3. Note which switch ports each PC uses (`Et0/1` for PC1 and `Et0/2` for PC2) so you can map MAC entries back to real cables.

---

## Task 1 — Generate the ARP Conversation

Use PC1 to reach PC2 so you can watch the ARP request bloom into the switch’s CAM table entries.

**Steps:**

1. From the desktop shell on `PC1`, prepare a connectivity test targeting `PC2`’s IP address.
2. Run the test once so PC1 is forced to resolve PC2’s MAC address before sending payload traffic.
3. Observe that the initial attempt takes slightly longer than subsequent tries because of the ARP exchange.

---

## Task 2 — Inspect the Switch CAM Table

Pivot to `Switch6` and confirm the MAC address table records both PCs and the uplink partner.

**Steps:**

1. Open the console on `Switch6` and reach the privileged prompt.
2. Display the switch’s MAC address table and note the entries tied to `Et0/1`, `Et0/2`, and `Et0/0`.
3. Match the listed MAC addresses with the devices you labeled earlier so you can identify PC1, PC2, and the uplink to CoreSwitch.

---

## Task 3 — Flush and Re-Learn Entries

Clear the dynamic table and repopulate it to prove you control what the switch remembers.

**Steps:**

1. While still on `Switch6`, remove the learned CAM entries without touching static mappings tied to interface descriptions.
2. Immediately re-run PC1’s connectivity test so ARP traffic rebuilds the table.
3. Confirm the switch once again records MAC entries on the correct interfaces after the traffic flows.

---

## Completion Check

- PC1 and PC2 display the correct static IP configurations and their ARP caches reveal one another after the test.
- Switch6’s CAM table shows distinct entries on Et0/1, Et0/2, and Et0/0 that map to PC1, PC2, and the uplink.
- Clearing the dynamic table and generating new traffic rebuilds the expected MAC address entries without manual intervention.

You just proved you can read—and reset—the lifeblood tables that keep Castle Rysen’s switches sane. Next stop: hunting down endpoints in the wild.