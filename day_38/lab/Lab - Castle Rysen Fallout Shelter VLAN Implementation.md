## Mission Briefing

Castle Rysen ordered a shelter segmentation rebuild, and the bunker simulators delivered a trimmed five-node lab. Jeremy's site survey calls for four VLANs: management, internal comms, video surveillance, and survivor guests. Your drill is to build the VLAN database on the shelter core, trunk it to the three access switches and router, place the available access ports in the right VLANs, and configure the router-on-a-stick gateway with DHCP pools for each subnet.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Publish VLANs 10, 20, 30, and 40 on `Fallout-SW1` in the `fallout` VTP domain.
- Trunk `Fallout-SW1` to `Fallout-RT1`, `Fallout-SW3`, `Fallout-SW4`, and `Fallout-SW5` using the live IOL interface map.
- Configure the available access ports and build the router-on-a-stick gateway and DHCP pools on `Fallout-RT1`.

## Live Topology Note

This lab exposes five console tabs: `Fallout-RT1`, `Fallout-SW1`, `Fallout-SW3`, `Fallout-SW4`, and `Fallout-SW5`. There are no `Fallout-SW2`, `Fallout-SW6`, or workstation console tabs in this Interactive Console lab.

Live links:

- `Fallout-SW1 Ethernet0/1` connects to `Fallout-RT1 Ethernet0/0`.
- `Fallout-SW1 Ethernet0/2` connects to `Fallout-SW3 Ethernet0/1`.
- `Fallout-SW1 Ethernet0/3` connects to `Fallout-SW4 Ethernet0/1`.
- `Fallout-SW1 Ethernet1/0` connects to `Fallout-SW5 Ethernet0/1`.

---

## Task 0 - Define the Shelter VLAN Map

Establish the fallout segmentation on `Fallout-SW1` so the core switch knows every VLAN and can share it with the access switches.

**Steps:**

1. From the `Fallout-SW1` console, enter privileged EXEC mode and capture the existing VTP status and VLAN inventory. The lab starts with only the default VLANs and no active trunks.
2. Assign the switch to the `fallout` VTP domain while keeping it in server mode.
3. Create VLANs 10, 20, 30, and 40 with names that match the management, internal communication, video surveillance, and guest access groups.

---

## Task 1 - Energize the Shelter Trunks

Activate the uplinks so the access switches and shelter router receive the VLAN catalogue.

**Steps:**

1. On `Fallout-SW1`, configure `Ethernet0/1` as the router trunk to `Fallout-RT1`.
2. Configure the three access-switch trunks: `Ethernet0/2` to `Fallout-SW3`, `Ethernet0/3` to `Fallout-SW4`, and `Ethernet1/0` to `Fallout-SW5`.
3. Visit `Fallout-SW3`, `Fallout-SW4`, and `Fallout-SW5` to confirm they learn the `fallout` VTP domain and VLANs 10, 20, 30, and 40.

---

## Task 2 - Place Shelter Access Ports in Their VLANs

Assign the available access ports to their new broadcast domains. Because this lab has no `Fallout-SW6`, use `Fallout-SW5` for both the video and guest access-port checks.

**Steps:**

1. On `Fallout-SW3`, label `Ethernet0/3` as the management console port and place it in VLAN 10.
2. On `Fallout-SW4`, label `Ethernet0/3` as the internal workstation port and place it in VLAN 20.
3. On `Fallout-SW5`, label `Ethernet0/3` as the video NVR port in VLAN 30 and `Ethernet1/1` as the guest kiosk port in VLAN 40.

---

## Task 3 - Build the Fallout Gateway

Stand up the router-on-a-stick gateway on `Fallout-RT1` and define DHCP pools for each VLAN.

**Steps:**

1. On `Fallout-RT1`, skip the initial setup dialog if it appears by answering `no`, then enter privileged EXEC mode.
2. Activate the physical `Ethernet0/0` interface, then create subinterfaces `.10`, `.20`, `.30`, and `.40` tagged for VLANs 10, 20, 30, and 40.
3. Define DHCP pools for each VLAN. This trimmed lab has no workstation tabs, so verify the router interface table and DHCP pool configuration; `show ip dhcp binding` is expected to be empty unless clients are later added.

---

## Completion Check

- `show vtp status` on `Fallout-SW1` lists the domain as `fallout`, server mode, 9 existing VLANs, and configuration revision 4 after VLAN creation.
- `show interface trunk | begin Port` on `Fallout-SW1` lists `Et0/1`, `Et0/2`, `Et0/3`, and `Et1/0` as trunking with VLANs 10, 20, 30, and 40 allowed and forwarding.
- `Fallout-SW3`, `Fallout-SW4`, and `Fallout-SW5` learn VLANs 10, 20, 30, and 40 through VTP.
- `Fallout-SW3 Et0/3`, `Fallout-SW4 Et0/3`, `Fallout-SW5 Et0/3`, and `Fallout-SW5 Et1/1` appear in VLANs 10, 20, 30, and 40 respectively.
- `Fallout-RT1` shows `Ethernet0/0` and subinterfaces `.10`, `.20`, `.30`, and `.40` as `up/up`; DHCP pools are configured, and `show ip dhcp binding` may remain empty because this lab has no workstation clients.