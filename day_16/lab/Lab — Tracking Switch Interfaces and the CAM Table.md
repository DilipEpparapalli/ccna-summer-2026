
## Topology

![[Pasted image 20260526195847.png]]
## What I Did

### Step 1 — Enter privileged mode
```ios
Switch6> enable
Switch6#
```

### Step 2 — Check interface status
```ios
Switch6# show ip interface brief
```
Showed all 4 interfaces. Et0/3 was `administratively down` — manually disabled,
not a cable issue.

```ios
Switch6# show interfaces status
```
Showed speed, duplex, VLAN, and description per port. All active ports
negotiated `full duplex, auto speed`. Et0/3 disabled.

### Step 3 — Read interface descriptions
```ios
Switch6# show interfaces description
```

| Interface | Description |
|-----------|-------------|
| Et0/0 | Uplink-to-CoreSwitch |
| Et0/1 | AccessPoint1 |
| Et0/2 | SensorPod-A |
| Et0/3 | Reserved-StackLink |

Descriptions act as a cable map — no need to trace physical wires.

### Step 4 — Read the CAM table
```ios
Switch6# show mac address-table
```

| VLAN | MAC Address | Type | Port |
|------|-------------|------|------|
| 10 | 5254.0072.6c9e | DYNAMIC | Et0/1 |
| 10 | 5a5a.1c1c.0d0d | STATIC | Et0/1 |
| 20 | 5254.0061.3a09 | DYNAMIC | Et0/2 |
| 20 | 7c7c.b2b2.2020 | STATIC | Et0/2 |
| 99 | 40a6.b77d.aa01 | STATIC | Et0/0 |
| 99 | 40a6.b77d.bb02 | STATIC | Et0/0 |

Et0/0 shows 2 MACs on VLAN 99 → confirms it's an uplink to another switch
(CoreSwitch + OpsServer behind it).

### Step 5 — Filter by MAC address
```ios
Switch6# show mac address-table address 5a5a.1c1c.0d0d
```
Confirmed AccessPoint1's static MAC is pinned to Et0/1.

### Step 6 — Filter by interface
```ios
Switch6# show mac address-table interface Ethernet 0/1
```
Showed both MACs (dynamic + static) on Et0/1.

### Step 7 — Repeat on CoreSwitch
```ios
CoreSwitch# show mac address-table
```

| VLAN | MAC Address | Type | Port |
|------|-------------|------|------|
| 99 | 5254.0019.eb64 | DYNAMIC | Et0/1 |
| 99 | aabb.cc00.0100 | DYNAMIC | Et0/0 |

Only 2 MACs total. Et0/0 (uplink to Switch6) shows just 1 MAC — Switch6's
interface. CoreSwitch never sees AccessPoint1 or SensorPod-A directly because
Switch6 is in between. CAM tables only record MACs of directly attached devices.

## Key Takeaways

- `show ip interface brief` → quick up/down status for all interfaces
- `show interfaces status` → speed, duplex, VLAN per port
- `show interfaces description` → your cable map, no wire tracing needed
- `show mac address-table` → full CAM table
- Multiple MACs on one port = uplink to another switch
- Upstream switches don't see downstream MACs — only what's directly attached
- STATIC entries = manually configured, never age out
- DYNAMIC entries = learned on ingress, expire after 300 seconds by default
