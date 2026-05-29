
> A lab diagram is not provided for this drill—rely on the switch commands and descriptions to map the topology.

## Mission Briefing

You’ve been tapped by the Castle Rysen senior engineering team to run the initial investigation. They need you to map how CoreSwitch and Switch6 tie together and nail down which cables feed the affected devices before they jump in. With only switch access in your toolkit, use the show commands to piece together the physical story and hand your notes to the lead engineer.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Translate an IP-only report into a MAC address using switch-side tools.
- Walk a MAC address from CoreSwitch down to Switch6 to identify the access port.
- Confirm the endpoint location for both a user PC and a suspicious server so responders can act.

---

## Scenario Overview

- User complaint: IP `192.168.1.118` (PC7 on VLAN 10) reports sluggish service.
- Security alert: IP `192.168.1.111` (Server2 on VLAN 10) needs containment.
- Additional noise: PC8 (`192.168.1.130`) also resides on Switch6 to keep the CAM table busy.
- Switch credentials: `cisco` / `cisco` on both CoreSwitch and Switch6.

---

## Task 0 — Convert the IP Ticket into a MAC Address

Start on **CoreSwitch** so you can turn the user’s complaint into something the switches understand.

**Steps:**

1. Connect to `CoreSwitch` with username `cisco` / password `cisco`, then enter privileged EXEC mode.
2. Run a reachability test to `192.168.1.118` so the switch refreshes its ARP entry.
3. Capture the MAC address associated with `192.168.1.118` and save it for the trace.

**Captured MAC:** `5254.00d1.9e38`

---

## Task 1 — See Where CoreSwitch Thinks the Device Lives

Identify which CoreSwitch interface leads toward the complaining user.

**Steps:**

1. Still on CoreSwitch, search the CAM table for the MAC you just collected.
2. Note the interface that lists the MAC—this points you toward the downstream switch or host.
3. If the interface is a trunk to Switch6, log into Switch6 with the same credentials to continue the trace.

**Interface identified:** `Et0/0` (uplink toward Switch6)

---

## Task 2 — Nail Down the Access Port on Switch6

Finish the trace on Switch6 so you can hand the field crew an exact cable location.

**Steps:**

1. On Switch6, query the CAM table for the same MAC to confirm it now resolves to an access-facing port.
2. Match the interface description or topology notes to identify the connected device name (e.g., PC7).
3. Record the switch/port combination in your incident notes.

**Access port located:** `Switch6 Et0/1` (PC7 drop)

---

## Task 3 — Repeat the Process for the Security Alert

Trace the SOC’s flagged server so containment teams can isolate it quickly.

**Steps:**

1. Back on CoreSwitch, ping `192.168.1.111` and capture its MAC address with `show arp`.
2. Check CoreSwitch’s CAM table to learn which interface hosts the MAC.
3. Confirm whether the MAC terminates directly on CoreSwitch (Server2) or flows down to Switch6. Document the final location.

**Endpoint confirmed:** `CoreSwitch Et0/1` (Server2 access port)

---

## Completion Check

- You produced MAC-to-port mappings for both `192.168.1.118` and `192.168.1.111` without guessing or unplugging anything.
- Your notes specify the exact switch/port for PC7 (Switch6 Et0/1) and Server2 (CoreSwitch Et0/1).
- You relied on ARP lookups plus `show mac address-table` to navigate the collapsed-core layout, even with extra CAM entries from PC8.

You just proved you can find the right cable in a maze of blinking lights—exactly what Castle Rysen’s engineers need when tickets start piling up. Keep that tracer instinct sharp.