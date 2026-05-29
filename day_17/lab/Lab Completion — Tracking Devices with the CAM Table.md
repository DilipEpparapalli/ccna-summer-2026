
## Topology (reconstructed from switch output)

```
                    CoreSwitch (192.168.1.1 / Vlan10)
                   /          \
              Et0/0            Et0/1
                |                |
            Switch6          Server2 (192.168.1.111)
              MAC: aabb.cc00.0100   MAC: 5254.0090.b20e
           /         \
        Et0/1        Et0/2
          |            |
    PC7 (192.168.1.118)  PC8 (192.168.1.130)
    MAC: 5254.0038.f03b   MAC: 5254.00bd.f03a
```

**VLAN 10** across all devices.  
CDP confirmed: CoreSwitch Et0/0 ↔ Switch6 Et0/0 (only neighbor).  
Server2 terminates directly on CoreSwitch Et0/1 — never appears on Switch6.

---

## Task 0 — Convert IP to MAC on CoreSwitch

**Ping to refresh ARP**
```
CoreSwitch#ping 192.168.1.118
Success rate is 80 percent (4/5), min/avg/max = 1/1/2 ms
```

**ARP lookup**
```
CoreSwitch#show arp
Internet  192.168.1.1      -   aabb.cc80.0200   ARPA   Vlan10
Internet  192.168.1.118    0   5254.0038.f03b   ARPA   Vlan10
```

✅ **PC7 MAC confirmed: `5254.0038.f03b`**

---

## Task 1 — Locate MAC on CoreSwitch CAM Table

```
CoreSwitch#show mac address-table

Vlan    Mac Address       Type        Ports
  10    5254.0038.f03b    DYNAMIC     Et0/0   ← PC7 (via Switch6)
  10    5254.0090.b20e    DYNAMIC     Et0/1   ← Server2 (direct)
  10    5254.00bd.f03a    DYNAMIC     Et0/0   ← PC8 (via Switch6)
  10    aabb.cc00.0100    DYNAMIC     Et0/0   ← Switch6 itself
```

`5254.0038.f03b` is on **Et0/0** alongside multiple other MACs → downstream switch behind it.

**CDP confirms Switch6 is on Et0/0**
```
CoreSwitch#show cdp neighbors
Switch6    Eth 0/0    148    R S I    Linux Uni    Eth 0/0
```

→ Log into Switch6 to continue trace.

---

## Task 2 — Nail Down Access Port on Switch6

> Note: Switch6 has no IP routing configured — `ping` returned "Unrecognized host or address".  
> No ping needed here; MAC is already fresh in the table from CoreSwitch ARP refresh.

```
Switch6#show mac address-table

Vlan    Mac Address       Type        Ports
  10    5254.0038.f03b    DYNAMIC     Et0/1   ← PC7 ✅
  10    5254.0090.b20e    DYNAMIC     Et0/0   ← back toward CoreSwitch
  10    5254.00bd.f03a    DYNAMIC     Et0/2   ← PC8
  10    aabb.cc00.0200    DYNAMIC     Et0/0   ← CoreSwitch
  10    aabb.cc80.0200    DYNAMIC     Et0/0   ← CoreSwitch uplink
```

`5254.0038.f03b` on **Et0/1** — single MAC on that port → direct endpoint.

✅ **PC7 (192.168.1.118) located: Switch6 Et0/1**

---

## Task 3 — Security Alert: 192.168.1.111 (Server2)

**Back on CoreSwitch — ping + ARP**
```
CoreSwitch#ping 192.168.1.111
Success rate is 80 percent (4/5), min/avg/max = 1/1/1 ms

CoreSwitch#show arp
Internet  192.168.1.111    0   5254.0090.b20e   ARPA   Vlan10
Internet  192.168.1.118    4   5254.0038.f03b   ARPA   Vlan10
```

**CAM lookup**
```
CoreSwitch#show mac address-table
  10    5254.0090.b20e    DYNAMIC     Et0/1
```

`5254.0090.b20e` on **Et0/1** — and Et0/1 is NOT the Switch6 uplink (that's Et0/0).  
Single MAC on Et0/1 → Server2 is directly connected to CoreSwitch. No further hops needed.

✅ **Server2 (192.168.1.111) located: CoreSwitch Et0/1 — direct connection**

---

## Final Incident Report

| Device | IP | MAC | Switch | Port | Notes |
|--------|-----|-----|--------|------|-------|
| PC7 | 192.168.1.118 | 5254.0038.f03b | Switch6 | Et0/1 | 2-hop trace via CoreSwitch |
| Server2 | 192.168.1.111 | 5254.0090.b20e | CoreSwitch | Et0/1 | Direct — 1 hop only |
| PC8 | 192.168.1.130 | 5254.00bd.f03a | Switch6 | Et0/2 | Extra noise in CAM table |

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ping <IP>` | Refresh ARP table before lookup |
| `show arp` | Translate IP → MAC |
| `show mac address-table` | Find which port holds the MAC |
| `show cdp neighbors` | Identify downstream switch on a port |

---

## Key Observations

**Server2 never appeared on Switch6** — its MAC (`5254.0090.b20e`) showed up on Switch6's CAM pointing *back* toward Et0/0 (the uplink), confirming it lives on CoreSwitch directly. This is how you distinguish a device behind a switch from one connected directly: follow which direction the MAC points.

**Switch6 had no IP routing** — ping failed with "Unrecognized host or address". Layer 2 only. The ARP refresh done on CoreSwitch was enough to keep the MAC alive in Switch6's CAM table.

**PC8 was intentional noise** — its MAC (`5254.00bd.f03a`) appeared on both switches on the same path, a deliberate distraction to test whether you could isolate the right MAC from a busy table.
