## Mission Briefing

Castle Rysen just greenlit Coffee House Beta for production, and you’re the recruit shadowing Jeremy on the final switch tune-up. Two access stacks—`Cafe-SW1` on the main floor and `Cafe-SW2` in the back office—are cabled, but the ports still wear factory defaults. Your job inside the bunker: label the interfaces, normalize duplex settings, and light up management reachability so the rollout crew can monitor the site without surprises.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Apply descriptive labels to the active access ports on `Cafe-SW1` and `Cafe-SW2` so operations can identify the uplink, point-of-sale, camera, and thermostat circuits quickly.
- Normalize interface duplex settings, then confirm healthy status and collision-free statistics on every live link.
- Assign management IP addresses on both switches and verify end-to-end reachability across the coffee house LAN.

**Access Credentials**

- Switch console / VTY: `cisco` / `cisco`
- Privileged access: `CrC0ffee!`

---

## Task 0 — Label the Active Ports

Document the live cables on each switch so the field crew knows what is plugged in where.

**Steps:**

- Connect to `Cafe-SW1`, verify the switch prompt, and note which Ethernet ports report a connected status (Et0/0-Et0/1).
- Apply the description template on `Cafe-SW1` so Et0/0 identifies the Cafe-SW2 uplink and Et0/1 indicates the Barista POS handoff.
- Move to `Cafe-SW2` and label the active access ports so Et0/0 marks the Cafe-SW1 uplink, Et0/1 labels the camera feed, and Et0/3 documents the thermostat circuit.
- Confirm the updated labels appear on the intended ports using your preferred interface summary.

---

## Task 1 — Normalize Duplex Settings

Eliminate auto-negotiation surprises on the busiest ports and confirm the links remain healthy. (Speed is fixed on these IOL interfaces, so focus on duplex only.)

**Steps:**

- On `Cafe-SW1`, adjust the uplink (Ethernet0/0) so it operates at full duplex, matching the remote switch.
- On `Cafe-SW2`, mirror the uplink duplex change and ensure the camera and thermostat drops (Ethernet0/1 and Ethernet0/3) also operate in full duplex.
- Review interface statistics on both switches to confirm the changes take effect and that collision or reset counters remain steady.

---

## Task 2 — Assign Management IPs

Put both switches on the management subnet and verify they can reach each other.

- Create `Vlan42` on both switches so the management SVI has a live VLAN database entry.
- Convert the uplink (`Ethernet0/0`) on each switch to a Dot1Q trunk and explicitly allow VLAN 42 across the link.
- Bring up the management SVI (`Vlan42`) on `Cafe-SW1` with IP `192.168.42.1/24`, then set the default gateway to `192.168.42.254`.
- Mirror the SVI configuration on `Cafe-SW2` with IP `192.168.42.2/24` and the same default gateway.
- Validate `show ip interface brief` shows the SVIs in an up/up state and confirm reachability with a `ping` between the two management addresses.

---

## Completion Check

- `show interface status` on both switches displays the updated descriptions for the Cafe-SW2/Cafe-SW1 uplink and the BaristaPOS, Cam01, and Thermostat drops.
- The uplink ports operate at full duplex, and collision counters remain steady at zero during verification.
- `ping` between `192.168.42.1` and `192.168.42.2` succeeds, and both SVIs report up/up in `show ip interface brief`.

The Roastery West switches are now prepped for the senior engineer’s final review.
