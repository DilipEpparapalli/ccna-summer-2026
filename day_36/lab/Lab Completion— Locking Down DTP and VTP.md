## Lab Environment
- **Platform:** Cisco Modeling Labs (CML) — IOSv Layer 2 switches and IOSv router
- **Devices:** Cafe-SW1, Cafe-SW2, Cafe-RTR1
- **Topology:**

```
Cafe-RTR1
    │ Et0/0 (trunk, ROAS)
    │
Cafe-SW1 ── Et0/1 (trunk) ── Cafe-SW2
```
![[Network_Diagrm.png]]
## Objective
Harden all trunk links by disabling DTP negotiation and locking port modes
explicitly. Set both switches to VTP transparent mode under a shared domain name
to prevent VLAN database replication. Confirm ROAS subinterfaces survive the
hardening and VLANs 10 and 20 remain active across both switches.

---

## Task 0 — Baseline: Both Switches Had No Custom VLANs

Both switches started fresh — no VLAN 10 or 20, all ports in VLAN 1.

```
! Cafe-SW1 baseline show vlan:
1    default    active    Et0/0, Et0/2, Et0/3
```

Et0/1 (inter-switch link) was already up but not yet a trunk.

---

## Task 1 — Harden Inter-Switch Trunk (Cafe-SW1 Et0/1)

### Error — Trunk Mode Rejected Without Encapsulation First

Attempted to set trunk mode directly:

```
Cafe-SW1(config-if)# switchport mode trunk
Command rejected: An interface whose trunk encapsulation is "Auto" can not be
configured to "trunk" mode.
```

IOSv requires explicit dot1Q encapsulation before trunk mode — same as every
previous CML lab. Corrected sequence:

```
Cafe-SW1(config)# interface ethernet0/1
Cafe-SW1(config-if)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if)# switchport mode trunk
Cafe-SW1(config-if)# switchport nonegotiate
Cafe-SW1(config-if)# end
```

Interface flapped briefly — expected when changing port mode.

### Verify DTP Disabled — Cafe-SW1 Et0/1

```
Cafe-SW1# show dtp interface ethernet 0/1

DTP information for Ethernet0/1:
  TOS/TAS/TNS:              TRUNK/NONEGOTIATE/TRUNK
  Hello timer expiration:   never/STOPPED
  Negotiation timer:        never/STOPPED
  FSM state:                S6:TRUNK
```

**What this confirmed:**
- `TAS: NONEGOTIATE` — DTP advertisements are suppressed, port will not respond to
  or send negotiation frames
- All timers show `never/STOPPED` — no DTP activity on this port
- `FSM state: S6:TRUNK` — port is permanently in trunk state

### Verify Trunk Status — Cafe-SW1 After Et0/1 Only

```
Cafe-SW1# show interfaces trunk

Port   Mode  Encapsulation  Status    Native vlan
Et0/1  on    802.1q         trunking  1

Port   Vlans allowed on trunk
Et0/1  10,20

Port   Vlans allowed and active in management domain
Et0/1  none     ← VLANs 10 and 20 don't exist yet on this switch
```

Trunking confirmed. VLANs showed `none` in the active section because VLAN 10 and
20 hadn't been created yet — that happens in Task 3 after VTP is configured.

---

## Task 2 — Harden Router Uplink (Cafe-SW1 Et0/0)

Applied the same three-command sequence to the switch port facing Cafe-RTR1:

```
Cafe-SW1(config)# interface ethernet0/0
Cafe-SW1(config-if)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if)# switchport mode trunk
Cafe-SW1(config-if)# switchport nonegotiate
Cafe-SW1(config-if)# end
```

### Router Subinterfaces Were Down — Physical Interface Was Shutdown

After hardening Et0/0 on the switch side, checked Cafe-RTR1:

```
Cafe-RTR1# show ip interface brief

Interface       IP-Address    OK?  Status                  Protocol
Ethernet0/0     unassigned    YES  administratively down   down
Ethernet0/0.10  10.0.18.1     YES  administratively down   down
Ethernet0/0.20  10.0.18.33    YES  administratively down   down
```

The physical interface Et0/0 on the router was shut down — subinterfaces inherit
state from their parent, so both were also down. Fixed with:

```
Cafe-RTR1(config)# interface ethernet0/0
Cafe-RTR1(config-if)# no shutdown
Cafe-RTR1(config-if)# end
```

**Verified subinterfaces came back up:**

```
Interface       IP-Address    OK?  Status  Protocol
Ethernet0/0     unassigned    YES  up      up
Ethernet0/0.10  10.0.18.1     YES  up      up
Ethernet0/0.20  10.0.18.33    YES  up      up
```

ROAS restored. Both subinterfaces up/up with correct gateway IPs.

**After both Et0/0 and Et0/1 were hardened, show interfaces trunk confirmed:**

```
Port   Mode  Encapsulation  Status    Native vlan
Et0/0  on    802.1q         trunking  1
Et0/1  on    802.1q         trunking  1

Port   Vlans allowed on trunk
Et0/0  10,20
Et0/1  10,20
```

---

## Task 3 — Harden Inter-Switch Trunk on Cafe-SW2

Applied same trunk hardening to Cafe-SW2 Et0/1:

```
Cafe-SW2(config)# interface ethernet0/1
Cafe-SW2(config-if)# switchport trunk encapsulation dot1q
Cafe-SW2(config-if)# switchport mode trunk
Cafe-SW2(config-if)# switchport nonegotiate
Cafe-SW2(config-if)# end
```

**Interesting observation — VTP MD5 checksum mismatch appeared:**

```
%SW_VLAN-4-VTP_USER_NOTIFICATION: VTP protocol user notification:
MD5 digest checksum mismatch on receipt of equal revision summary on trunk: Et0/1
```

And in `show vtp status` on Cafe-SW2:

```
*** MD5 digest checksum mismatch on trunk: Et0/1 ***
```

**What this means:** When Cafe-SW1 was set to VTP domain COOKIE (in Task 4 below),
it started sending VTP advertisements with the new domain name. Cafe-SW2 was still
in its default domain (NULL/empty) and received those frames — the MD5 hash in the
advertisement didn't match what Cafe-SW2 expected. This is VTP's way of flagging
that it received an update it couldn't validate. It's a warning, not a failure — and
it confirmed that VTP was still active and needed to be shut down on SW2. Fixed by
moving SW2 to transparent mode in the next task.

---

## Task 4 — VTP Transparent Mode on Both Switches

### Cafe-SW1 VTP Baseline

```
Cafe-SW1# show vtp status

VTP Operating Mode    : Server     ← default, dangerous
VTP Domain Name       :            ← no domain set
Configuration Revision: 0
```

### Error — Typo on VTP Domain Command

```
Cafe-SW1(config)# vtp domin COOKIE
% Invalid input detected at '^' marker.
```

Correct command is `vtp domain` (not `domin`). Minor typo, caught immediately by IOS.

### Configure VTP on Cafe-SW1

```
Cafe-SW1(config)# vtp domain COOKIE
Changing VTP domain name from NULL to COOKIE.

Cafe-SW1(config)# vtp mode transparent
Setting device to VTP Transparent mode for VLANS.

! Re-create VLANs — transparent mode manages VLAN DB locally
Cafe-SW1(config)# vlan 10
Cafe-SW1(config-vlan)# name ADMIN-DEV
Cafe-SW1(config)# vlan 20
Cafe-SW1(config-vlan)# name PATRON-DEV
```

**Why re-create VLANs:** When a switch is in VTP server or client mode, VLANs can
be wiped when the mode changes — the local VLAN database may be cleared. In
transparent mode the switch manages its own database, so VLANs must be defined
locally. Always verify with `show vlan` after a VTP mode change.

### Configure VTP on Cafe-SW2

```
Cafe-SW2(config)# vtp mode transparent
Setting device to VTP Transparent mode for VLANS.

Cafe-SW2(config)# vlan 10
Cafe-SW2(config-vlan)# name ADMIN-DEV
Cafe-SW2(config)# vlan 20
Cafe-SW2(config-vlan)# name PATRON-DEV
```

Note: SW2 already picked up domain COOKIE from the VTP advertisement SW1 sent
earlier. Setting transparent mode was enough — domain was already set.

### Verified VTP Status on Both Switches

```
VTP Operating Mode    : Transparent    ← no longer server
VTP Domain Name       : COOKIE
Configuration Revision: 0              ← frozen at 0 in transparent mode
Number of existing VLANs: 7
```

Revision 0 and frozen — no rogue switch can overwrite this VLAN database.

### Verified Final Trunk State on Both Switches

```
! Cafe-SW1
Port   Vlans allowed and active in management domain
Et0/0  10,20
Et0/1  10,20

Port   Vlans in spanning tree forwarding state and not pruned
Et0/0  10,20
Et0/1  10,20
```

VLANs 10 and 20 now active and forwarding on all trunks — because the VLANs exist
locally on both switches in transparent mode.

---

## Mistakes Made

| Mistake | What Happened | Correction |
|---------|---------------|------------|
| `switchport mode trunk` before encapsulation | Rejected — IOSv requires `encapsulation dot1q` first | Set encapsulation before mode |
| `vtp domin COOKIE` | Typo — invalid command | `vtp domain COOKIE` |
| `exot` instead of `exit` on Cafe-SW1 | Invalid input, stayed in config mode | Typed `exit` correctly |
| `show interfaces trunck` on Cafe-SW2 | Typo — invalid command | `show interfaces trunk` |
| `shwo vlan` on Cafe-SW2 | Typo — invalid command | `show vlan` |
| Cafe-RTR1 Et0/0 was shutdown | Subinterfaces went down — ROAS broke | `no shutdown` on physical interface |

---

## State After Lab

```
Cafe-SW1:
  Et0/0  → trunk (dot1Q, nonegotiate) → Cafe-RTR1
  Et0/1  → trunk (dot1Q, nonegotiate) → Cafe-SW2
  VTP    → Transparent, domain COOKIE, revision 0
  VLANs  → 10 (ADMIN-DEV), 20 (PATRON-DEV) — locally managed

Cafe-SW2:
  Et0/1  → trunk (dot1Q, nonegotiate) → Cafe-SW1
  VTP    → Transparent, domain COOKIE, revision 0
  VLANs  → 10 (ADMIN-DEV), 20 (PATRON-DEV) — locally managed

Cafe-RTR1:
  Et0/0     → up/up (physical, no IP)
  Et0/0.10  → 10.0.18.1/27, up/up
  Et0/0.20  → 10.0.18.33/27, up/up
```

---

## Key Commands Used

```
! Harden a trunk port — three commands required on IOSv
interface ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport nonegotiate

! Verify DTP is disabled
show dtp interface ethernet 0/1
  → look for: TAS: NONEGOTIATE and all timers STOPPED

! Verify trunk status
show interfaces trunk

! Bring up a shutdown router interface
interface ethernet0/0
 no shutdown

! Set VTP transparent mode and domain
vtp domain COOKIE
vtp mode transparent

! Verify VTP state
show vtp status
  → look for: Operating Mode: Transparent, Revision: 0

! Re-create VLANs locally after VTP mode change
vlan 10
 name ADMIN-DEV
vlan 20
 name PATRON-DEV

! Verify VLANs exist after mode change
show vlan
```
