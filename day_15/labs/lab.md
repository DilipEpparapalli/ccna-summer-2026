# Skill 5 • Skill Lab 01 — District Shop Bring-Up

---

## Field Briefing

Castle Rysen just unlocked District Shop Delta-7. The doors can’t open until its micro-campus network is online. Supply runners dropped two access switches and a WAN edge router; everything else is up to you. Jeremy’s message is blunt: “You’ve drilled the base configs, banner work, and clean wipes. This is deployment time. Get the district lit, keep it secure, and prove the management team can reach it from the bunker — nothing fancy, just the fundamentals”

## Objectives

- Re-establish device identity, banners, and secrets across both switches and the router.
- Lock down console and VTY access with the Castle credential set and enable password encryption.
- Stand up the management VLAN on each switch and tie it to the router’s inside interface.
- Save, verify, and (when asked) wipe configurations without losing control of the site.

## Task 1. Identity & Access Hardening

- Update each device so its hostname reflects `DS-07-SW1`, `DS-07-SW2`, and `DS-07-RTR1`.
- Ensure all three systems display the Castle warning banner text `Castle Rysen Ops: Authorized engineers only.` whenever someone connects.
- Protect privileged access with the shared secret `CrC0ffee!` and confirm stored passwords are hidden by the standard encryption feature.

## Task 2. Console & VTY Security

- Require the console password `VaultAccess` before anyone can move past the initial prompt on every device.
- Guard the remote-access lines with the password `ShelterAccess`, limiting them to SSH sessions, and confirm both passwords are stored in encrypted form.

## Task 3. Switch Management Interfaces

- Assign DS-07-SW1 the management address `192.168.10.11/24`, ensure the virtual interface is active, and document which physical ports face the router and the neighboring switch.
- Assign DS-07-SW2 the management address `192.168.10.12/24`, confirm the interface is active, and label the uplink along with the ports that feed the existing WAP and server drops.
- Point both switches at `192.168.10.1` as their management gateway.

## Task 4. Router Management Interface

- Place the router’s inside interface on `192.168.10.1/24`, describing the link back to DS-07-SW1.
- Confirm the interface is brought online so switch traffic reaches the router through that connection.

## Task 5. Save and Verify

- Compare the live configuration to the saved startup copy on each device to check whether the new banner, hostnames, and secrets have been preserved.
- Commit the changes to non-volatile storage, repeat the comparison, and restart DS-07-SW1 to confirm the passwords still take effect (`VaultAccess` on the console, `CrC0ffee!` for privileged access).

---

## Completion Check

- Every prompt reads `DS-07-*`, the banner flashes `Castle Rysen Ops: Authorized engineers only.`, and the enable secret reports as type 5 under `show running-config`.
- Both switches report the management interface as operational with addresses `192.168.10.11` and `192.168.10.12`, and their management gateway is set to `192.168.10.1`.
- The router lists `192.168.10.1/24` on its inside connection with the Castle security controls active.
- Saved configurations match the running state on every device, and DS-07-SW2 powers up with the hardened settings instead of reverting to the factory prompt.

Record the outputs in the Castle Rysen engineering log. District Shop Delta-7 now has a hardened transport and is ready for survivor traffic.