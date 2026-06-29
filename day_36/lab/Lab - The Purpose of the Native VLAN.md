## Mission Briefing

The virtualization cluster that keeps Castle Rysen's coffee roasters humming just rolled into the bunker lab. Jeremy wants you to trace how untagged management frames survive the trunked wasteland. Today's drill: discover the native VLAN's roots, repurpose it for the management subnet, and squash the mismatch that could splice guest traffic straight into the command bunker.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Audit the Cafe-SW1 <-> Cafe-SW2 uplink to confirm its starting switchport state and the VLANs already in the lab.
- Carve out VLAN 99 as the Castle Rysen management space, force the Cafe-SW1 uplink into 802.1Q trunking, and shift its native VLAN.
- Align Cafe-SW2 with the same trunk and native VLAN settings, then verify the uplink settles into a mismatch-free trunking state.

---

## Task 0 - Profile the Untagged Lifeline

Document how the Ethernet0/1 link between Cafe-SW1 and Cafe-SW2 currently handles untagged frames so you know what "normal" looks like before you start rewiring management traffic. In this lab, the link starts physically connected, but it is not trunking yet.

**Steps:**

1. From the Cafe-SW1 console, enter privileged EXEC mode and inspect the trunk summary. A blank result means no interfaces are trunking yet.
2. Check the switchport details for Ethernet0/1. Confirm that it starts in `dynamic auto`, is operating as a static access port, and uses access VLAN 1.
3. Confirm VLANs 10 and 20 already exist for the lab access ports, then review the switch messages for any existing native VLAN mismatch alerts so you have a clean baseline.

---

## Task 1 - Anchor Management Traffic on Cafe-SW1

Prepare VLAN 99 as the management enclave on Cafe-SW1, force Ethernet0/1 into 802.1Q trunking, and move that trunk's native VLAN so untagged frames land in the right VLAN.

**Steps:**

1. Still on Cafe-SW1, define VLAN 99 with a name that clearly marks it as the management landing zone for untagged control traffic.
2. On Ethernet0/1, set the trunk encapsulation to 802.1Q, force the interface into trunk mode, and set VLAN 99 as the native VLAN.
3. Check the trunk report and switch logs to confirm Ethernet0/1 now shows VLAN 99 as native. A CDP native VLAN mismatch warning appears because Cafe-SW2 is still on VLAN 1.

---

## Task 2 - Stabilize the Native VLAN Across the Uplink

Bring Cafe-SW2 in line with the new management plan so the uplink stops complaining and untagged packets on the trunk ride safely on VLAN 99.

**Steps:**

1. Access Cafe-SW2, enter privileged EXEC mode, and ensure VLAN 99 exists with the same management label you used on Cafe-SW1.
2. On Ethernet0/1, set the trunk encapsulation to 802.1Q, force the interface into trunk mode, and set VLAN 99 as the native VLAN so both ends agree.
3. Verify the trunk summary now lists VLAN 99 as native on both sides. If you check the log, old mismatch entries may still be listed, but no new mismatch messages should appear after Cafe-SW2 is corrected.

---

## Completion Check

- `show interface trunk | begin Port` on Cafe-SW1 lists Ethernet0/1 as trunking with VLAN 99 as the native VLAN while VLANs 1, 10, 20, and 99 are active and forwarding.
- `show interface trunk | begin Port` on Cafe-SW2 mirrors the change.
- `show logging | include Native|mismatch` may still show the earlier historical mismatch messages, but no new native VLAN mismatch messages appear after both sides are corrected.
- Devices that send untagged traffic toward the uplink now land inside VLAN 99, confirming the Castle Rysen management VLAN is ready for the trunk.

 copy on select