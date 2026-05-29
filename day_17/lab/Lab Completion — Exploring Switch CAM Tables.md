
## Topology
![[ccna-summer-2026/day_17/lab/Topology.png]]

**VLAN 10** → PC1 (Et0/1), PC2 (Et0/2)  
**VLAN 99** → CoreSwitch uplink (Et0/0)

---

## Task 0 — Verify Endpoint Addressing

### PC1
```
cisco@pc1:~$ ifconfig eth0
eth0  HWaddr 52:54:00:0B:BF:21
      inet addr:192.168.1.50  Mask:255.255.255.0

cisco@pc1:~$ route -n
0.0.0.0   192.168.1.1   0.0.0.0   UG   0   0   0   eth0
```
✅ PC1 confirmed: `192.168.1.50/24`, gateway `192.168.1.1`, MAC `5254.000b.bf21`

### PC2
```
cisco@pc2:~$ ifconfig eth0
eth0  HWaddr 52:54:00:4C:AD:B6
      inet addr:192.168.1.51  Mask:255.255.255.0

cisco@pc2:~$ route -n
0.0.0.0   192.168.1.1   0.0.0.0   UG   0   0   0   eth0
```
✅ PC2 confirmed: `192.168.1.51/24`, gateway `192.168.1.1`, MAC `5254.004c.adb6`

---

## Task 1 — Generate ARP Traffic to Populate the CAM Table

### Ping from PC1 → PC2 (triggers ARP)
```
cisco@pc1:~$ ping 192.168.1.51
64 bytes from 192.168.1.51: seq=0 ttl=64 time=1.354 ms
64 bytes from 192.168.1.51: seq=1 ttl=64 time=0.819 ms
64 bytes from 192.168.1.51: seq=2 ttl=64 time=0.843 ms
64 bytes from 192.168.1.51: seq=3 ttl=64 time=0.862 ms
4 packets transmitted, 4 received, 0% packet loss
```

### ARP cache confirmed on PC1
```
cisco@pc1:~$ arp -a
? (192.168.1.51) at 52:54:00:4c:ad:b6 [ether]  on eth0
```
✅ PC1 resolved PC2's MAC via ARP before the first ICMP echo.

### ARP cache confirmed on PC2
```
cisco@pc2:~$ arp -v
? (192.168.1.50) at 52:54:00:0b:bf:21 [ether]  on eth0
```
✅ PC2 also learned PC1's MAC from the ARP request it received.

---

## Task 2 — Inspect the Switch CAM Table

```
Switch6#show mac address-table

Vlan    Mac Address       Type        Ports
  10    5254.000b.bf21    DYNAMIC     Et0/1   ← PC1
  10    5254.004c.adb6    DYNAMIC     Et0/2   ← PC2
  10    5a5a.1c1c.0d0d    STATIC      Et0/1
  10    7c7c.b2b2.2020    STATIC      Et0/2
  99    40a6.b77d.aa01    STATIC      Et0/0   ← CoreSwitch
  99    40a6.b77d.bb02    STATIC      Et0/0   ← CoreSwitch
```

### Filtered view — by interface
```
Switch6#show mac address-table interface ethernet 0/1

Vlan    Mac Address       Type        Ports
  10    5254.000b.bf21    DYNAMIC     Et0/1
  10    5a5a.1c1c.0d0d    STATIC      Et0/1
```

### Filtered view — static entries only
```
Switch6#show mac address-table static

Vlan    Mac Address       Type        Ports
  10    5a5a.1c1c.0d0d    STATIC      Et0/1
  10    7c7c.b2b2.2020    STATIC      Et0/2
  99    40a6.b77d.aa01    STATIC      Et0/0
  99    40a6.b77d.bb02    STATIC      Et0/0
```
✅ Dynamic entries for PC1 and PC2 confirmed on correct ports. Static entries on uplink and access ports also present.

---

## Task 3 — Flush and Re-Learn

### Clear all dynamic entries
```
Switch6#clear mac address-table dynamic
```

### Clear dynamic entries scoped to VLAN 10 only
```
Switch6#clear mac address-table dynamic vlan 10
```

### CAM table after clearing VLAN 10 dynamics — only statics remain
```
Switch6#show mac address-table

Vlan    Mac Address       Type        Ports
  10    5a5a.1c1c.0d0d    STATIC      Et0/1
  10    7c7c.b2b2.2020    STATIC      Et0/2
  99    40a6.b77d.aa01    STATIC      Et0/0
  99    40a6.b77d.bb02    STATIC      Et0/0
```

### Second ping from PC1 — forces re-learn
```
cisco@pc1:~$ ping 192.168.1.51
6 packets transmitted, 6 received, 0% packet loss
min/avg/max = 0.771/0.956/1.526 ms
```

### CAM table after re-learn — dynamic entries back
```
Switch6#show mac address-table

Vlan    Mac Address       Type        Ports
  10    5254.000b.bf21    DYNAMIC     Et0/1   ← PC1 re-learned
  10    5254.004c.adb6    DYNAMIC     Et0/2   ← PC2 re-learned
  10    5a5a.1c1c.0d0d    STATIC      Et0/1
  10    7c7c.b2b2.2020    STATIC      Et0/2
  99    40a6.b77d.aa01    STATIC      Et0/0
  99    40a6.b77d.bb02    STATIC      Et0/0
```
✅ Dynamic entries rebuilt automatically after traffic. Static entries untouched throughout.

---

## Commands Used

| Command | What it does |
|---------|-------------|
| `show mac address-table` | Full CAM table |
| `show mac address-table static` | Static entries only |
| `show mac address-table interface ethernet 0/1` | Entries on one port |
| `clear mac address-table dynamic` | Wipe all dynamic entries |
| `clear mac address-table dynamic vlan 10` | Wipe dynamics for one VLAN only |
| `ifconfig eth0` | Linux: show IP and MAC |
| `route -n` | Linux: show routing table / default gateway |
| `ping 192.168.1.51` | Generate traffic to trigger ARP + CAM learning |
| `arp -a` | Linux: show resolved ARP cache |

---

## What This Lab Proved

- The switch builds its CAM table automatically from **source MACs** — no config needed
- ARP is what generates the initial traffic that seeds the table
- `clear mac address-table dynamic` only removes dynamic entries — static entries survive
- Scoped clearing (`vlan 10`) is possible — VLAN 99 entries were untouched
- Dynamic entries rebuild the moment fresh traffic hits the switch
