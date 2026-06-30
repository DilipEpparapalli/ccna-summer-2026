## Mission Briefing

District Shop 01 is buzzing again, and Jeremy has you back in the bunker simulator to finish the VLAN rollout before the morning queue hits the door. The router-on-a-stick subinterfaces are configured, but the router parent interface still needs to be enabled, the switch uplinks are not trunking yet, the wireless uplinks still pass untagged traffic, the Plex streamer sits in VLAN 1, and every idle jack is a potential breach. Bring the cafe switches into compliance so patrons and admins land in the right subnets without exposing Castle Rysen's beans to wasteland freeloaders.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Confirm the live VLAN map on Cafe-SW1 and Cafe-SW2 and identify the real IOL interface names used by the lab.
- Enable the router-facing trunk, inter-switch trunk, and wireless access point trunks so VLANs 10 and 20 can cross the cafe.
- Return the Plex server to the admin VLAN and secure all unused switchports to harden the cafe edge.

## Live Interface Map

- Cafe-RTR1 `Ethernet0/0` connects to Cafe-SW1 `Ethernet0/0`.
- Cafe-SW1 `Ethernet0/1` connects to Cafe-SW2 `Ethernet0/1`.
- Cafe-SW1 `Ethernet1/0` connects to Cafe-WAP1.
- Cafe-SW2 `Ethernet1/0` connects to Cafe-WAP2.
- Cafe-SW2 `Ethernet6/0` connects to Cafe-PLEX1.
- Cafe-PLEX1 uses Linux console commands and logs in with username `cisco` and password `cisco`.

---

## Task 0 - Map the Cafe VLAN Assignments

Capture the current VLAN landscape on both switches so you know which ports already belong to the admin and patron segments and which links still need attention.

**Steps:**

1. From the Cafe-SW1 console, collect the VLAN database. VLANs 10 and 20 exist, but they initially use the default names `VLAN0010` and `VLAN0020`.
2. Review the trunk summary. At the start of the lab, `show interface trunk` returns directly to the prompt because no switch interfaces are actively trunking yet.
3. Move to Cafe-SW2, repeat the VLAN and trunk checks, and note that Cafe-PLEX1 is connected to `Ethernet6/0`, not `Ethernet0/24`.

---

## Task 1 - Light Up the Router, Switch, and Wireless Trunks

Enable the router parent interface, standardize the VLAN names, and force the cafe trunk links to carry tagged VLAN 10 and VLAN 20 traffic.

**Steps:**

1. On Cafe-RTR1, enable `Ethernet0/0` so the already-configured router-on-a-stick subinterfaces can come up.
2. On Cafe-SW1, name VLANs 10 and 20, then force `Ethernet0/0`, `Ethernet0/1`, and `Ethernet1/0` into 802.1Q trunk mode with only VLANs 10 and 20 allowed.
3. On Cafe-SW2, name VLANs 10 and 20, then force `Ethernet0/1` and `Ethernet1/0` into 802.1Q trunk mode with only VLANs 10 and 20 allowed.
4. Verify the trunk table. The WAP trunks may show `none` in the spanning-tree forwarding section until traffic is present, but the trunk ports should appear as trunking with VLANs 10 and 20 allowed and active.

---

## Task 2 - Rehome the Plex Server

Place the Plex media server back inside the admin subnet so the bunker team can stream morale videos without crossing VLAN boundaries.

**Steps:**

1. On Cafe-SW2, enter `Ethernet6/0`, the access port wired to Cafe-PLEX1.
2. Apply a descriptive label and make the port an access interface in VLAN 10.
3. From the Cafe-PLEX1 console, log in with `cisco` / `cisco`, verify the `/27` address, and ping the admin gateway at `10.0.18.1`. If the first ping loses a packet while ARP settles, run the ping again.

---

## Task 3 - Seal the Idle Ports

Disable every unused switchport so scavengers cannot plug in rogue gear and ride Castle Rysen's VLANs.

**Steps:**

1. On Cafe-SW1, shut down the unused IOL interfaces from `Ethernet1/1` through `Ethernet6/3`. Leave `Ethernet0/0`, `Ethernet0/1`, `Ethernet0/2`, `Ethernet0/3`, and `Ethernet1/0` active.
2. On Cafe-SW2, shut down the unused IOL interfaces while leaving `Ethernet0/1`, `Ethernet0/3`, `Ethernet1/0`, `Ethernet1/2`, and `Ethernet6/0` active.
3. Review interface status and VLAN tables on both switches to confirm the unused ports now show `UNUSED-LOCKDOWN`, `disabled`, and VLAN 1.

---

## Completion Check

- `show interface trunk | begin Port` on Cafe-SW1 lists `Et0/0`, `Et0/1`, and `Et1/0` as trunking with VLANs 10 and 20 allowed.
- `show interface trunk | begin Port` on Cafe-SW2 lists `Et0/1` and `Et1/0` as trunking with VLANs 10 and 20 allowed.
- `show vlan brief` on Cafe-SW2 shows `Et6/0` under VLAN 10, and Cafe-PLEX1 successfully pings `10.0.18.1` using the `255.255.255.224` mask.
- Each switch reports the unused ports as administratively disabled in VLAN 1 with the `UNUSED-LOCKDOWN` description.

 copy on select