Full deployment: two switches (Shelter-SW1, Shelter-SW2), one router (Shelter-RT1), two live endpoints (Shelter-Admin1, Shelter-Patron1). Five VLANs including a dedicated management-native VLAN, hardened trunks, router-on-a-stick with DHCP, endpoint placement, idle port lockdown, and end-to-end validation.

## Topology

```
                              Shelter-RT1
                                   │ Et0/0 (trunk, native 99)
                                   │ subinterfaces: .10 .20 .30 .40 .99
                                   │
                              Shelter-SW1  (VTP domain: FALLOUT → Transparent)
                                   │ Et0/1 (trunk, native 99, DTP off)
                                   │
                              Shelter-SW2  (VTP domain: FALLOUT → Transparent, synced via VTP)
                        ┌──────────┼──────────────────┐
                   Et0/2│          │Et0/3              │Et1/0 (trunk → AP, native 99,
                        │          │                    │       allowed 10,20 only, DTP off)
                  Shelter-Admin1  Shelter-Patron1
                  VLAN 10          VLAN 20
                  10.0.16.2/25     10.0.16.130/25
```
![[ccna-summer-2026/day_38/lab/Network_Diagram_1.png]]
**VLAN Plan:**

| VLAN | Name | Subnet | Gateway |
|------|------|--------|---------|
| 10 | MGMT-FALLOUT | 10.0.16.0/25 | 10.0.16.1 |
| 20 | INTERNAL-COMMS | 10.0.16.128/25 | 10.0.16.129 |
| 30 | VIDEO-SURVEILLANCE | 10.0.17.0/25 | 10.0.17.1 |
| 40 | GUEST-ACCESS | 10.0.17.128/25 | 10.0.17.129 |
| 99 | MGMT-NATIVE | 10.0.99.0/27 | 10.0.99.1 |

---

## Task 1 — Publish the Segment Blueprint

### Shelter-SW1: domain + VLAN catalog

```
Shelter-SW1#show vtp status
VTP Domain Name                 :
Number of existing VLANs        : 5
Configuration Revision          : 0

Shelter-SW1(config)#vtp domain FALLOUT
Changing VTP domain name from NULL to FALLOUT

Shelter-SW1(config)#vlan 10
Shelter-SW1(config-vlan)#name MGMT-FALLOUT
Shelter-SW1(config)#vlan 20
Shelter-SW1(config-vlan)#name INTERNAL-COMMS
Shelter-SW1(config)#vlan 30
Shelter-SW1(config-vlan)#name VIDEO-SURVEILLANCE
Shelter-SW1(config)#vlan 40
Shelter-SW1(config-vlan)#name GUEST-ACCESS
Shelter-SW1(config)#vlan 99
Shelter-SW1(config-vlan)#name MGMT-NATIVE
```

Confirmed:
```
Shelter-SW1#show vtp status
VTP Domain Name                 : FALLOUT
Number of existing VLANs        : 10
Configuration Revision          : 5

Shelter-SW1#show vlan brief
10   MGMT-FALLOUT          active
20   INTERNAL-COMMS        active
30   VIDEO-SURVEILLANCE    active
40   GUEST-ACCESS          active
99   MGMT-NATIVE           active
```

### Shelter-SW2: inherited the catalog via VTP, not retyped

SW2 started as VTP server with an empty domain (5 default VLANs, revision 0). Once the Et0/1 trunk to SW1 came up, SW2 pulled the full database automatically:
```
Shelter-SW2#
*%SW_VLAN-4-VTP_USER_NOTIFICATION: VTP protocol user notification: MD5 digest
 checksum mismatch on receipt of equal revision summary on trunk: Et0/1
```
That message is a transient artifact — SW2 received SW1's advertisement mid-sync and briefly flagged a digest mismatch before settling. It resolved on its own:
```
Shelter-SW2#show vtp status
VTP Domain Name                 : FALLOUT
Number of existing VLANs        : 10
Configuration Revision          : 5
MD5 digest                      : 0x70 0x46 ... (matches SW1 exactly)

Shelter-SW2#show vlan brief
10   MGMT-FALLOUT          active
20   INTERNAL-COMMS        active
30   VIDEO-SURVEILLANCE    active
40   GUEST-ACCESS          active
99   MGMT-NATIVE           active
```
This is a legitimate technique worth remembering: pushing the VLAN database via VTP server mode *before* switching both devices to transparent mode saves manually retyping the catalog twice. Once both switches agree, transparent mode locks the database against any further external overwrite.

### Both switches locked to VTP Transparent

```
Shelter-SW1(config)#vtp mode transparent
Setting device to VTP Transparent mode for VLANS.

Shelter-SW2(config)#vtp mode transparent
Setting device to VTP Transparent mode for VLANS.
```
Both report Operating Mode: Transparent, domain FALLOUT, 10 VLANs, revision 5 — matching digests confirm the catalog is identical on both and now frozen against future VTP updates from either side.

---

## Task 2 — Harden the Trunk Fabric

### Shelter-SW1 ↔ Shelter-SW2 (Et0/1 both ends)

**Shelter-SW1 side:**
```
Shelter-SW1(config)#interface ethernet0/1
Shelter-SW1(config-if)#switchport trunk encapsulation dot1q
Shelter-SW1(config-if)#switchport mode trunk
Shelter-SW1(config-if)#switchport nonegotiate
Shelter-SW1(config-if)#switchport trunk native vlan 99
```

The moment native VLAN 99 was set on SW1's end — before SW2 had made the matching change — the switch immediately flagged the mismatch:
```
%SPANTREE-2-BLOCK_PVID_PEER: Blocking Ethernet0/1 on VLAN0001. Inconsistent peer vlan.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking Ethernet0/1 on VLAN0099. Inconsistent local vlan.
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on Ethernet0/1 (99), with Shelter-SW2 Ethernet0/1 (1).
```
This is the exact live version of the native-VLAN-mismatch mechanism discussed earlier in this session: SW1 was sending untagged frames expecting them to mean VLAN 99, while SW2 (still on default native VLAN 1) was receiving those same untagged frames and reading them as VLAN 1. STP detected the inconsistency and blocked the port on **both** conflicting VLANs (1 and 99) rather than silently letting traffic leak between them — that's PVST+'s built-in protection kicking in.

Once SW2's side was also set to native VLAN 99 (below), the mismatch cleared on its own:
```
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0001. Port consistency restored.
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0099. Port consistency restored.
```

**Shelter-SW2 side — hit a real command-ordering error:**
```
Shelter-SW2(config-if)#switchport nonegotiate
Command rejected: Conflict between 'nonegotiate' and 'dynamic' status on this interface: Et0/1
```
At this point the port was still in DTP `auto` mode (dynamically negotiated into trunking, not statically forced) — confirmed by the earlier `show interface trunk` showing `Mode: auto`. **`nonegotiate` can only be applied to a port that is already a static trunk** (`switchport mode trunk`); IOS refuses to disable negotiation on a port whose mode is still dynamic, since that would leave the port in an undefined state. The fix was to set static trunk mode first, then reapply native VLAN and nonegotiate:
```
Shelter-SW2(config-if)#switchport trunk encapsulation dot1q
Shelter-SW2(config-if)#switchport mode trunk
Shelter-SW2(config-if)#switchport trunk native vlan 99
Shelter-SW2(config-if)#switchport nonegotiate
```

**Final verification, both ends:**
```
Shelter-SW1#show interface trunk
Et0/1   on   802.1q   trunking   Native vlan 99
Vlans allowed on trunk: 1-4094

Shelter-SW2#show interface trunk
Et0/1   on   802.1q   trunking   Native vlan 99
Vlans allowed on trunk: 1-4094

Shelter-SW2#show dtp interface ethernet 0/1
TOS/TAS/TNS: TRUNK/NONEGOTIATE/TRUNK
```
DTP confirmed off (`NONEGOTIATE`), native VLAN matched at 99 on both ends, mismatch alerts cleared.

⚠️ **Deviation — not pruned:** `switchport trunk allowed vlan 10,20,30,40,99` was never applied on either end of this link. Both sides still report `1-4094` allowed. The guide's stated goal for Task 2 is "only the approved VLANs traverse the uplinks" — that objective is only half met on this particular trunk.

### Shelter-SW1 → Shelter-RT1 (Et0/0)

```
Shelter-SW1(config)#interface ethernet0/0
Shelter-SW1(config-if)#switchport trunk encapsulation dot1q
Shelter-SW1(config-if)#switchport mode trunk
Shelter-SW1(config-if)#switchport trunk native vlan 99
```
Verified:
```
Shelter-SW1#show interface trunk
Et0/0   on   802.1q   trunking   Native vlan 99
Vlans allowed on trunk: 1-4094
```

⚠️ Two things worth flagging on this port:
- Same pruning gap as Et0/1 — `1-4094` allowed instead of being restricted to the five active VLANs.
- `switchport nonegotiate` was never applied here. Functionally this is low-risk since routers don't participate in DTP at all (Shelter-RT1 can't negotiate trunking either way), but the guide's wording — "treat Ethernet0/0 ... with the same static trunk and native VLAN 99 **policy**" — reads as wanting the full policy replicated, DTP-disable included, for consistency's sake even where it's not strictly load-bearing.

### Shelter-SW2 → access point (Et1/0) — done correctly

```
Shelter-SW2(config)#interface ethernet1/0
Shelter-SW2(config-if)#switchport trunk encapsulation dot1q
Shelter-SW2(config-if)#switchport mode trunk
Shelter-SW2(config-if)#switchport trunk native vlan 99
Shelter-SW2(config-if)#switchport trunk allowed vlan 10,20
Shelter-SW2(config-if)#switchport nonegotiate
```
Verified:
```
Shelter-SW2#show interface trunk
Et1/0   on   802.1q   trunking   Native vlan 99
Vlans allowed on trunk: 10,20

Shelter-SW2#show dtp interface ethernet 1/0
TOS/TAS/TNS: TRUNK/NONEGOTIATE/TRUNK
```
This is the one trunk in the whole lab that got full hardening: static mode, DTP disabled, native aligned, **and** properly pruned to just the two VLANs the access point actually needs. This is the template the other three trunks should have matched.

---

## Task 3 — Router-on-a-Stick Gateway (Shelter-RT1)

### Subinterfaces — hit the encapsulation-before-IP rule

First attempt on `.10` tried assigning the IP before setting encapsulation:
```
Shelter-RT1(config-subif)#ip address 10.0.16.1 255.255.255.128

% Configuring IP routing on a LAN subinterface is only allowed if that
subinterface is already configured as part of an IEEE 802.10, IEEE 802.1Q,
or ISL vLAN.
```
**Why this happens:** a subinterface is just a logical placeholder until it's tied to a VLAN tag via `encapsulation dot1Q <id>`. Until that's done, IOS doesn't know which broadcast domain the subinterface belongs to, so it refuses to let you bind a Layer 3 address to it. Encapsulation always has to come first:
```
Shelter-RT1(config-subif)#encapsulation dot1Q 10
Shelter-RT1(config-subif)#ip address 10.0.16.1 255.255.255.128
```
The remaining four subinterfaces (`.20`, `.30`, `.40`, `.99`) were configured in the correct order from the start — encapsulation, then IP:
```
interface Ethernet0/0.20
 encapsulation dot1Q 20
 ip address 10.0.16.129 255.255.255.128

interface Ethernet0/0.30
 encapsulation dot1Q 30
 ip address 10.0.17.1 255.255.255.128

interface Ethernet0/0.40
 encapsulation dot1Q 40
 ip address 10.0.17.129 255.255.255.128

interface Ethernet0/0.99
 encapsulation dot1Q 99
 ip address 10.0.99.1 255.255.255.224
```

**Verified all five up/up:**
```
Shelter-RT1#show ip interface brief
Ethernet0/0            unassigned      up   up
Ethernet0/0.10         10.0.16.1       up   up
Ethernet0/0.20         10.0.16.129     up   up
Ethernet0/0.30         10.0.17.1       up   up
Ethernet0/0.40         10.0.17.129     up   up
Ethernet0/0.99         10.0.99.1       up   up
```

### DHCP pools — all five correct this time

```
ip dhcp pool MGMT
 network 10.0.16.0 255.255.255.128
 default-router 10.0.16.1
 dns-server 1.1.1.1
 domain-name fallout.local

ip dhcp pool INTERNAL
 network 10.0.16.128 255.255.255.128
 default-router 10.0.16.129
 dns-server 1.1.1.1
 domain-name fallout.local

ip dhcp pool VIDEO
 network 10.0.17.0 255.255.255.128
 default-router 10.0.17.1
 dns-server 1.1.1.1
 domain-name fallout.local

ip dhcp pool GUEST
 network 10.0.17.128 255.255.255.128
 default-router 10.0.17.129
 dns-server 1.1.1.1
 domain-name fallout.local

ip dhcp pool MGMT-NATIVE
 network 10.0.99.0 255.255.255.224
 default-router 10.0.99.1
 dns-server 1.1.1.1
 domain-name fallout.local
```
Every `default-router` correctly matches its own subinterface's IP this time — the INTERNAL-pool mistake from the earlier Fallout-shelter lab did not repeat.

**Bindings confirmed active clients:**
```
Shelter-RT1#show ip dhcp binding
10.0.16.2       ...  Automatic  Active  Ethernet0/0.10
10.0.16.130     ...  Automatic  Active  Ethernet0/0.20
```
Only MGMT and INTERNAL show leases because Shelter-Admin1 and Shelter-Patron1 are the only live endpoints in this trimmed lab — VIDEO, GUEST, and MGMT-NATIVE pools are configured and ready but have no clients to lease to, same pattern as the earlier Fallout lab's empty binding table.

---

## Task 4 — Rehome Endpoints, Secure Idle Ports

### Endpoint placement — Shelter-SW2

```
interface Ethernet0/2
 switchport mode access
 switchport access vlan 10
 description SHELTER-ADMIN1

interface Ethernet0/3
 switchport mode access
 switchport access vlan 20
 description SHELTER-PATRON1
```
Confirmed:
```
Shelter-SW2#show vlan
10   MGMT-FALLOUT      active    Et0/2
20   INTERNAL-COMMS    active    Et0/3
```

### Virtualization management uplink — Shelter-SW1 Et0/1

Already covered in Task 2 — Et0/1 was the same interface repurposed as the trunk to Shelter-SW2 with native VLAN 99 carrying the untagged management traffic. No separate action needed here since it was built as part of the trunk hardening.

### Idle port lockdown — both switches

**Shelter-SW1** — all unused ports (Et0/2-3, Et1/0-3, Et2/0-3, 11 ports total):
```
Shelter-SW1(config)#interface range ethernet 0/2-3, ethernet 1/0-3, ethernet 2/0-3
Shelter-SW1(config-if-range)#switchport mode access
Shelter-SW1(config-if-range)#switchport access vlan 1
Shelter-SW1(config-if-range)#shutdown
```

**Shelter-SW2** — remaining unused ports (Et1/1-3, Et2/0-3, 7 ports; Et0/0 was already down, Et0/1 and Et1/0 are trunks, Et0/2-3 are the endpoint ports):
```
Shelter-SW2(config)#interface range ethernet 1/1-3, ethernet 2/0-3
Shelter-SW2(config-if-range)#switchport mode access
Shelter-SW2(config-if-range)#switchport access vlan 1
Shelter-SW2(config-if-range)#shutdown
```

Confirmed administratively down on both:
```
Shelter-SW1#show ip interface brief
Ethernet1/0-1/3, Ethernet2/0-2/3   administratively down   down

Shelter-SW2#show ip interface brief
Ethernet1/1-1/3, Ethernet2/0-2/3   administratively down   down
```

⚠️ **Deviation:** the guide's Task 4 step 3 explicitly calls for "a protective description" on every sealed port. Neither switch's idle-port range command included a `description` line — the ports are shut down and parked in VLAN 1 as required, but they're unlabeled. A future admin looking at these ports would see "shutdown, VLAN 1" with no explanation of *why*, which is exactly the ambiguity a protective description is meant to prevent.

---

## Task 5 — Validate Segmentation and Services

### Shelter-Admin1 (VLAN 10)
```
cisco@shelter-admin1:~$ ifconfig
eth0   inet addr:10.0.16.2  Mask:255.255.255.128
```
Lease landed correctly inside `10.0.16.0/25` — confirms VLAN 10 → MGMT pool → correct subnet.

```
cisco@shelter-admin1:~$ ping 10.0.16.130
64 bytes from 10.0.16.130: ttl=63 ...  (0% loss)

cisco@shelter-admin1:~$ ping 10.0.17.129
64 bytes from 10.0.17.129: ttl=255 ... (0% loss)
```
Both succeeded. The `ttl=63` reply (down from Linux's default 64) confirms this packet crossed exactly one router hop — proof that Admin1 → Patron1 traffic is being routed through Shelter-RT1, not switched directly. The `ttl=255` reply to 10.0.17.129 is the router itself answering directly (that address is RT1's own GUEST subinterface, so no further hop is involved).

⚠️ **Validation gap:** the guide's Task 5 step 1 specifically asks to test reachability to *the device's own gateway* (10.0.16.1) and *the Shelter-Patron1 gateway* (10.0.16.129). What was actually tested was Patron1's **host** address (10.0.16.130) rather than its gateway (10.0.16.129), and the own-gateway ping (10.0.16.1) wasn't run at all. The tests performed still meaningfully prove inter-VLAN routing works (arguably a stronger proof, since host-to-host is a more complete test than host-to-router-interface) — but for a from-scratch validation pass, the exact addresses the guide names are worth hitting too, since exam scenarios sometimes grade against the literal completion checklist.

### Shelter-Patron1 (VLAN 20)
```
cisco@shelter-patron1:~$ ifconfig
eth0   inet addr:10.0.16.130  Mask:255.255.255.128
```
Correct subnet for INTERNAL (10.0.16.128/25).

```
cisco@shelter-patron1:~$ ping 10.0.16.2
64 bytes from 10.0.16.2: ttl=63 ... (0% loss)

cisco@shelter-patron1:~$ ping 10.0.17.129
64 bytes from 10.0.17.129: ttl=255 ... (0% loss)
```
Both succeed, `ttl=63` again confirming a single router hop for the Admin1 reachability test.

**Note on the guide's Task 5 step 2 wording:** it says to "confirm traffic cannot reach the VLAN 10 host directly" — worth reading carefully. There's no ACL anywhere in this lab restricting inter-VLAN traffic, so full reachability between VLAN 10 and VLAN 20 through the router is expected and correct. "Cannot reach... directly" most likely means *not directly at Layer 2* (i.e., only reachable by routing through RT1, never switched straight across), which is exactly what the `ttl=63` result demonstrates — it's proof the traffic **was** routed, not that it was blocked.

---

## Mistakes & Deviations Summary

| # | Where | What happened | Caught? | Impact |
|---|-------|----------------|---------|--------|
| 1 | Shelter-RT1, subinterface .10 | Tried `ip address` before `encapsulation dot1Q` | Yes — IOS rejected it with a clear error | None, corrected immediately; good rule to internalize |
| 2 | Shelter-SW2, Et0/1 | `switchport nonegotiate` rejected while port was still in dynamic (auto) trunk mode | Yes — IOS rejected it, fixed by setting `switchport mode trunk` first | None, corrected; important nonegotiate/dynamic ordering rule |
| 3 | Shelter-SW1 Et0/1, Et0/0; Shelter-SW2 Et0/1 | `switchport trunk allowed vlan` never applied — all show `1-4094` | **No** | Trunks are not pruned to the approved VLAN set; only Et1/0 on SW2 achieved full hardening |
| 4 | Shelter-SW1 Et0/0 | `switchport nonegotiate` never applied to the router-facing trunk | **No** | Low real-world impact (routers don't run DTP) but leaves the port's policy inconsistent with the rest of the fabric |
| 5 | Shelter-SW1 & Shelter-SW2 idle ports | No protective description applied when shutting down unused ports | **No** | Ports are correctly secured (shutdown, VLAN 1) but unlabeled |
| 6 | Shelter-Admin1 validation | Pinged Patron1's host IP (10.0.16.130) and skipped the own-gateway ping (10.0.16.1) instead of testing the exact addresses the guide named | N/A | Functional proof still obtained, but doesn't match the literal completion checklist |

## Key Reinforcement

- **Native VLAN mismatch is a live, visible event, not just a theory question.** Configuring native VLAN 99 on one end of a trunk before the other end matches triggers STP to block the port on both the old and new native VLANs simultaneously, plus a CDP mismatch alert — this is PVST+ actively preventing traffic from leaking between what each switch thinks the untagged VLAN is. It self-resolves the moment both ends agree.
- **`switchport nonegotiate` requires a static trunk first.** IOS won't let you disable DTP negotiation on a port that's still in dynamic (`auto`/`desirable`) mode — the port has to be explicitly forced into `switchport mode trunk` before nonegotiate is accepted.
- **Subinterfaces need `encapsulation` before they can hold an IP.** Without a VLAN tag bound to it, IOS has no broadcast domain to associate the address with.
- **VTP server mode can do the heavy lifting before you lock things down.** Publishing the VLAN catalog from one server and letting a second switch inherit it over a trunk — then switching both to transparent — avoids manually re-typing the same VLAN list twice, while still ending in a locked, VTP-independent state.
- **"Trunking" status and "hardened" are not the same thing.** Every trunk in this lab showed `Status: trunking` correctly, but only one of the four actually had its allowed-VLAN list pruned. A trunk that's up and passing traffic can still be an unfinished hardening job.
