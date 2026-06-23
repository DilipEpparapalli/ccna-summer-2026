## Mission Briefing

The bunker simulators are spinning up the district cafe access layer. Castle Rysen leadership wants hard proof that you can split a single switch into separate security zones before we deploy to the Fallout Shelters. Jeremy is standing beside you at the training rack, guiding you as you carve up the cafe switches so the administrative gear and patron kiosks stop sharing the same broadcast domain.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Define dedicated VLAN IDs for the administrative and patron device groups on both cafe switches.
- Apply clear, policy-aligned names to each VLAN to match the Castle Rysen RFP language.
- Confirm the patron-facing access ports on CafeSwitch01 now reside exclusively within their new VLAN.

---

## Task 0 - Inspect the Default VLAN Footprint

Start from the `CafeSwitch01` console prompt and document the current switching landscape before you begin carving new boundaries.

**Steps:**

1. Move into the elevated command context on CafeSwitch01 so you can review platform status.
2. Capture the active VLAN inventory to record the baseline you are about to change.
3. Note that every live access port still depends on the default VLAN prior to your adjustments.

---

## Task 1 - Establish Security VLANs on CafeSwitch01

Reconfigure CafeSwitch01 so it holds distinct containers for administrative services and cafe patrons, matching the RFP's security boundary requirements.

**Steps:**

1. Enter configuration mode on CafeSwitch01 to begin shaping the VLAN table.
2. Create VLAN ID 10 for the administrative devices and label it so operations can readily identify the segment.
3. Create VLAN ID 20 for the cafe patron access layer and assign its descriptive name.
4. Exit to the monitoring view and confirm both VLANs now appear alongside the default entry.

---

## Task 2 - Mirror the VLAN Definitions on CafeSwitch02

Duplicate the new VLAN structure on CafeSwitch02 so the downstream links share the same segmentation strategy when you later enable inter-switch trunks.

**Steps:**

1. Access the CafeSwitch02 console and elevate to the privileged context.
2. Enter configuration mode to adjust its VLAN database.
3. Add VLAN IDs 10 and 20 with the same administrative and patron names you used on CafeSwitch01.
4. Verify the VLAN table on CafeSwitch02 matches the definitions established on the first switch.

---

## Task 3 - Place Patron Ports into the Correct VLAN

Assign the cafe-facing user ports on CafeSwitch01 to the patron VLAN so any device that taps those jacks is isolated from the administrative control plane.

**Steps:**

1. Return to CafeSwitch01 and enter configuration mode to adjust the designated access ports.
2. Select Ethernet2/2, Ethernet2/3, and Ethernet3/0 through Ethernet3/3 as a group so you can apply one consistent profile.
3. Lock those interfaces to operate as single-VLAN access ports and bind them to the patron VLAN.
4. Confirm the VLAN table shows Ethernet2/2, Ethernet2/3, and Ethernet3/0 through Ethernet3/3 listed under the patron segment.

---

## Completion Check

- CafeSwitch01 lists VLAN 10 `ADMIN_DEVICES` and VLAN 20 `PATRON_DEVICES`, with Ethernet2/2, Ethernet2/3, and Ethernet3/0 through Ethernet3/3 assigned to VLAN 20.
- CafeSwitch02 replicates the VLAN 10 and VLAN 20 definitions using the same names.
- Both switches still retain the default VLAN for legacy connectivity while the new VLANs sit ready for forthcoming trunk configuration.