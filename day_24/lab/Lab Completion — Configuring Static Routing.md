
## Topology

```
[192.168.1.0/24]          [192.168.2.0/30]          [192.168.3.0/24]
  Cafe LAN                   WAN Link                  Fallout LAN
     |                          |                           |
  Cafe-RT1                  .1 --- .2               Fallout-RT1
  Eth0/0: 192.168.1.1    Eth0/1    Eth0/1            Eth0/0: 192.168.3.1
  Eth0/1: 192.168.2.1                                Eth0/1: 192.168.2.2
     |                                                    |
 Host: 192.168.1.50                            Server: 192.168.3.100
```
![[ccna-summer-2026/day_24/lab/Network_Diagram.png]]
---

## Task 0 — Baseline Verification

Confirmed both routers had their WAN and LAN interfaces up before touching any routing config.

**Cafe-RT1:**
```
Cafe-RT1# show ip interface brief
Interface     IP-Address    OK? Method  Status    Protocol
Ethernet0/0   192.168.1.1   YES TFTP    up        up
Ethernet0/1   192.168.2.1   YES TFTP    up        up
Ethernet0/2   unassigned    YES unset   admin down down
Ethernet0/3   unassigned    YES unset   admin down down
```

**Cafe-RT1 routing table (before static routes):**
```
C  192.168.1.0/24  is directly connected, Ethernet0/0
L  192.168.1.1/32  is directly connected, Ethernet0/0
C  192.168.2.0/30  is directly connected, Ethernet0/1
L  192.168.2.1/32  is directly connected, Ethernet0/1
```
→ Only connected routes present. No knowledge of 192.168.3.0/24 yet.

**Fallout-RT1:**
```
Fallout-RT1# show ip interface brief
Interface     IP-Address    OK? Method  Status    Protocol
Ethernet0/0   192.168.3.1   YES TFTP    up        up
Ethernet0/1   192.168.2.2   YES TFTP    up        up
Ethernet0/2   unassigned    YES unset   admin down down
Ethernet0/3   unassigned    YES unset   admin down down
```

**Fallout-RT1 routing table (before static routes):**
```
C  192.168.2.0/30  is directly connected, Ethernet0/1
L  192.168.2.2/32  is directly connected, Ethernet0/1
C  192.168.3.0/24  is directly connected, Ethernet0/0
L  192.168.3.1/32  is directly connected, Ethernet0/0
```
→ Only connected routes present. No knowledge of 192.168.1.0/24 yet.

**What this confirmed:** Each router knew its own networks automatically. Neither knew the other's LAN. Static routes required on both sides.

---

## Task 1 — Static Route on Cafe-RT1

Told Cafe-RT1 how to reach the Fallout LAN (192.168.3.0/24) via the WAN next-hop (192.168.2.2).

```
Cafe-RT1# conf t
Cafe-RT1(config)# ip route 192.168.3.0 255.255.255.0 192.168.2.2
Cafe-RT1(config)# end
```

**Verified in routing table:**
```
S  192.168.3.0/24 [1/0] via 192.168.2.2
```
→ `S` code confirmed. Administrative distance 1, metric 0 — expected for a static route.

---

## Task 2 — Static Route on Fallout-RT1

Told Fallout-RT1 how to reach the Cafe LAN (192.168.1.0/24) via the WAN next-hop (192.168.2.1).

> **Mistake logged:** Ran `ip route` at the exec prompt (`#`) instead of config mode. Got `% Invalid input detected` error. Fixed by entering `conf t` first.

```
Fallout-RT1# conf t
Fallout-RT1(config)# ip route 192.168.1.0 255.255.255.0 192.168.2.1
Fallout-RT1(config)# exit
```

**Verified in routing table:**
```
S  192.168.1.0/24 [1/0] via 192.168.2.1
C  192.168.2.0/30  is directly connected, Ethernet0/1
L  192.168.2.2/32  is directly connected, Ethernet0/1
C  192.168.3.0/24  is directly connected, Ethernet0/0
L  192.168.3.1/32  is directly connected, Ethernet0/0
```
→ Both static and connected routes present. Return path confirmed.

---

## Task 3 — End-to-End Connectivity Test

**Forward path — Cafe-RT1 → Fallout server:**
```
Cafe-RT1# ping 192.168.3.100
Sending 5, 100-byte ICMP Echos to 192.168.3.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
→ 5/5. Forward path working.

**Return path — Fallout-SRV → Cafe host:**
```
cisco@fallout-srv:~$ ping -c5 192.168.1.50
64 bytes from 192.168.1.50: seq=0 ttl=62 time=2.198 ms
64 bytes from 192.168.1.50: seq=1 ttl=62 time=1.146 ms
64 bytes from 192.168.1.50: seq=2 ttl=62 time=1.090 ms
64 bytes from 192.168.1.50: seq=3 ttl=62 time=1.141 ms
64 bytes from 192.168.1.50: seq=4 ttl=62 time=1.305 ms
5 packets transmitted, 5 packets received, 0% packet loss
```
→ 5/5. Return path working. TTL=62 (started at 64, decremented once per router hop × 2 hops).

**Configs saved:**
```
Cafe-RT1# write memory
Fallout-RT1# write memory
```

---

## What Each Step Confirmed

| Step | What it proved |
|------|---------------|
| Baseline `show ip route` | Connected routes are automatic; remote networks are not |
| Static route on Cafe-RT1 | Forward path: Cafe → Fallout LAN now routable |
| Static route on Fallout-RT1 | Return path: Fallout → Cafe LAN now routable |
| Router ping (Cafe-RT1 → 192.168.3.100) | Forward path functional |
| Host ping (fallout-srv → 192.168.1.50) | Full end-to-end reachability confirmed, both directions |

---

## Mistakes & Observations

- **Exec vs config mode:** Ran `ip route` at `#` prompt on Fallout-RT1 — got invalid input error. Static routes require global config mode (`conf t`). Easy to do when moving fast.
- **TTL=62 on the host ping:** Each router hop decrements TTL by 1. Two hops (Fallout-RT1 → Cafe-RT1) = TTL 64 → 62. Useful sanity check when reading ping output.
- **No first-ping drop:** Both pings succeeded 5/5 with no timeout on the first probe. ARP had already resolved from earlier testing, so no delay.
