## Simple Explanation
A switch learns which devices live on which ports by reading the source MAC
address of every incoming frame. It stores this in a CAM table. When it needs
to forward a frame, it looks up the destination MAC — if found, it sends it
to one port; if not, it floods all ports except the one it came in on.

## Why It Exists
Without the CAM table, a switch would have to flood every frame to every port,
every time — making it no better than a hub. The CAM table is what makes
switching intelligent and efficient.

## How It Works

1. Frame arrives on a port
2. Switch reads **source MAC** → records it in CAM table with that port number
3. Switch checks **destination MAC** in CAM table
   - **Found** → forward out that specific port only (unicast)
   - **Not found** → flood all ports except the source port (Unknown Unicast Flood)
4. The destination device replies → switch learns *its* MAC on ingress too
5. Subsequent frames between these two devices are forwarded directly, no flooding

```
Frame arrives on Gi0/3
        │
        ▼
[ Read SOURCE MAC ]──────────────────────────────► Record in CAM table
        │                                           (Gi0/3 → AA:BB:CC:DD:EE:FF)
        ▼
[ Check DESTINATION MAC in CAM table ]
        │
        ├── FOUND ──────────────────────────────► Forward to that port only
        │
        └── NOT FOUND ──────────────────────────► Flood all ports except Gi0/3
```

## Key Terms

| Term | Meaning |
|------|---------|
| CAM Table | Content Addressable Memory — maps MAC addresses to switch ports |
| Unknown Unicast Flood | Flooding behavior when destination MAC isn't in CAM table |
| Ingress | Traffic arriving *into* the switch port |
| Egress | Traffic leaving *out of* the switch port |
| MAC Learning | Recording source MAC + port number on frame arrival |
| Modular Switch | Chassis-based switch with swappable line card modules |
| Switch Stack | Multiple physical switches managed as one logical unit |

## Port Naming Convention

```
FastEthernet 0/1     → Fa0/1    (10/100 Mbps)
GigabitEthernet 0/1  → Gi0/1   (1000 Mbps)
TenGigabitEthernet   → Te0/1   (10 Gbps — uplinks)

Format: [Type] [module/port]
```

- **Module number** comes from the physical chassis slot on a modular switch,
  or the switch unit number in a stack
- **Port number** is the physical interface on that module
- Even small fixed switches use this format because the software supports
  stacking and modularity at scale

## Real-World Connection

If you see 30 MAC addresses learned on a single interface (e.g., `Gi0/1`),
that port is almost certainly an uplink to another switch — not 30 devices
on one cable. The correct move:

1. Note the uplink port
2. SSH to the next switch
3. Run `show mac address-table` there
4. Repeat until you find the port with a **single MAC** — that's your end device

This is standard troubleshooting in any enterprise environment with a
multi-tier switching topology (access → distribution → core).

## Exam Traps

1. **Learning vs. Flooding are separate events** — Learning happens on
   **ingress** (source MAC). Flooding happens when the **destination** is
   unknown. Don't conflate them — different triggers, different jobs.

2. **Unknown Unicast Flood ≠ Broadcast** — An unknown unicast flood is the
   switch saying *"I don't know where this goes."* A broadcast is intentional
   (destination MAC = `FF:FF:FF:FF:FF:FF`). Different causes, same-looking
   behavior on the wire.

3. **Many MACs on one port ≠ a problem** — It means a downstream switch
   is behind that port. Only flag it if you expected a single end device.

## Commands

```ios
show mac address-table          ! View all learned MAC-to-port mappings
show ip interface brief         ! Fast interface inventory and up/down status
show interface status           ! Detailed: speed, duplex, VLAN, status
```

## Recall Questions

1. A frame arrives with a source MAC the switch has never seen. What does
   the switch do with that MAC, and exactly when does it do it?
2. What is the difference between an Unknown Unicast Flood and a broadcast?
3. You see 45 MAC addresses on `GigabitEthernet0/2`. What does that tell
   you, and what is your next troubleshooting step?
