# What is a Switch?

## Simple Explanation
A switch connects devices on a local network and forwards traffic intelligently — only to the device that actually needs it. It learns where devices are by watching traffic, and remembers using a MAC address table.

## Why It Exists
Before switches, hubs existed. A hub blasted every frame out every port — noisy, inefficient, and a security nightmare. The switch fixed this by making forwarding decisions instead of just repeating signals.

## How It Works
1. A frame arrives on a switch port
2. Switch reads the **source MAC address** → records it in the CAM table (which port that device lives on)
3. Switch reads the **destination MAC address** → looks it up in the CAM table
4. **If found:** forwards the frame to the correct port only
5. **If not found:** floods the frame out all ports in the same VLAN (unknown unicast flood)
6. The destination device replies → switch learns its MAC → no more flooding needed

```
[Device A] ──port1──┐
[Device B] ──port2──┤ SWITCH ── makes forwarding decisions
[Device C] ──port3──┘
```

## VLAN Tagging on Access vs Trunk Ports

```
[Pi - tagged VLAN 20] ──trunk──► [SWITCH] ──access port──► [End Device - untagged]

TRUNK side             │    ACCESS side
Tagged (VLAN 20)  ────►│────► Untagged (tag STRIPPED)
Tagged (VLAN 20)  ◄────│◄──── Untagged (tag ADDED)
```

- **Access port:** switch strips the VLAN tag before delivering to end device (end devices don't speak 802.1Q)
- **Trunk port:** VLAN tag is kept — allows multiple VLANs across one link
- **Flooding stays inside the VLAN** — unknown unicast flood never crosses VLAN boundaries

## Key Terms
| Term | Meaning |
|------|---------|
| CAM Table | MAC address table — maps MAC addresses to switch ports |
| Unknown Unicast Flood | Flooding a frame to all ports in the VLAN when destination MAC is unknown |
| Access Port | Switch port configured for a single VLAN — strips/adds tags |
| Trunk Port | Switch port that carries multiple VLANs — keeps tags intact |
| 802.1Q | The standard for VLAN tagging on Ethernet frames |
| Frame | Layer 2 unit of data — contains source/destination MAC addresses |

## Exam Traps
1. **Switch flooding ≠ ARP.** The *switch* floods when it doesn't know a MAC. The *end device* uses ARP when it doesn't know a MAC. Two different devices, two different jobs.
2. **Unknown unicast flood stays within the VLAN** — it does NOT cross to other VLANs
3. **Switches operate at Layer 2** — they don't read IP addresses, only MAC addresses

## Commands
```
! View the MAC address table
show mac-address-table

! View interfaces and their VLAN assignments  
show interfaces trunk
show vlan brief
```

## Recall Questions
1. What is the CAM table and what triggers an entry being added?
2. A frame arrives for a MAC the switch has never seen. What does the switch do — and does it flood everywhere?
3. Your Pi sends a VLAN-tagged frame to an access port device. What does the device actually receive?
