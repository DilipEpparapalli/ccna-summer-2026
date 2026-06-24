# Lab — Using Trunks to Extend VLANs (Cafe-SW1 & Cafe-SW2)

## Lab Environment
- **Platform:** Cisco Modeling Labs (CML) — IOSv Layer 2 switches
- **Devices:** Cafe-SW1, Cafe-SW2, Cafe-Admin1, Cafe-Client1
- **Topology:** Two switches connected via Et0/1 and Et0/2 (redundant uplinks);
  end devices on Et0/3 of each switch

```
[Cafe-Admin1] ─── Et0/3 ─── Cafe-SW1 ─── Et0/1 / Et0/2 ─── Cafe-SW2 ─── Et0/3 ─── [Cafe-Client1]
                              VLAN 10                trunk                  VLAN 10 → VLAN 20
```
![[ccna-summer-2026/day_35/lab/Network_diagram.png]]
## Objective
Configure trunk links between Cafe-SW1 and Cafe-SW2. Place both end devices in
VLAN 10, confirm they can ping each other across the trunk, then move Cafe-Client1
to VLAN 20 and confirm the ping dies — proving VLAN separation without routing.

---

## Task 0 — Baseline and VLAN Creation on Both Switches

Both switches started with all ports in VLAN 1 and no custom VLANs.

**Cafe-SW1:**

```
Cafe-SW1# conf t
Cafe-SW1(config)# vlan 10
Cafe-SW1(config-vlan)# name ADMIN-DEVICES
Cafe-SW1(config-vlan)# exit
Cafe-SW1(config)# vlan 20
Cafe-SW1(config-vlan)# name PATRON-DEVICES
Cafe-SW1(config-vlan)# exit
Cafe-SW1(config)# exit
```

**Cafe-SW2** (identical config):

```
Cafe-SW2# conf t
Cafe-SW2(config)# vlan 10
Cafe-SW2(config-vlan)# name ADMIN-DEVICES
Cafe-SW2(config-vlan)# exit
Cafe-SW2(config)# vlan 20
Cafe-SW2(config-vlan)# name PATRON-DEVICES
Cafe-SW2(config-vlan)# exit
Cafe-SW2(config)# exit
```

**Verified on both switches — VLANs exist but no ports assigned yet:**

```
VLAN Name                Status    Ports
1    default             active    Et0/0, Et0/1, Et0/2, Et0/3
10   ADMIN-DEVICES       active
20   PATRON-DEVICES      active
```

---

## Task 1 — Configure Trunk Ports on Both Switches

### Error Encountered — Missing Encapsulation Command (CML/IOSv)

On Cafe-SW1, the first trunk attempt failed:

```
Cafe-SW1(config)# interface ethernet0/1 - 2
                                       ^
% Invalid input detected at '^' marker.
```

Syntax issue — IOSv requires a space before and after the hyphen in interface range,
and uses `ethernet` not `fastEthernet` for this platform. Corrected to:

```
Cafe-SW1(config)# interface range ethernet 0/1-2
```

Then hit a second error immediately:

```
Cafe-SW1(config-if-range)# switchport mode trunk
Command rejected: An interface whose trunk encapsulation is "Auto" can not be
configured to "trunk" mode.
```

**What this means:** IOSv (CML) switches don't default to 802.1Q — the encapsulation
is set to "auto" and must be explicitly declared before setting trunk mode. Packet
Tracer switches skip this step because they only support 802.1Q. On real IOS, you
must set encapsulation first:

```
Cafe-SW1(config-if-range)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if-range)# switchport mode trunk
```

Applied same two-command sequence on Cafe-SW2:

```
Cafe-SW2(config)# interface range ethernet 0/1-2
Cafe-SW2(config-if-range)# switchport trunk encapsulation dot1q
Cafe-SW2(config-if-range)# switchport mode trunk
Cafe-SW2(config-if-range)# exit
```

Both switches showed the expected interface flap after trunk config:

```
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/2, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/2, changed state to up
```

### Trunk Verification — Cafe-SW1

```
Cafe-SW1# show interfaces trunk

Port     Mode   Encapsulation  Status    Native vlan
Et0/1    on     802.1q         trunking  1
Et0/2    on     802.1q         trunking  1

Port     Vlans allowed on trunk
Et0/1    1-4094
Et0/2    1-4094

Port     Vlans allowed and active in management domain
Et0/1    1,10,20
Et0/2    1,10,20

Port     Vlans in spanning tree forwarding state and not pruned
Et0/1    1,10,20
Et0/2    1,10,20
```

### Trunk Verification — Cafe-SW2

```
Port     Vlans in spanning tree forwarding state and not pruned
Et0/1    1,10,20
Et0/2    none        ← STP blocked the redundant uplink — expected behavior
```

**What this confirmed:**
- Both trunks are up and using 802.1Q
- VLANs 1, 10, and 20 are active and being carried
- Et0/2 on Cafe-SW2 shows `none` in the STP forwarding section — Spanning Tree
  blocked the redundant link to prevent a switching loop. This is correct behavior,
  not an error.
- Trunk ports no longer appear in `show vlan` — confirmed on both switches

---

## Task 2 — Place Both Devices in VLAN 10 and Test Connectivity

Assigned Et0/3 on both switches to VLAN 10:

```
! Cafe-SW1
interface ethernet0/3
 switchport mode access
 switchport access vlan 10

! Cafe-SW2
interface ethernet0/3
 switchport mode access
 switchport access vlan 10
```

**Verified via `show vlan` on Cafe-SW1:**

```
1    default        active    Et0/0
10   ADMIN-DEVICES  active    Et0/3
20   PATRON-DEVICES active
```

Configured IP addresses on the Linux end devices:

```
! Cafe-Admin1 (VLAN 10 — admin subnet 10.0.18.0/27)
sudo ifconfig eth0 10.0.18.2 netmask 255.255.255.224 up
sudo route del default 2>/dev/null
sudo route add default gw 10.0.18.1 eth0

! Cafe-Client1 (VLAN 10 — same subnet for this task)
sudo ifconfig eth0 10.0.18.3 netmask 255.255.255.224 up
sudo route del default 2>/dev/null
sudo route add default gw 10.0.18.1 eth0
```

**Ping result — same VLAN, trunk carrying the traffic:**

```
cisco@cafe-admin1:~$ ping 10.0.18.3
64 bytes from 10.0.18.3: seq=0 ttl=64 time=0.051 ms
64 bytes from 10.0.18.3: seq=1 ttl=64 time=0.064 ms
...
8 packets transmitted, 8 packets received, 0% packet loss
```

**What this confirmed:**
- VLAN 10 successfully spans Cafe-SW1 and Cafe-SW2 via the trunk
- 802.1Q tags carried the frames correctly across the inter-switch link
- Both devices are in the same broadcast domain and the same /27 subnet

---

## Task 3 — Move Cafe-Client1 to VLAN 20 and Prove Separation

Reassigned Et0/3 on Cafe-SW2 to VLAN 20 — no need to re-enter `switchport mode
access` since the port was already an access port:

```
Cafe-SW2(config)# interface ethernet0/3
Cafe-SW2(config-if)# switchport access vlan 20
```

**Verified via `show vlan` on Cafe-SW2:**

```
1    default        active    Et0/0
10   ADMIN-DEVICES  active
20   PATRON-DEVICES active    Et0/3
```

Updated Cafe-Client1 to the patron subnet:

```
sudo ifconfig eth0 10.0.18.34 netmask 255.255.255.224 up
sudo route del default 2>/dev/null
sudo route add default gw 10.0.18.33 eth0
```

**Ping result — different VLANs, no routing:**

```
cisco@cafe-admin1:~$ ping -c 4 10.0.18.34
4 packets transmitted, 0 packets received, 100% packet loss
```

**What this confirmed:**
- VLAN 10 (10.0.18.0/27) and VLAN 20 (10.0.18.32/27) are isolated broadcast domains
- No traffic crosses the VLAN boundary without a router
- The trunk is still carrying both VLANs — the separation is logical, not physical
- Moving a port to a different VLAN takes effect immediately with no other changes

---

## Mistakes Made

| Mistake | What Happened | Correction |
|---------|---------------|------------|
| `interface ethernet0/1 - 2` with spaces around hyphen | Invalid input on IOSv — range syntax doesn't allow spaces in that format | `interface range ethernet 0/1-2` |
| `switchport mode trunk` before setting encapsulation | Rejected — IOSv requires explicit `dot1q` encapsulation declaration first | Added `switchport trunk encapsulation dot1q` before `switchport mode trunk` |
| `sudo ifconig` and `sudo ipconig` typos on Cafe-Admin1 | Command not found | `sudo ifconfig` |

---

## Subnet Design Used in This Lab

| VLAN | Name | Subnet | Gateway |
|------|------|--------|---------|
| 10 | ADMIN-DEVICES | 10.0.18.0/27 | 10.0.18.1 |
| 20 | PATRON-DEVICES | 10.0.18.32/27 | 10.0.18.33 |

The original /26 (10.0.18.0/26) was split into two /27s — one per VLAN. Each VLAN
gets its own broadcast domain and its own subnet. This is the direct application of
the VLAN-to-subnet 1:1 mapping.

---

## State After Lab

```
Cafe-SW1:
  VLAN 1  (default)      → Et0/0
  VLAN 10 (ADMIN)        → Et0/3  (Cafe-Admin1: 10.0.18.2/27)
  VLAN 20 (PATRON)       → no ports
  Et0/1, Et0/2           → trunk (802.1Q, carrying VLANs 1, 10, 20)

Cafe-SW2:
  VLAN 1  (default)      → Et0/0
  VLAN 10 (ADMIN)        → no ports
  VLAN 20 (PATRON)       → Et0/3  (Cafe-Client1: 10.0.18.34/27)
  Et0/1, Et0/2           → trunk (Et0/1 forwarding, Et0/2 STP blocked)
```

Inter-VLAN routing is not configured — VLAN 10 and VLAN 20 are isolated. That
comes next.

---

## Key Commands Used

```
! Create VLANs
vlan 10
 name ADMIN-DEVICES
vlan 20
 name PATRON-DEVICES

! Configure trunk — IOSv requires encapsulation first
interface range ethernet 0/1-2
 switchport trunk encapsulation dot1q
 switchport mode trunk

! Assign access port to VLAN
interface ethernet0/3
 switchport mode access
 switchport access vlan 10

! Move access port to different VLAN (no need to re-set mode)
interface ethernet0/3
 switchport access vlan 20

! Verify trunk status
show interfaces trunk

! Verify port VLAN membership
show vlan

! Linux — set static IP on end device
sudo ifconfig eth0 10.0.18.2 netmask 255.255.255.224 up
sudo route del default 2>/dev/null
sudo route add default gw 10.0.18.1 eth0
```
