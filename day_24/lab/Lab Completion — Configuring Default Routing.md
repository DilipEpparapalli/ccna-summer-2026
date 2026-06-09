## Topology

```
[Cafe LAN]        [Branch WAN]       [ISP WAN]         [Simulated Internet]
192.168.1.0/24    192.168.2.0/30     216.0.5.0/30          8.8.8.8
     |                  |                 |                    |
  Cafe-RT1          Eth0/1           ISP-RT1               Loopback0
  Eth0/0: 192.168.1.1              Eth0/0: 216.0.5.1      8.8.8.8
  Eth0/1: 192.168.2.1              Eth0/1: 8.8.8.1
  Eth0/2: 216.0.5.2 ←── NEW
     |
 Cafe-PC: 192.168.1.50
```
![[ccna-summer-2026/day_24/lab/Network_Diagram_1.png]]
---

## Task 0 — Prepare the ISP Link

Ethernet0/2 on Cafe-RT1 was present but unassigned and shutdown. Configured the ISP-provided address and brought the interface up.

```
Cafe-RT1# conf t
Cafe-RT1(config)# interface ethernet0/2
Cafe-RT1(config-if)# ip address 216.0.5.2 255.255.255.252
Cafe-RT1(config-if)# no shutdown
Cafe-RT1(config-if)# exit
```

**Verified with `show ip interface brief`:**
```
Interface     IP-Address    OK? Method  Status    Protocol
Ethernet0/0   192.168.1.1   YES TFTP    up        up
Ethernet0/1   192.168.2.1   YES TFTP    up        up
Ethernet0/2   216.0.5.2     YES manual  up        up      ← new ISP link
Ethernet0/3   unassigned    YES unset   admin down down
```

**Ping to ISP next-hop to confirm link is live:**
```
Cafe-RT1# ping 216.0.5.1
!!!!!   (4/5 — first probe dropped while ARP resolved)
Success rate is 80 percent (4/5)
```
→ Interface up, ISP next-hop reachable. Ready to add default route.

---

## Task 1 — Configure the Default Route

```
Cafe-RT1# conf t
Cafe-RT1(config)# ip route 0.0.0.0 0.0.0.0 216.0.5.1
Cafe-RT1(config)# exit
```

**Verified with `show ip route`:**
```
Gateway of last resort is 216.0.5.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 216.0.5.1
C     192.168.1.0/24 is directly connected, Ethernet0/0
L     192.168.1.1/32 is directly connected, Ethernet0/0
C     192.168.2.0/30 is directly connected, Ethernet0/1
L     192.168.2.1/32 is directly connected, Ethernet0/1
S     192.168.3.0/24 [1/0] via 192.168.2.2
C     216.0.5.0/30 is directly connected, Ethernet0/2
L     216.0.5.2/32 is directly connected, Ethernet0/2
```

Key observations:
- `S*` = static route, candidate default
- "Gateway of last resort is 216.0.5.1" now populated
- Existing static route to 192.168.3.0/24 still present — specific routes coexist with default

---

## Task 2 — Validate Internet Reachability

**From Cafe-PC → 8.8.8.8 (after default route added):**
```
cisco@cafe-pc:~$ ping -c 5 8.8.8.8
64 bytes from 8.8.8.8: seq=0 ttl=254 time=1.332 ms
64 bytes from 8.8.8.8: seq=1 ttl=254 time=0.860 ms
64 bytes from 8.8.8.8: seq=2 ttl=254 time=0.825 ms
64 bytes from 8.8.8.8: seq=3 ttl=254 time=0.828 ms
64 bytes from 8.8.8.8: seq=4 ttl=254 time=0.806 ms
5 packets transmitted, 5 packets received, 0% packet loss
```
→ 5/5. Internet reachability confirmed.

**Traceroute from Cafe-PC:**
```
cisco@cafe-pc:~$ traceroute 8.8.8.8
 1  192.168.1.1 (192.168.1.1)   0.456 ms
 2  216.0.5.1   (216.0.5.1)     0.629 ms
```
→ Two hops: Cafe-RT1, then ISP-RT1. Traffic exits via default route as expected.

**Config saved:**
```
Cafe-RT1# write memory
```

---

## ISP-RT1 Reference (read-only, no config changes)

```
ISP-RT1# show ip interface brief
Interface     IP-Address   Status    Protocol
Ethernet0/0   216.0.5.1    up        up        ← WAN link to Cafe-RT1
Ethernet0/1   8.8.8.1      up        up
Loopback0     8.8.8.8      up        up        ← simulated internet target
```

---

## What Each Step Confirmed

| Step | What it proved |
|------|---------------|
| Interface config on Eth0/2 | ISP uplink can be added to an existing router without touching other interfaces |
| Ping to 216.0.5.1 (80%) | First-probe ARP drop is normal on a new interface — not a routing failure |
| `show ip route` after default route | `S*` and "Gateway of last resort" appear only after default route is configured |
| Ping 8.8.8.8 from Cafe-PC | Default route carries unknown-destination traffic to ISP successfully |
| Traceroute output | Forward path: Cafe-PC → Cafe-RT1 → ISP-RT1 → 8.8.8.8 (Loopback) |

---

## TTL Analysis

**Ping reply TTL = 254**

- 8.8.8.8 is ISP-RT1's loopback — the reply **originated on ISP-RT1**, not a distant server
- ISP-RT1 started the reply with TTL = 255 (Cisco IOS default for locally generated packets)
- One router hop back (ISP-RT1 → Cafe-RT1 → Cafe-PC) decremented TTL by 1
- 255 − 1 = **254** — confirms exactly one router hop on the return path

Compare to last lab: TTL=62 from a Linux host (default 64) crossing two routers = 64 − 2 = 62.
TTL arithmetic is a free path verification tool — no traceroute needed.

---

## Mistakes & Observations

- **PnP Agent Discovery log spam:** Entering `conf t` triggered several `%PNP` and `%SYS-5-CONFIG_P` messages mid-session. These are background Cisco plug-and-play processes, not errors — safe to ignore in a lab environment.
- **First ping to ISP (80% not 100%):** Eth0/2 was a new interface with no ARP cache entry for 216.0.5.1. The first probe triggered ARP resolution and timed out. Expected behavior on any newly configured interface.
- **8.8.8.8 is a loopback, not the real internet:** In this NetAcad topology, 8.8.8.8 is simulated on ISP-RT1's Loopback0. Real-world 8.8.8.8 (Google DNS) would involve many more hops and a much lower TTL on the reply.
