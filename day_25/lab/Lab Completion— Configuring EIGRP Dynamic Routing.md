## Topology

```
[Cafe-PC]                                        [Fallout-SRV]
192.168.1.50                                     192.168.3.100
     |                                                 |
[Cafe-RT1]──────── WAN 192.168.2.0/30 ────────[Fallout-RT1]
 E0/0: 192.168.1.1                              E0/0: 192.168.3.1
 E0/1: 192.168.2.1                              E0/1: 192.168.2.2
 E0/2: 216.0.5.2 (ISP — EIGRP excluded)
```
![[ccna-summer-2026/day_25/lab/Network_Diagram.png]]
---

## Objective

Replace manually configured static routes between Cafe-RT1 and Fallout-RT1 with EIGRP (AS 1) so routers discover each other's LANs dynamically and end-to-end connectivity is restored without any hand-typed route entries.

---

## Baseline Verification

### Cafe-RT1 — interfaces confirmed up

```
Interface              IP-Address      OK? Method Status    Protocol
Ethernet0/0            192.168.1.1     YES TFTP   up        up
Ethernet0/1            192.168.2.1     YES TFTP   up        up
Ethernet0/2            216.0.5.2       YES TFTP   up        up
Ethernet0/3            unassigned      YES unset  admin down down
```

### Cafe-RT1 — routing table before any changes

Static route to Fallout LAN confirmed present:
```
S*    0.0.0.0/0 [1/0] via 216.0.5.1
S     192.168.3.0/24 [1/0] via 192.168.2.2
C     192.168.1.0/24 — directly connected, Ethernet0/0
C     192.168.2.0/30 — directly connected, Ethernet0/1
C     216.0.5.0/30  — directly connected, Ethernet0/2
```

### Fallout-RT1 — routing table before any changes

Static route to Cafe LAN confirmed present:
```
S*    0.0.0.0/0 [1/0] via 192.168.2.1
S     192.168.1.0/24 [1/0] via 192.168.2.1
C     192.168.2.0/30 — directly connected, Ethernet0/1
C     192.168.3.0/24 — directly connected, Ethernet0/0
```

---

## Task 0 — Remove Static Routes

### Cafe-RT1

```
Cafe-RT1# conf t
Cafe-RT1(config)# no ip route 192.168.3.0 255.255.255.0
Cafe-RT1(config)# exit
Cafe-RT1# show ip route
```

**Result:** `S 192.168.3.0/24` entry gone. Only connected routes and default route remain.

### Fallout-RT1

```
Fallout-RT1# conf t
Fallout-RT1(config)# no ip route 192.168.1.0 255.255.255.0
Fallout-RT1(config)# exit
Fallout-RT1# show ip route
```

**Result:** `S 192.168.1.0/24` entry gone. Default route (`S*`) left in place — intentional.

> At this point, end-to-end connectivity is broken. The routers have no path to each other's LANs. This is the expected failure state before EIGRP takes over.

---

## Task 1 — Enable EIGRP on Cafe-RT1

```
Cafe-RT1# conf t
Cafe-RT1(config)# router eigrp 1
Cafe-RT1(config-router)# network 192.168.1.0
Cafe-RT1(config-router)# network 192.168.2.0
Cafe-RT1(config-router)# exit
```

**Note on wildcard masks:** The lab guide used explicit wildcards (`0.0.0.255` for the /24 LAN, `0.0.0.3` for the /30 WAN). The commands above omit them, causing IOS to apply classful defaults (`0.0.0.255` for both Class C networks). This worked here because no unintended interfaces exist in the `192.168.2.x` range. In production, always use explicit wildcards to scope EIGRP precisely — especially on WAN links shared with ISPs or partners.

**E0/2 (216.0.5.2 — ISP-facing) intentionally excluded.** No `network 216.0.5.0` statement added. Internal routes must not be advertised toward the ISP.

Cafe-RT1 sent hellos on E0/1 (WAN link) and waited for a neighbor. No adjacency formed yet — Fallout-RT1 not configured.

---

## Task 2 — Enable EIGRP on Fallout-RT1

```
Fallout-RT1# conf t
Fallout-RT1(config)# router eigrp 1
Fallout-RT1(config-router)# network 192.168.3.0
Fallout-RT1(config-router)# network 192.168.2.0
Fallout-RT1(config-router)# end
```

**Adjacency formed immediately — actual log message:**

```
%DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.2.1 (Ethernet0/1) is up: new adjacency
```

Same message appeared on Cafe-RT1:
```
%DUAL-5-NBRCHANGE: EIGRP-IPv4 1: Neighbor 192.168.2.2 (Ethernet0/1) is up: new adjacency
```

Both routers exchanged hello packets, matched AS number 1, and formed a neighbor relationship. Route exchange followed immediately.

---

## Task 3 — Verify Dynamic Routing

### Cafe-RT1 routing table after EIGRP

```
S*    0.0.0.0/0 [1/0] via 216.0.5.1
C     192.168.1.0/24 is directly connected, Ethernet0/0
L     192.168.1.1/32 is directly connected, Ethernet0/0
C     192.168.2.0/30 is directly connected, Ethernet0/1
L     192.168.2.1/32 is directly connected, Ethernet0/1
D     192.168.3.0/24 [90/307200] via 192.168.2.2, 00:00:28, Ethernet0/1
C     216.0.5.0/30 is directly connected, Ethernet0/2
```

`D 192.168.3.0/24` confirmed — learned via EIGRP, not manually entered.

### Fallout-RT1 routing table after EIGRP

```
S*    0.0.0.0/0 [1/0] via 192.168.2.1
D     192.168.1.0/24 [90/307200] via 192.168.2.1, 00:00:41, Ethernet0/1
C     192.168.2.0/30 is directly connected, Ethernet0/1
C     192.168.3.0/24 is directly connected, Ethernet0/0
```

`D 192.168.1.0/24` confirmed — learned via EIGRP.

### EIGRP metric decoded

`[90/307200]` — two values:
- `90` = administrative distance for EIGRP (how trustworthy the source is)
- `307200` = EIGRP metric (composite value based on bandwidth and delay of the path)

### End-to-end ping — Cafe-PC → Fallout-SRV

```
cisco@cafe-pc:~$ ping 192.168.3.100
64 bytes from 192.168.3.100: seq=0 ttl=62 time=2.099 ms
...
18 packets transmitted, 18 received, 0% packet loss
```

### End-to-end ping — Fallout-SRV → Cafe-PC

```
cisco@fallout-srv:~$ ping 192.168.1.50
64 bytes from 192.168.1.50: seq=0 ttl=62 time=0.794 ms
...
10 packets transmitted, 10 received, 0% packet loss
```

**TTL=62 on both pings** — default Linux TTL is 64, two router hops consumed two TTL decrements. Confirms traffic is traversing Cafe-RT1 and Fallout-RT1 as expected.

---

## Configurations Saved

```
Cafe-RT1# write memory
Fallout-RT1# write memory
```

---

## What Each Step Confirmed

| Step | What it proved |
|------|---------------|
| Baseline `show ip route` | Static routes were present and working before EIGRP |
| Removed static routes | Network broke as expected — no path to remote LAN |
| Added `router eigrp 1` + `network` statements | EIGRP activated on correct interfaces |
| `%DUAL-5-NBRCHANGE` log message | Adjacency formed the moment both routers were configured |
| `D` route in routing table | Routers exchanged LAN routes dynamically — no manual entry |
| TTL=62 on end-to-end pings | Two hops traversed, path confirmed correct |
| 0% packet loss both directions | Full two-way connectivity restored via dynamic routing |

---

## Wildcard Mask Note (Lesson Learned)

During this lab, `network` statements were entered without explicit wildcard masks. IOS applied classful defaults and the lab succeeded. However, this is imprecise — on a router with multiple WAN links in the same classful range, the classful default would activate EIGRP on unintended interfaces.

**Best practice:** always use explicit wildcard masks.

```
! Precise — activates EIGRP only on the /24 LAN interface
network 192.168.1.0 0.0.0.255

! Precise — activates EIGRP only on the /30 WAN interface
network 192.168.2.0 0.0.0.3
```

Wildcard mask quick reference:
- `0.0.0.255` = matches any /24 network (last octet ignored)
- `0.0.0.3`   = matches only a /30 (last two bits ignored)
- Derived by subtracting subnet mask from 255.255.255.255

---

## Mistakes Made

None during this lab — commands executed correctly on first attempt. Static route removal used the full syntax (`no ip route <network> <mask>`) without needing to specify the next hop, which IOS accepted since only one matching entry existed.
