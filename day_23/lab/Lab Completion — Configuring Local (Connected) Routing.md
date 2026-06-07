

**Week:** 07
**Lab:** Skill 9 Lesson 02 — Configuring Local Routing
**Environment:** Cisco Modeling Labs (CML) — IOSv routers
**Topology:** Cafe-RT1 ↔ Fallout-RT1 via point-to-point /30 link

---

## Topology

```
[Coffee House LAN]          [P2P Link /30]         [Fallout Shelter LAN]
192.168.1.0/24              192.168.2.0/30           192.168.3.0/24
       |                                                      |
  Eth0/0 (.1)──── Cafe-RT1 ──── Eth0/1 (.1)──(.2) Eth0/1 ──── Fallout-RT1 ──── Eth0/0 (.1)
  192.168.1.1                   192.168.2.1  192.168.2.2                        192.168.3.1
```
![[ccna-summer-2026/day_23/lab/Circuit_Diagram.png]]

---

## Task 0 — Prepare Cafe-RT1

**Starting state:** All interfaces unassigned, administratively down.

```
Interface              IP-Address      OK? Method Status       Protocol
Ethernet0/0            unassigned      YES TFTP   admin down   down
Ethernet0/1            unassigned      YES TFTP   admin down   down
Ethernet0/2            unassigned      YES TFTP   admin down   down
Ethernet0/3            unassigned      YES TFTP   admin down   down
```

**Commands run:**

```
Cafe-RT1# conf t
Cafe-RT1(config)# interface ethernet0/0
Cafe-RT1(config-if)# ip address 192.168.1.1 255.255.255.0
Cafe-RT1(config-if)# no shutdown
Cafe-RT1(config-if)# exit
Cafe-RT1(config)# interface ethernet0/1
Cafe-RT1(config-if)# ip address 192.168.2.1 255.255.255.252
Cafe-RT1(config-if)# no shutdown
Cafe-RT1(config-if)# exit
Cafe-RT1(config)# exit
```

**Verified with `show ip interface brief`:**

```
Interface              IP-Address      OK? Method Status   Protocol
Ethernet0/0            192.168.1.1     YES manual up       up
Ethernet0/1            192.168.2.1     YES manual up       up
Ethernet0/2            unassigned      YES TFTP   admin down down
Ethernet0/3            unassigned      YES TFTP   admin down down
```

**What this confirmed:** Both active interfaces came up/up immediately after `no shutdown`. Eth0/1 went up because Fallout-RT1 had already been configured on the other end of the P2P link.

---

## Task 1 — Configure Fallout-RT1

**Commands run:**

```
Fallout-RT1# conf t
Fallout-RT1(config)# interface ethernet0/1
Fallout-RT1(config-if)# ip address 192.168.2.2 255.255.255.252
Fallout-RT1(config-if)# no shutdown
Fallout-RT1(config-if)# exit
Fallout-RT1(config)# interface ethernet0/0
Fallout-RT1(config-if)# ip address 192.168.3.1 255.255.255.0
Fallout-RT1(config-if)# no shutdown
Fallout-RT1(config-if)# exit
Fallout-RT1(config)# exit
```

**Verified with `show ip interface brief`:**

```
Interface              IP-Address      OK? Method Status   Protocol
Ethernet0/0            192.168.3.1     YES manual up       up
Ethernet0/1            192.168.2.2     YES manual up       up
Ethernet0/2            unassigned      YES TFTP   admin down down
Ethernet0/3            unassigned      YES TFTP   admin down down
```

---

## Task 2 — Inspect Connected Routes

### Cafe-RT1 — `show ip route`

```
Gateway of last resort is not set

      192.168.1.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.1.0/24 is directly connected, Ethernet0/0
L        192.168.1.1/32 is directly connected, Ethernet0/0
      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/30 is directly connected, Ethernet0/1
L        192.168.2.1/32 is directly connected, Ethernet0/1
```

### Fallout-RT1 — `show ip route`

```
Gateway of last resort is not set

      192.168.2.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.2.0/30 is directly connected, Ethernet0/1
L        192.168.2.2/32 is directly connected, Ethernet0/1
      192.168.3.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.3.0/24 is directly connected, Ethernet0/0
L        192.168.3.1/32 is directly connected, Ethernet0/0
```

**What the C and L entries mean:**

| Code | Meaning |
|------|---------|
| `C` | Connected — the network address of the subnet on that interface |
| `L` | Local — the specific /32 host address assigned to the router's own interface |

**Key observation:** Neither router shows the other's LAN in its routing table. Cafe-RT1 has no entry for `192.168.3.0/24`. Fallout-RT1 has no entry for `192.168.1.0/24`. This is expected — connected routes only appear for directly attached networks. Static or dynamic routing is required to reach the remote LANs.

---

## Task 3 — Validate the Point-to-Point Link

### Cafe-RT1 → Fallout-RT1

```
Cafe-RT1# ping 192.168.2.2

Sending 5, 100-byte ICMP Echos to 192.168.2.2, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```

**Note:** First probe dropped (`.!!!!`). This is normal — the first ICMP packet triggered ARP on the P2P link while the router resolved the next-hop MAC address. Subsequent probes succeeded immediately.

### Fallout-RT1 → Cafe-RT1

```
Fallout-RT1# ping 192.168.2.1

Sending 5, 100-byte ICMP Echos to 192.168.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

**Note:** 100% success because ARP was already resolved from the Cafe-RT1 ping — the MAC address was cached.

### Configurations saved

```
Cafe-RT1# write memory
Fallout-RT1# write memory
```

---

## Lab Outcomes

| Check | Result |
|-------|--------|
| Cafe-RT1 Eth0/0 up/up with 192.168.1.1/24 | ✓ |
| Cafe-RT1 Eth0/1 up/up with 192.168.2.1/30 | ✓ |
| Fallout-RT1 Eth0/0 up/up with 192.168.3.1/24 | ✓ |
| Fallout-RT1 Eth0/1 up/up with 192.168.2.2/30 | ✓ |
| Cafe-RT1 shows C routes for 192.168.1.0/24 and 192.168.2.0/30 | ✓ |
| Fallout-RT1 shows C routes for 192.168.3.0/24 and 192.168.2.0/30 | ✓ |
| P2P ping Cafe → Fallout succeeds | ✓ (80% — first probe ARP) |
| P2P ping Fallout → Cafe succeeds | ✓ (100%) |
| Configs saved on both routers | ✓ |

---

## What's Missing (Next Lab)

Neither router can reach the other's LAN yet:

- Cafe-RT1 cannot reach `192.168.3.0/24`
- Fallout-RT1 cannot reach `192.168.1.0/24`

Next step: add static routes on both routers to enable LAN-to-LAN reachability across the P2P link.
