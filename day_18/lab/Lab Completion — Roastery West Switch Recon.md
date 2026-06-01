
**Environment:** Cisco CML — Castle Rysen topology  
**Switches:** RW-CORE-SW (IOS-XE), RW-ACC-SW (IOS-XE)  
**VLAN in use:** VLAN 42 (192.168.42.0/24)

---
## Task 1 — Establish the Core View

### Commands run

```
RW-CORE-SW# show interfaces status
RW-CORE-SW# show ip interface brief
RW-CORE-SW# show mac address-table
RW-CORE-SW# show arp
RW-CORE-SW# show cdp neighbors
```

### What I observed

`show interfaces status` revealed the port roles immediately:

```
Port    Status      Vlan    Duplex  Speed
Et0/0   connected   trunk   full    auto
Et0/1   connected   42      full    auto
Et0/2   disabled    1       full    auto
Et0/3   disabled    1       full    auto
```

- **Et0/0** — trunk port connecting to RW-ACC-SW. Shows "trunk" instead of a VLAN number because it carries frames for multiple VLANs simultaneously, tagging each with an 802.1Q header.
- **Et0/1** — access port assigned to VLAN 42. Shows "42" because it belongs to exactly one VLAN.
- **Et0/2 / Et0/3** — administratively shut down.

`show ip interface brief` confirmed management addressing:

```
Interface    IP-Address      Status    Protocol
Ethernet0/0  unassigned      up        up
Ethernet0/1  unassigned      up        up
Vlan42       192.168.42.1    up        up
```

The SVI (Vlan42) holds the management IP — Layer 2 ports themselves carry no IP address.

`show cdp neighbors` confirmed the uplink peer:

```
Device ID   Local Intrfce   Capability   Port ID
RW-ACC-SW   Eth 0/0         R S I        Eth 0/0
```

Et0/0 on the core connects directly to Et0/0 on the access switch.

### Descriptions applied

```
RW-CORE-SW(config)# interface ethernet 0/0
RW-CORE-SW(config-if)# description "coreswitch-uplink-to-RW-ACC-SW"
```

---

## Task 2 — Discover the Access Layer

### Commands run

```
RW-ACC-SW# show interfaces status
RW-ACC-SW# show ip interface brief
RW-ACC-SW# show cdp neighbors
RW-ACC-SW# show mac address-table
```

### What I observed

`show cdp neighbors` confirmed the uplink back to core:

```
Device ID    Local Intrfce   Capability   Port ID
RW-CORE-SW   Eth 0/0         R S I        Eth 0/0
```

Et0/0 on the access switch connects to Et0/0 on the core switch — trunk link confirmed on both ends.

Management SVI on access switch:

```
Interface    IP-Address      Status    Protocol
Vlan42       192.168.42.2    up        up
```

### Descriptions applied

```
RW-ACC-SW(config)# interface ethernet 0/0
RW-ACC-SW(config-if)# description "uplink-to-coreswitch"
```

---

## Task 3 — Trace Each Endpoint

### Method

The ARP table on each switch only populates after traffic is exchanged. I pinged each workstation IP from the switch to force ARP resolution, then cross-referenced the ARP table (IP → MAC) with the MAC address table (MAC → port).

```
RW-ACC-SW# ping 192.168.42.37
RW-ACC-SW# ping 192.168.42.38
RW-ACC-SW# ping 192.168.42.39
RW-ACC-SW# ping 192.168.42.1
RW-ACC-SW# show arp
RW-ACC-SW# show mac address-table
```

### ARP table (after pings)

```
Protocol  Address         Hardware Addr   Interface
Internet  192.168.42.1    aabb.cc80.0200  Vlan42
Internet  192.168.42.2    aabb.cc80.0100  Vlan42
Internet  192.168.42.37   5254.0061.a088  Vlan42
Internet  192.168.42.38   5254.00fb.3037  Vlan42
Internet  192.168.42.39   5254.0039.96c0  Vlan42
```

### MAC address table — RW-ACC-SW

```
Vlan   Mac Address       Type      Ports
42     5254.0061.a088    DYNAMIC   Et0/0   ← seen via trunk (lives on core side)
42     5254.00fb.3037    DYNAMIC   Et0/1
42     5254.0039.96c0    DYNAMIC   Et0/2
42     aabb.cc80.0200    DYNAMIC   Et0/0   ← core switch SVI MAC, seen via trunk
```

### MAC address table — RW-CORE-SW (after second refresh)

```
Vlan   Mac Address       Type      Ports
42     5254.0039.96c0    DYNAMIC   Et0/0   ← seen via trunk (lives on access side)
42     5254.0061.a088    DYNAMIC   Et0/1
42     5254.00fb.3037    DYNAMIC   Et0/0   ← seen via trunk (lives on access side)
42     aabb.cc00.0100    DYNAMIC   Et0/0
42     aabb.cc80.0100    DYNAMIC   Et0/0
```

### Endpoint mapping (final)

| IP | MAC | Switch | Port | Device |
|----|-----|--------|------|--------|
| 192.168.42.37 | 5254.0061.a088 | RW-CORE-SW | Et0/1 | BaristaPOS |
| 192.168.42.38 | 5254.00fb.3037 | RW-ACC-SW  | Et0/1 | InventoryStation |
| 192.168.42.39 | 5254.0039.96c0 | RW-ACC-SW  | Et0/2 | ManagerConsole |

### Descriptions applied

```
RW-CORE-SW(config)# interface ethernet 0/1
RW-CORE-SW(config-if)# description "coreswitch-to-BaristaPOS"

RW-ACC-SW(config)# interface ethernet 0/1
RW-ACC-SW(config-if)# description "accessswitch-to-InventoryStation"

RW-ACC-SW(config)# interface ethernet 0/2
RW-ACC-SW(config-if)# description "accessswitch-to-ManagerConsole"
```

---

## Task 4 — Validate Continuity

### What I verified

- Re-ran `show mac address-table` on both switches — all three workstation MACs present and mapped to the correct ports.
- Trunk link Et0/0 ↔ Et0/0 remained up/up throughout; VLAN 42 traffic flowing normally.
- No error-disabled ports observed. No VLAN mismatches detected.
- All three workstation pings succeeded (80–100% success rate — normal for first ICMP on a freshly resolved ARP entry).

### Anomalies

None. All endpoints reachable, trunk healthy, MAC table consistent across both switches.

---

## Key Observations from This Lab

**Ping before ARP lookup.** The ARP table only populates after traffic flows. If you run `show arp` on a cold switch, it returns nothing useful. Ping the target first to force the exchange.

**Trunk port shows "trunk" not a VLAN number.** An access port belongs to one VLAN — the switch can display that VLAN number. A trunk port carries frames for many VLANs simultaneously, tagging each frame with an 802.1Q header that identifies VLAN membership. There's no single VLAN to display, so IOS shows "trunk."

**The 802.1Q tag identifies VLAN membership, not origin.** The 4-byte tag stamped on frames crossing the trunk tells the receiving switch which VLAN the frame belongs to — not where it came from. Source MAC handles origin; the tag handles VLAN classification.

**Upstream switches don't see downstream MACs directly.** RW-CORE-SW sees InventoryStation and ManagerConsole MACs on Et0/0 (the trunk) — not on dedicated access ports. Those devices live behind the access switch; the core only sees them through the uplink. This is topology, not VLAN isolation.

**MAC table grows after second refresh.** The first `show mac address-table` on the core may miss entries that haven't been learned yet. After pinging from the access switch and re-running, additional MACs appear. Timing matters when reading these tables live.

---

## Management Summary (for senior engineer handoff)

| Device | Management IP | Gateway | SVI | Reachability |
|--------|--------------|---------|-----|--------------|
| RW-CORE-SW | 192.168.42.1 | — | Vlan42 | Up |
| RW-ACC-SW  | 192.168.42.2 | 192.168.42.1 | Vlan42 | Up (pinged from acc-sw) |
