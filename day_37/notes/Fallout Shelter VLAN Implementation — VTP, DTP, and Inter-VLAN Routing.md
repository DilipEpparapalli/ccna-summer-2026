## Simple Explanation
Implementing VLANs is a network redesign, not a minor tweak. Each VLAN is a separate network requiring its own subnet, gateway, and DHCP scope. VTP automates VLAN database replication across switches, but only after trunk links exist. DTP negotiates whether a link becomes a trunk at all.

## Why It Exists
A flat /23 network puts management traffic, user devices, cameras, and guests all in the same broadcast domain with no isolation. VLANs segment traffic by purpose and risk. VTP reduces the overhead of manually creating VLANs on every switch. DTP reduces the overhead of manually configuring every inter-switch link.

## How It Works

### Step 1 — Design: One /23 Becomes Four /25s
The fallout shelter was assigned 10.0.16.0/23. A /23 gives one big flat network — not what the business needed. Borrowing one more bit splits it into four /25s, each with 126 usable hosts.

| VLAN | Purpose | Subnet |
|---|---|---|
| VLAN 10 | Management | 10.0.16.0/25 |
| VLAN 20 | Internal Users | 10.0.16.128/25 |
| VLAN 30 | Video Surveillance | 10.0.17.0/25 |
| VLAN 40 | Guest Access | 10.0.17.128/25 |

VLANs numbered by tens intentionally — leaves room to insert 11, 12, etc. without breaking the scheme.

### Step 2 — Why a Management VLAN?
A management VLAN isolates SSH, Telnet, and web admin access to network devices. If user devices can reach switch management interfaces directly, any compromised endpoint becomes a potential attack path into the infrastructure. Segmenting management traffic removes that exposure.

> Think in terms of risk, not just departments. Ask: *which systems should never be casually reachable from a user device?*

### Step 3 — VTP: VLAN Replication (Not Trunking)
VTP (VLAN Trunking Protocol) replicates the VLAN database from a VTP server to VTP clients across the network. The name is misleading — VTP does **not** create trunk links. It travels *over* them.

**The prerequisite chain:**
```
1. Trunk links must exist between switches
        ↓
2. VTP can then replicate VLAN database entries over those trunks
        ↓
3. VLANs appear on all switches without manual creation on each
```

If no trunk exists, VTP frames have no path and nothing replicates.

### Step 4 — DTP: Trunk Negotiation
DTP (Dynamic Trunking Protocol) negotiates whether a link becomes a trunk. Default mode on most Cisco switches is `dynamic auto`.

| Side A | Side B | Result |
|---|---|---|
| dynamic auto | dynamic auto | **No trunk** — neither side initiates |
| dynamic auto | dynamic desirable | **Trunk forms** — desirable sends DTP frames, auto responds |
| dynamic desirable | dynamic desirable | Trunk forms |
| trunk (hard set) | any | Trunk forms |

**Key mechanic:** `dynamic desirable` actively advertises "I want to trunk." `dynamic auto` only agrees if the other side asks first. Two auto ports sit silently waiting — nobody moves.

```
! Force a port to actively negotiate trunking
Switch(config-if)# switchport mode dynamic desirable

! Or hard-set trunk (preferred in production — no negotiation ambiguity)
Switch(config-if)# switchport mode trunk
```

### Step 5 — Router-on-a-Stick for Inter-VLAN Routing
One physical router interface handles all four VLANs using subinterfaces. Each subinterface is tied to a VLAN tag and holds the default gateway IP for that VLAN.

```
Router(config)# interface ethernet0/0
Router(config-if)# no shutdown          ← parent must be up first

Router(config)# interface ethernet0/0.10
Router(config-subif)# encapsulation dot1q 10
Router(config-subif)# ip address 10.0.16.1 255.255.255.128

Router(config)# interface ethernet0/0.20
Router(config-subif)# encapsulation dot1q 20
Router(config-subif)# ip address 10.0.16.129 255.255.255.128
! (repeat for .30 and .40)
```

The switch port facing the router **must be a trunk** — otherwise the router receives untagged traffic and subinterfaces can't distinguish VLANs.

### Step 6 — DHCP Scopes per VLAN
Each VLAN needs its own DHCP scope. Devices pull addresses automatically and land in the correct subnet.

```
Router(config)# ip dhcp pool VLAN10-MGMT
Router(dhcp-config)# network 10.0.16.0 255.255.255.128
Router(dhcp-config)# default-router 10.0.16.1
Router(dhcp-config)# dns-server 8.8.8.8
```

## Key Terms
| Term | Meaning |
|------|---------|
| VTP (VLAN Trunking Protocol) | Replicates VLAN database entries across switches over existing trunk links |
| VTP Server | Switch that originates and distributes VLAN database changes |
| VTP Client | Switch that receives and applies VLAN database changes; cannot create VLANs locally |
| VTP Transparent | Switch that forwards VTP frames but ignores them; manages its own VLAN database independently |
| DTP (Dynamic Trunking Protocol) | Negotiates trunk formation between directly connected switch ports |
| dynamic auto | Listens for DTP; will trunk only if the other side initiates |
| dynamic desirable | Actively sends DTP frames; initiates trunk negotiation |
| Management VLAN | Dedicated VLAN for switch/router admin access (SSH, SNMP, web UI) |
| Subinterface | Virtual interface on a router tied to a specific VLAN tag; used in ROAS designs |

## Real-World Connection
An enterprise campus with four functional areas (IT, operations, cameras, guest WiFi) maps directly to this design. The IT team's switches and routers are only reachable from the management VLAN — a contractor on the guest network can't even ping the switch management interface. VTP keeps the VLAN database consistent across 20 access switches without an engineer touching each one. DTP is usually disabled in mature environments in favor of hard-set trunk configs to eliminate negotiation risk.

## Exam Traps
1. **VTP does not create trunk links.** It replicates VLAN databases *over* trunks that already exist. A common trap question implies VTP is responsible for trunking — it isn't.
2. **dynamic auto + dynamic auto = no trunk.** This is the silent failure mode. The link is up, traffic flows in VLAN 1, but nothing trunks. If VLANs aren't replicating, check DTP mode on both ends.
3. **VTP deletions replicate instantly.** If someone deletes a VLAN on a VTP server, that deletion propagates to all clients. Ports assigned to the deleted VLAN go into an inactive state immediately. This is why VTP transparent mode or no-VTP is preferred in production.
4. **Router-on-a-stick requires the router-facing port to be a trunk.** Forgetting this is the most common ROAS failure — DHCP won't hand out addresses and pings between VLANs fail silently.

## Commands Reference
```
! VTP configuration
Switch(config)# vtp mode server
Switch(config)# vtp domain CASTLE-RYSEN
Switch(config)# vtp password cisco123

! DTP — force desirable (initiates trunk negotiation)
Switch(config-if)# switchport mode dynamic desirable

! DTP — disable entirely (best practice in production)
Switch(config-if)# switchport mode trunk
Switch(config-if)# switchport nonegotiate

! Verify VTP status
Switch# show vtp status

! Verify trunk and VLAN propagation
Switch# show interfaces trunk
Switch# show vlan brief
```

Look at the packet tracer file Fall_out_Shelter_design.pkt for these all changes.

## Recall Questions
1. What must exist before VTP can replicate VLAN information?
2. What is the result of two ports both set to `dynamic auto`? Why?
3. Why is a management VLAN a security best practice and not just an organizational nicety?
4. Why is VTP considered dangerous in production environments?
5. What happens on a router-on-a-stick if the router-facing switch port is not configured as a trunk?
6. A VLAN is deleted on the VTP server. What happens to ports assigned to that VLAN on client switches?
