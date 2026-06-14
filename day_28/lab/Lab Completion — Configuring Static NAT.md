
## Topology

```
PC1 (192.168.1.50) ─┐
                     ├── Cafe-Rtr ── ISP-Rtr (1.1.1.1 / 8.8.8.8)
PC2 (192.168.1.51) ─┘
     LAN: 192.168.1.0/24        WAN: 216.0.5.0/30
     E0/0: 192.168.1.1          E0/1: 216.0.5.2
     (ip nat inside)            (ip nat outside)
```

**NAT Mappings:**
| Inside Local | Inside Global |
|---|---|
| 192.168.1.50 | 216.0.5.20 |
| 192.168.1.51 | 216.0.5.21 |

![[ccna-summer-2026/day_28/lab/Network_Diagram.png]]
---
## Task 0 — Pre-NAT Assessment

Verified interfaces on Cafe-Rtr:
```
Interface              IP-Address      OK? Method Status     Protocol
Ethernet0/0            192.168.1.1     YES TFTP   up         up
Ethernet0/1            216.0.5.2       YES TFTP   up         up
```

Pre-NAT ping from PC1 — 100% packet loss as expected:
```
5 packets transmitted, 0 packets received, 100% packet loss
```

ISP-Rtr routing table confirmed the reason: host routes exist for `216.0.5.20/32` and `216.0.5.21/32` pointing back to `216.0.5.2`, but no route to `192.168.1.0/24`. Without NAT, the ISP has no way to return traffic to a private address — packets are dropped.

```
S        216.0.5.20/32 [1/0] via 216.0.5.2
S        216.0.5.21/32 [1/0] via 216.0.5.2
```

---

## Task 1 — Create the Static Mapping

```
Cafe-Rtr(config)# ip nat inside source static 192.168.1.50 216.0.5.20
```

Verified mapping exists before interface roles were configured:
```
show ip nat translations

Pro  Inside global    Inside local      Outside local    Outside global
---  216.0.5.20       192.168.1.50      ---              ---
```

Static entry visible immediately — but traffic still fails at this point because interfaces are not yet labeled.

---

## Task 2 — Mark Inside and Outside Interfaces

```
interface ethernet0/0
 ip nat inside

interface ethernet0/1
 ip nat outside
```

Confirmed in running config:
```
interface Ethernet0/0
 description Cafe LAN
 ip address 192.168.1.1 255.255.255.0
 ip nat inside

interface Ethernet0/1
 description WAN toward ISP-Rtr
 ip address 216.0.5.2 255.255.255.252
 ip nat outside
```

---

## Task 3 — Verify Translated Traffic

Post-NAT ping from PC1 — 0% packet loss:
```
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.796/0.870/1.000 ms
```

NAT table during active traffic — static entry persists, dynamic ICMP entry appears and ages out:
```
Pro   Inside global        Inside local        Outside local   Outside global
icmp  216.0.5.20:684       192.168.1.50:684    1.1.1.1:684     1.1.1.1:684
---   216.0.5.20           192.168.1.50        ---             ---
```

Inbound reachability confirmed from ISP-Rtr:
```
ISP-Rtr# ping 216.0.5.20
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

This confirms the mapping is truly bidirectional — the ISP router initiates a connection to the public IP and it arrives at the internal server.

---

## Task 4 — Second Static Mapping

```
Cafe-Rtr(config)# ip nat inside source static 192.168.1.51 216.0.5.21
```

PC2 ping confirmed:
```
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.773/0.922/1.430 ms
```

Both mappings active simultaneously — static NAT scales cleanly per device as long as public IPs are available.

---

## Key Observations

**Why pre-NAT traffic failed:** The ISP router had no route to `192.168.1.0/24` — private addresses are not routable on the internet. Even with a default route on Cafe-Rtr, return traffic from the ISP had nowhere to go. NAT solves this by making inside traffic appear to come from a public IP the ISP *does* have a route for.

**Interface roles are not optional:** The static mapping was visible in `show ip nat translations` before the interfaces were labeled, but no translation occurred. The router needs `ip nat inside` and `ip nat outside` to know which traffic to intercept and translate.

**Static entry vs dynamic entry:** The permanent `---` entry in the NAT table is the static mapping — always present. The `icmp` entry is a dynamic session entry that appears during active traffic and ages out. Both show up in `show ip nat translations` simultaneously.

**One public IP per device:** Static NAT consumed one public IP per inside host. PC1 uses `216.0.5.20`, PC2 uses `216.0.5.21`. This does not scale — which is exactly why Dynamic NAT and PAT exist.

---

## Commands Used

```
! Static NAT mapping
ip nat inside source static [inside-local] [inside-global]

! Interface roles
ip nat inside
ip nat outside

! Verification
show ip nat translations
show ip interface brief
show running-config interface [interface]
show ip route
```

---

## Completion Checklist
- [x] Pre-NAT behavior assessed and failure reason identified
- [x] Static mapping created for PC1 (192.168.1.50 → 216.0.5.20)
- [x] Interface roles configured (E0/0 inside, E0/1 outside)
- [x] Outbound traffic verified — PC1 reaches 1.1.1.1 with 0% loss
- [x] Inbound traffic verified — ISP-Rtr pings 216.0.5.20 successfully
- [x] Second mapping added for PC2 (192.168.1.51 → 216.0.5.21)
- [x] Configuration saved with `wr`
