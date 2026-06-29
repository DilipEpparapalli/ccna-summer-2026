## Mission Briefing

Castle Rysen leadership just intercepted chatter about scavengers hauling rogue switches into the district lobbies. Jeremy pulls you into the bunker simulator to harden every uplink before someone rides the old dynamic trunking handshake into our VLAN core. Your drill: freeze trunks into a no-compromise state and silence the VLAN Trunking Protocol so nothing can rewrite the network from a forgotten closet.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Convert the Cafe-SW1 Cafe-SW2 backbone into a static trunk that refuses all DTP negotiations.
- Harden the Cafe-SW1 uplink toward Cafe-RTR1 so the router-on-a-stick gateway keeps working without responding to trunk invitations.
- Reconfigure both switches so VTP sits in transparent mode under a shared domain, preventing surprise VLAN database wipes.

---

## Task 0 - Freeze the Inter-Switch Negotiation

Lock the Ethernet0/1 link between Cafe-SW1 and Cafe-SW2 into a permanent trunk state and prove DTP can't coax it back into negotiation.

**Steps:**

1. From the Cafe-SW1 console, set Ethernet0/1 (the path to Cafe-SW2) to operate as a trunk at all times and suppress the dynamic handshake packets.
2. Apply the same treatment to Ethernet0/1 on Cafe-SW2 so both ends refuse auto or desirable offers.
3. Review each switch's trunk and DTP reports to confirm Ethernet0/1 continues forwarding VLANs 10 and 20 with trunk negotiation disabled.

---

## Task 1 - Seal the Router Lifeline

Keep the Ethernet0/0 uplink from Cafe-SW1 to Cafe-RTR1 passing tagged traffic while ensuring no dynamic discovery frames ever leave the switch.

**Steps:**

1. On Cafe-SW1, confirm Ethernet0/0 faces Cafe-RTR1 and force it to stay a trunk even if the far end never answers a negotiation attempt.
2. Disable dynamic trunking on that same interface so only static 802.1Q tags reach the router.
3. Verify the router's subinterfaces remain up and the switch now lists Ethernet0/0 as a trunk with negotiation off.

---

## Task 2 - Neutralize VTP Replication

Audit the VTP state on both switches, set a Castle-approved domain, and shift to transparent mode so VLAN databases stay local.

**Steps:**

1. On Cafe-SW1, capture the current VTP status, then assign the domain name COOKIE to identify trusted peers.
2. Place Cafe-SW1 into transparent mode so it stops sending or honoring VTP advertisements while still passing trunk tags.
3. Recreate VLANs 10 and 20 after the VTP mode change if they are removed from the VLAN table.
4. Repeat the domain, mode, and VLAN checks on Cafe-SW2, then confirm both switches now report VTP mode transparent with VLANs 10 and 20 present.

---

## Completion Check

- `show dtp interface` on Cafe-SW1 lists Ethernet0/0 and Ethernet0/1 with negotiation off, and Cafe-SW2 lists Ethernet0/1 with negotiation off. `show interface trunk` still reports VLANs 10 and 20 on the hardened trunks.
- Cafe-RTR1's subinterfaces stay up/up with the expected gateway addresses, proving router-on-a-stick service survived the DTP shutdown.
- `show vtp status` on each switch displays domain COOKIE in transparent mode with the configuration revision frozen, confirming no rogue VLAN updates can propagate.
