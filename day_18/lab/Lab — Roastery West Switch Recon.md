## Field Briefing

Jeremy just handed you the keys to Roastery West, a newly lit Castle Rysen micro-roastery on the outskirts of the blast zone. Two switches and three critical PCs are online, but the senior engineer wants a ground-truth map before pushing automation. Your job: work the consoles, discover exactly how the access switch ties into the core, identify which ports host each endpoint, and label every connected switchport before handing off your notes.

## Objectives

For this deployment to be successful, you must complete the following:

- Confirm the physical and logical ties between `RW-CORE-SW` and `RW-ACC-SW`, including VLAN usage and uplink interfaces.
- Trace each workstation’s IP address to its MAC entry and access port without disturbing service.
- Document the final topology in engineering notes so the senior engineer can move straight to remediation work.

**Access Credentials**

- Switch console / VTY: `cisco` / `cisco`
    
- Privileged access: `CrC0ffee!`
    
- Note: `RW-CORE-SW` management IP is `192.168.42.1/24`; `RW-ACC-SW` uses `192.168.42.2/24`.
    

---

## Task 1 — Establish the Core View

**Objective:** Build a baseline of the core switch so you know which interfaces feed the access layer.

- Log into `RW-CORE-SW`, confirm the hostname prompt, and identify the trunk toward Roastery West’s access switch; add an accurate interface description once you confirm its role.
- Catalog every active access-facing interface on the core and add clear descriptions for any devices you discover.
- Capture the current management IP addressing on the core so later notes reference the correct subnet and default gateway.



## Task 2 — Discover the Access Layer

**Objective:** Map `RW-ACC-SW` and verify how it feeds the workstations.

- Connect to `RW-ACC-SW`, validate the hostname, and discover which access ports feed InventoryStation and ManagerConsole; add clear interface descriptions once you confirm each connection.
- Identify the uplink that returns to `RW-CORE-SW`, including its trunk VLAN list, and confirm the link reports as up/up.
- Record the management interface details on the access switch so the senior engineer can remote in after your hand-off.



## Task 3 — Trace Each Endpoint

**Objective:** Convert the reported workstation IPs into MAC-to-port mappings using switch-only visibility.

- Obtain the MAC addresses for `192.168.42.37` (BaristaPOS), `192.168.42.38` (InventoryStation), and `192.168.42.39` (ManagerConsole) by refreshing the ARP table from the switch viewpoints.
- Trace each MAC until you know exactly which switch and interface host the device, adding or refining interface descriptions as you go.
- Document your findings: IP, MAC, switch name, interface, and the descriptions you applied to those interfaces, then store the notes in the Roastery West log for the senior engineer.


## Task 4 — Validate Continuity

**Objective:** Ensure connectivity between endpoints and the core remains intact after your trace work.

- Recheck the MAC tables on both switches to verify all three workstation entries remain present and associated with the interfaces you recorded.
- Confirm the trunk between the two switches still carries the VLAN for the workstations and has not been error-disabled or renegotiated unexpectedly.
- Summarize any anomalies (missing MAC, downed port, mismatched VLAN) so escalation has actionable data.


---

## Completion Check

- You have a written record tying `192.168.42.37`, `.38`, and `.39` to specific MAC addresses, switch names, and access ports, including the interface descriptions you added.
- The uplink between `RW-CORE-SW` and `RW-ACC-SW` is verified as operational with the correct VLAN assignment.
- Both switches’ management interfaces are documented with IP, gateway, and reachability status, ready for senior engineer follow-up.
