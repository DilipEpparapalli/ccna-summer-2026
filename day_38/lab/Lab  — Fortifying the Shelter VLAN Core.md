## Field Briefing

Castle Rysen command just air-dropped you into Fallout Shelter Delta, where four production segments, a shared management plane, and a router lifeline must come online before the next supply convoy arrives. Jeremy is monitoring from the bunker, but this deployment is yours: two shelter switches (`Shelter-SW1`, `Shelter-SW2`), a hardened router (`Shelter-RT1`), and the critical endpoints `Shelter-Admin1` and `Shelter-Patron1`. Survivors can't tap coffee without clean VLAN separation, so stitch these systems together, lock the trunks, and deliver routed services without exposing the core.

## Objectives

For this deployment to be successful, you must complete the following:

- Publish the shelter VLAN blueprint across `Shelter-SW1` and `Shelter-SW2`, including a dedicated management native VLAN.
- Harden every trunk so only the approved VLANs traverse the uplinks with negotiation disabled and the correct native alignment.
- Stand up router-on-a-stick services on `Shelter-RT1`, delivering gateway reachability and DHCP scopes for each VLAN.
- Rehome the shelter endpoints to their assigned segments, secure idle ports, and prove the management plan works end to end.

**Access Credentials**

- Console & VTY: `castle` / `StayAlive!12`
- Privileged EXEC: `Bunker!Shield`
- Device clock: UTC (adjust if validation timestamps require it)

---

## Task 1 — Publish the Segment Blueprint

**Objective:** Establish the VLAN catalog and VTP posture the shelter will run moving forward.

- On `Shelter-SW1`, capture the current VLAN inventory and VTP status before making changes, then set the VTP domain name to `FALLOUT` so all records share the same identifier.
- Create VLANs 10 (`MGMT-FALLOUT`), 20 (`INTERNAL-COMMS`), 30 (`VIDEO-SURVEILLANCE`), 40 (`GUEST-ACCESS`), and 99 (`MGMT-NATIVE`) on `Shelter-SW1`, and mirror the names and IDs on `Shelter-SW2`.
- Place both switches in VTP transparent mode once the catalog is in place, recording the configuration revision each device reports to confirm the database is locked against external overwrites.

## Task 2 — Harden the Trunk Fabric

**Objective:** Guarantee every uplink forwards the right VLANs without dynamic negotiations or mismatched natives.

- Between `Shelter-SW1` and `Shelter-SW2`, convert the `Ethernet0/1` link into a static trunk, disable Dynamic Trunking Protocol, and set VLAN 99 as the native on both ends.
- On `Shelter-SW1`, treat `Ethernet0/0` (the path to `Shelter-RT1`) with the same static trunk and native VLAN 99 policy so router-on-a-stick tags pass cleanly.
- Use `Ethernet1/0` on `Shelter-SW2` as the uplink toward the shelter access point, forcing it to operate as a trunk that only carries VLANs 10 and 20 while leaving its native VLAN at 99 to match the rest of the fabric.
- Document the post-change trunk status and DTP reports from each interface to prove negotiation is disabled and only the required VLANs remain allowed.

## Task 3 — Deploy the Router-on-a-Stick Gateway

**Objective:** Deliver routed gateways and DHCP services for every shelter VLAN using `Shelter-RT1`.

- Activate `Shelter-RT1`'s `Ethernet0/0` interface, then build subinterfaces `.10`, `.20`, `.30`, `.40`, and `.99` tagged for VLANs 10, 20, 30, 40, and 99 respectively, assigning the following gateway addresses and masks: 10.0.16.1/25, 10.0.16.129/25, 10.0.17.1/25, 10.0.17.129/25, and 10.0.99.1/27.
- Configure DHCP pools on `Shelter-RT1` for each subinterface. The /25 mask divides each /24 block into two separate subnets, so each pool must reference its own network address: `MGMT` serves 10.0.16.0/25 (gateway 10.0.16.1), `INTERNAL` serves 10.0.16.128/25 (gateway 10.0.16.129), `VIDEO` serves 10.0.17.0/25 (gateway 10.0.17.1), `GUEST` serves 10.0.17.128/25 (gateway 10.0.17.129), and `MGMT-NATIVE` serves 10.0.99.0/27 (gateway 10.0.99.1). Reserve the gateway address in each scope, point clients to DNS server 1.1.1.1, and publish the domain name `fallout.local`.
- Verify that the router reports each subinterface as up/up and that the DHCP service lists all four production networks plus the management subnet.

## Task 4 — Rehome and Secure Shelter Endpoints

**Objective:** Place critical hosts in their proper VLANs while eliminating idle attack surfaces.

- On `Shelter-SW2`, connect `Shelter-Admin1` to `Ethernet0/2` inside VLAN 10 and `Shelter-Patron1` to `Ethernet0/3` inside VLAN 20, ensuring interface descriptions capture each role.
- Repurpose `Ethernet0/1` on `Shelter-SW1` for the virtualization management uplink and bind it to VLAN 99 so untagged control traffic rides the new native segment.
- Shut down every unused access interface on both switches (including remaining `Ethernet0/x` and `Ethernet1/x` ports), placing them in VLAN 1 with a protective description so scavenged hardware cannot auto-join the network.
- Collect updated VLAN and interface status outputs confirming the host ports are active in the intended VLANs while the sealed ports show administratively down.

## Task 5 — Validate Segmentation and Services

**Objective:** Prove that routing, isolation, and management access operate exactly as designed.

- From `Shelter-Admin1`, renew the network connection to obtain a lease from VLAN 10, verify the issued IPv4 address lands inside 10.0.16.0/25, and confirm reachability to 10.0.16.1 as well as to the `Shelter-Patron1` gateway at 10.0.16.129.
- From `Shelter-Patron1`, renew the lease for VLAN 20, confirm traffic cannot reach the VLAN 10 host directly, then test outbound reachability toward 10.0.17.129 to ensure the router is handling inter-VLAN policy correctly.
- Review `Shelter-RT1` DHCP bindings and the trunk reports on both switches to ensure all VLANs (10, 20, 30, 40, 99) are present, the native VLAN mismatch alerts are clear, and DTP remains disabled.

---

## Completion Check

- `Shelter-SW1` and `Shelter-SW2` list VLANs 10, 20, 30, 40, and 99 with matching names while reporting VTP transparent mode inside the `FALLOUT` domain.
- Every trunk interface shows VLAN 99 as native, only the approved VLANs allowed, and DTP negotiation disabled.
- `Shelter-RT1` advertises active subinterfaces for all five VLANs and records DHCP bindings for the admin, internal, video, guest, and management networks.
- `Shelter-Admin1` and `Shelter-Patron1` hold leases in the correct subnets, permitted cross-VLAN traffic succeeds only through the router, and all unused switchports remain shut down.
