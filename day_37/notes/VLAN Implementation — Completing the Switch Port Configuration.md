
## Simple Explanation
Creating VLANs on a switch is only half the job. Every switch port must be explicitly assigned to the right VLAN and the right mode — access or trunk — or traffic won't flow the way the design intends. A VLAN is a traffic boundary; devices only participate in it when their switch port is actually a member.

## Why It Exists
You can have a perfect logical VLAN design with correct subnets, subinterfaces, and DHCP scopes — and still have a broken network. If the switch ports aren't assigned correctly, Layer 2 doesn't match Layer 3, and traffic goes nowhere.

## How It Works

### Access Port vs Trunk Port
| Port Type | Carries | Used For |
|---|---|---|
| Access | One VLAN only | PCs, servers, printers, IP phones |
| Trunk | Multiple VLANs (802.1Q tagged) | Switch-to-switch links, WAP uplinks, router subinterface uplinks |

The decision is simple: **one VLAN → access. Multiple VLANs → trunk.**

### Why the WAP Needs a Trunk Port
A Wireless Access Point hosting multiple SSIDs (e.g., one for staff, one for guests) carries traffic for multiple VLANs simultaneously. The WAP tags frames per SSID so the switch knows which VLAN each wireless client belongs to. That tagging requirement makes the uplink a trunk.

```
[Staff Laptop]  → SSID: Staff  → WAP tags VLAN 10 → Trunk port → Switch
[Guest Phone]   → SSID: Guest  → WAP tags VLAN 20 → Trunk port → Switch
```

### The VLAN 1 Trap — Layer 2 Must Match Layer 3
When router-on-a-stick is configured, the router's physical interface loses its IP. All routing moves to subinterfaces (one per VLAN). VLAN 1 gets no subinterface → no router path → anything still in VLAN 1 is stranded.

```
Before fix:
[Plex Server] → Switch port (VLAN 1, default) → NO ROUTER PATH → dead

After fix:
[Plex Server] → Switch port (access, VLAN 10) → subinterface g0/0.10 → routed
```
A correct IP address means nothing if the Layer 2 VLAN membership is wrong. Both layers must align.

### Unused Port Security — Three Steps
Unused switch ports are an open door. Best practice:
1. Force to **access mode** — prevents trunk negotiation if someone plugs in a switch
2. Assign to a **dead/unused VLAN** — isolates any rogue device from production traffic
3. **Shut down** — port is administratively down until explicitly needed

```
Switch(config)# interface range fa0/10 - 24
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 999
Switch(config-if-range)# shutdown
```

## Key Terms
| Term | Meaning |
|------|---------|
| Access Port | Switch port carrying traffic for exactly one VLAN; endpoint-facing |
| Trunk Port | Switch port carrying tagged traffic for multiple VLANs |
| 802.1Q | The tagging standard trunks use to identify which VLAN a frame belongs to |
| SSID | The name of a wireless network; each SSID maps to a VLAN on the WAP |
| Dead VLAN | An unused VLAN used to park inactive switch ports; no devices or routing exist in it |
| Router-on-a-Stick | Single physical router interface running multiple subinterfaces, one per VLAN |

## Real-World Connection
In an enterprise office, a WAP on a conference room wall serves the corporate SSID (VLAN 10) and a guest SSID (VLAN 99). Its uplink to the access switch is a trunk. Every PC port in the room is an access port in VLAN 10. Every unused port is shut down and dumped in VLAN 999. An auditor walking in would find no open attack surface on the switch layer — that's the goal.

## Exam Traps
1. **IP address correct, still no connectivity** — always check Layer 2 VLAN membership first. The switch doesn't care about your IP until the frame reaches the right VLAN.
2. **WAP uplinks are trunks, not access ports** — a WAP looks like an endpoint physically but behaves like a switch logically. Multi-SSID = multi-VLAN = trunk.
3. **Unused ports left in default mode can negotiate trunking** — if DTP is active and someone plugs in another switch, you've accidentally opened a trunk. Force access mode on all unused ports.

## Commands
```
! Assign an access port to a VLAN
Switch(config)# interface fa0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10

! Configure a trunk port (IOSv requires encapsulation first)
Switch(config)# interface g0/1
Switch(config-if)# switchport trunk encapsulation dot1q
Switch(config-if)# switchport mode trunk

! Lock down unused ports
Switch(config)# interface range fa0/10 - 24
Switch(config-if-range)# switchport mode access
Switch(config-if-range)# switchport access vlan 999
Switch(config-if-range)# shutdown

! Verify VLAN assignments
Switch# show vlan brief

! Verify trunk status
Switch# show interfaces trunk
```

## Recall Questions
1. A server has the correct IP and default gateway but can't reach anything. What Layer 2 issue should you check first?
2. Why does a WAP uplink need to be a trunk port?
3. What three things should you do to every unused switch port?
4. Why isn't forcing an unused port to access mode enough on its own — why also assign it to a dead VLAN?
5. What happens to VLAN 1 connectivity when you move to router-on-a-stick and don't create a subinterface for it?
