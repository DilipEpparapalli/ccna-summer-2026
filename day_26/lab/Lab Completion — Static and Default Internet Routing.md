**Platform:** Cisco Modeling Labs (CML) — IOSv routers, Linux endpoints  
**Objective:** Replace EIGRP with static routing, establish default Internet gateway, validate route trust and floating static failover logic

---

## Topology

```
Cafe-PC1              Cafe-RT1                    Fallout-RT1         Shelter-SRV
192.168.1.50  ──────  Eth0/0: 192.168.1.1         Eth0/0: 192.168.2.2
                      Eth0/1: 192.168.2.1 ──────── Eth0/1: 192.168.3.1 ──────  192.168.3.100
                      Eth0/2: 216.0.5.2
                            │
                       ISP-GW: 216.0.5.1
                       Loopback: 203.0.113.8 (simulated Internet)
```

**Point-to-point WAN link:** 192.168.2.0/30  
**Cafe LAN:** 192.168.1.0/24  
**Shelter LAN:** 192.168.3.0/24  
**ISP link:** 216.0.5.0/30
![[ccna-summer-2026/day_26/lab/Network_Diagram.png]]
---

## Task 1 — Interface Verification

Confirmed all interfaces up/up with correct addressing before touching routing.

**Cafe-RT1 `show ip interface brief`:**
```
Ethernet0/0   192.168.1.1   up/up     ← LAN
Ethernet0/1   192.168.2.1   up/up     ← WAN (point-to-point to Fallout-RT1)
Ethernet0/2   216.0.5.2     up/up     ← ISP uplink
Ethernet0/3   unassigned    admin down
```

**Fallout-RT1 `show ip interface brief`:**
```
Ethernet0/0   192.168.2.2   up/up     ← WAN (point-to-point to Cafe-RT1)
Ethernet0/1   192.168.3.1   up/up     ← Shelter LAN
Ethernet0/2   unassigned    admin down
Ethernet0/3   unassigned    admin down
```

**Baseline routing tables** showed `C` and `L` entries for directly connected networks only (plus a leftover `D` EIGRP route on Cafe-RT1 — removed in Task 2).

---

## Task 2 — Replace EIGRP with Static Routes

**Problem found:** Cafe-RT1 still had a live EIGRP-learned route:
```
D     192.168.3.0/24 [90/307200] via 192.168.2.2
```

EIGRP process was still active. Entered `router eigrp 1` on Cafe-RT1 to confirm, then exited without removing it — removal happened implicitly when the static route was installed and EIGRP was no longer needed. *(Note: clean practice would be `no router eigrp 1` to fully remove the process.)*

**Static routes configured:**

```
! Cafe-RT1
ip route 192.168.3.0 255.255.255.0 192.168.2.2

! Fallout-RT1
ip route 192.168.1.0 255.255.255.0 192.168.2.1
```

**Verification — Cafe-RT1 routing table after static install:**
```
S     192.168.3.0/24 [1/0] via 192.168.2.2
```
EIGRP `D` entry replaced by `S` entry. AD confirmed as `[1/0]`.

**End-to-end ping — Cafe-PC1 → Shelter-SRV:**
```
cisco@Cafe-PC1:~$ ping 192.168.3.100
64 bytes from 192.168.3.100: ttl=62 time=1.347ms avg — 0% packet loss
```

**Return path — Shelter-SRV → Cafe-PC1:**
```
cisco@Shelter-SRV:~$ ping 192.168.1.50
64 bytes from 192.168.1.50: ttl=62 time=1.082ms avg — 0% packet loss
```

`ttl=62` on both confirms traffic is crossing **two routers** (started at 64, decremented once per hop).

---

## Task 3 — Default Internet Gateway on Cafe-RT1

**ISP-GW verified reachable** at 216.0.5.1 (Ethernet0/0 up/up, Loopback0 = 203.0.113.8 simulating Internet).

**Default route installed on Cafe-RT1:**
```
ip route 0.0.0.0 0.0.0.0 216.0.5.1
```

**Routing table after install:**
```
Gateway of last resort is 216.0.5.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 216.0.5.1
S     192.168.3.0/24 [1/0] via 192.168.2.2
```

`S*` = static candidate default. "Gateway of last resort" line confirms IOS recognizes it.

**Internet reachability test:**
```
Cafe-RT1# ping 203.0.113.8
!!!!!  — 100% success, 1ms avg
```

Static LAN route (`192.168.3.0/24`) still present and unaffected — more specific routes always win over the default.

---

## Task 4 — Route Trust and Floating Static

### AD Verification

| Route | AD | Metric | Notes |
|-------|----|--------|-------|
| `S 192.168.3.0/24` | 1 | 0 | Static inter-shelter route |
| `S* 0.0.0.0/0` | 1 | 0 | Default Internet gateway |
| `C` connected routes | 0 | — | Always preferred |

Static LAN routes outrank the default path because of **route specificity** (longest prefix match), not AD. Both are AD 1 — the `/24` wins over `/0` for matching LAN destinations regardless of AD.

### Floating Static Default on Fallout-RT1

**Purpose:** Simulate a backup default path that stays dormant while any preferred route exists, activating automatically if that route disappears.

**Command:**
```
Fallout-RT1(config)# ip route 0.0.0.0 0.0.0.0 192.168.2.1 5
```
The trailing `5` manually overrides the default AD of 1, setting it to 5.

**Routing table after install:**
```
Gateway of last resort is 192.168.2.1 to network 0.0.0.0

S*    0.0.0.0/0 [5/0] via 192.168.2.1
S     192.168.1.0/24 [1/0] via 192.168.2.1
```

**Observation — why it appeared in the table:**  
Fallout-RT1 had no competing default route. With nothing better (lower AD) for `0.0.0.0/0`, the AD 5 route installed itself as the active gateway of last resort. This is expected — "floating" only means it yields to a *lower-AD* route for the *same destination*. Since no primary default existed here, it became active.

**The dormant behavior explained:**  
If a primary default (e.g., AD 1 learned via another protocol) were also present, IOS would install only the AD 1 route. The AD 5 entry would sit in the config (`show run`) but be absent from `show ip route` — waiting silently. The moment the AD 1 route's next-hop became unreachable, the AD 5 entry would surface automatically with no manual intervention.

**Cleanup — restore baseline:**
```
Fallout-RT1(config)# no ip route 0.0.0.0 0.0.0.0 192.168.2.1 5
```

### Final Routing Tables

**Cafe-RT1 (final):**
```
Gateway of last resort is 216.0.5.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 216.0.5.1
C     192.168.1.0/24 via Ethernet0/0
L     192.168.1.1/32 via Ethernet0/0
C     192.168.2.0/30 via Ethernet0/1
L     192.168.2.1/32 via Ethernet0/1
S     192.168.3.0/24 [1/0] via 192.168.2.2
C     216.0.5.0/30 via Ethernet0/2
L     216.0.5.2/32 via Ethernet0/2
```

**Fallout-RT1 (final, floating static removed):**
```
Gateway of last resort is not set

S     192.168.1.0/24 [1/0] via 192.168.2.1
C     192.168.2.0/30 via Ethernet0/0
L     192.168.2.2/32 via Ethernet0/0
C     192.168.3.0/24 via Ethernet0/1
L     192.168.3.1/32 via Ethernet0/1
```

---

## Route Code Reference (used in this lab)

| Code | Meaning |
|------|---------|
| `C` | Connected — interface is up with an IP assigned |
| `L` | Local — the router's own interface address (/32) |
| `S` | Static — manually configured route |
| `S*` | Static candidate default — the active gateway of last resort |
| `D` | EIGRP-learned route (DUAL algorithm) — present at start, replaced |

---

## Mistakes and Lessons Learned

- **EIGRP process not explicitly removed:** After installing the static route on Cafe-RT1, the `D` entry was displaced but `router eigrp 1` process was not removed with `no router eigrp 1`. In a production environment, leaving a routing process running when it's no longer needed is unnecessary overhead. Clean practice: remove the process explicitly.
- **Floating static behavior context:** The AD 5 default on Fallout-RT1 *appeared active* in `show ip route` because there was no competing default to displace it. Floating statics are dormant only when a lower-AD route to the same destination exists. Without that competing route, they install normally.
