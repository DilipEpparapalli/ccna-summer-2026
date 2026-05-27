## Mission Briefing

You’re back in the Castle Rysen bunker lab, peering over Jeremy’s shoulder at **Switch6**, the aggregation switch that keeps the surveillance wing, café floor, and uplink spine humming. Today’s drill is all about translating what the interfaces are telling you: reading the summary sheets, spotting negotiation mismatches before they take down the barista tablets, and using the CAM table like a map so you always know what’s hanging off each port. Jeremy will walk you through the workflow once—after that, it’s on you to read the prompts and keep the coffee flowing.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Distinguish interface naming conventions and roles using the switch’s summary views.
- Evaluate duplex and speed negotiation on active ports to flag connectivity issues early.
- Correlate interface descriptions and CAM table entries to pinpoint where devices live.

---

## Task 0 — Read the Interface Roll Call

From the `Switch6` console, confirm you can reach the elevated prompt, then pull the quick status sheet that lists every interface so you can see which Ethernet ports are live versus dormant.

**Steps:**

1. Climb from the user prompt to the privileged prompt on `Switch6`.
2. Display the concise interface roster that shows each port name, IP assignment, and operational state.
3. Note the interface numbering (`Ethernet0/0` through `Ethernet0/3`) so you can match ports to the deployment diagram.

---

## Task 1 — Inspect Duplex and Speed Negotiation

Still at the privileged prompt, call up the status view that reveals duplex, speed, and VLAN information so you can verify everything is running full tilt and auto-negotiated correctly.

**Steps:**

1. Stay on `Switch6#` and present the interface status snapshot that includes VLAN, duplex, and speed columns.
2. Focus on Et0/0 for the core uplink, Et0/1 for AccessPoint1, and Et0/2 for SensorPod-A; confirm each reports full duplex and auto-negotiated speed.
3. Record any port that drops to half duplex or unexpectedly locks to a low speed so you can escalate it for follow-up.

---

## Task 2 — Match Descriptions to Physical Runs

Use the switch’s descriptions to tie interfaces back to their cable endpoints so you can avoid tracing wires in the dark.

**Steps:**

1. While still on `Switch6#`, display the view that lists each interface alongside its description text.
2. Read the descriptions so you can map Et0/0 to the core uplink, Et0/1 to AccessPoint1, and Et0/2 to SensorPod-A without tracing cables.
3. Confirm every described interface is currently in the state you expect based on the topology.

---

## Task 3 — Trace Devices with the CAM Table

Interrogate the MAC address table to locate the coffee shop endpoints and recognize when a port is actually another switch in disguise.

**Steps:**

1. From `Switch6#`, display the learned MAC addresses so you can see which ports are populated.
2. Spot the uplink port by finding the interface that lists multiple remote MAC addresses (the CoreSwitch itself plus OpsServer riding behind it).
3. Filter the table for the access point’s MAC address (5a5a.1c1c.0d0d)—it’s preloaded in the switch so you can immediately confirm the expected port.

```

```

---

## Completion Check

- The interface summary on Switch6 confirms the state of `Ethernet0/0` through `Ethernet0/3`.
- The status report shows full-duplex, healthy links for AccessPoint1, SensorPod-A, and the core uplink, or flags any negotiation issues you noted.
- The MAC address table review pinpoints where each device connects and highlights which ports backhaul to the core.

Castle Rysen now knows you can read a switch like a storybook—consider yourself cleared to audit the rest of the bunker floor.