
## Lab Environment
- **Platform:** Cisco Modeling Labs (CML) — IOSv router and IOSv L2 switch
- **Devices:** Cafe-RTR1, Cafe-SW1, Cafe-Admin1 (VLAN 10), Cafe-Client1 (VLAN 20)
- **Topology:**

```
[Cafe-Admin1]──Et0/1──Cafe-SW1──Et0/0──(trunk)──Cafe-RTR1 Et0/0
                          │                         ├── Et0/0.10  10.0.18.1/27
[Cafe-Client1]──Et0/2──Cafe-SW1                    └── Et0/0.20  10.0.18.33/27
```
![[Netwotk_diagram_1.png]]
## Objective
Convert the Cafe-SW1 uplink to Cafe-RTR1 into a trunk. Build ROAS subinterfaces
on Cafe-RTR1 for VLAN 10 and VLAN 20. Configure per-VLAN DHCP pools. Confirm
end devices receive correct leases and can ping across VLANs through the router.

---

## Task 0 — Convert Switch Uplink to Trunk (Cafe-SW1)

The Cafe-SW1 Et0/0 port facing the router was still an access port. Converting it
to a trunk is what allows VLAN-tagged frames to reach the router's subinterfaces.

```
Cafe-SW1# conf t
Cafe-SW1(config)# interface ethernet0/0
Cafe-SW1(config-if)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if)# switchport mode trunk
```

Interface flapped briefly — expected when changing port mode:

```
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to up
```

**Trunk verification immediately after — STP still converging:**

```
Cafe-SW1# show interfaces trunk

Port     Mode   Encapsulation  Status    Native vlan
Et0/0    on     802.1q         trunking  1

Port     Vlans in spanning tree forwarding state and not pruned
Et0/0    none   ← STP still converging at this point, not an error
```

The trunk was up and trunking, but the STP forwarding section showed `none` — normal
while Spanning Tree converges after a port mode change. VLANs 10 and 20 showed as
active once the router subinterfaces came up and tagged traffic started flowing.

---

## Task 1 — Build Router Subinterfaces (Cafe-RTR1)

### Remove Legacy IP from Physical Interface

Cafe-RTR1 had a pre-existing IP on Ethernet0/0 from a previous single-pool config.
Cleared it before creating subinterfaces:

```
Cafe-RTR1# conf t
Cafe-RTR1(config)# interface ethernet0/0
Cafe-RTR1(config-if)# no ip address
Cafe-RTR1(config-if)# end
```

Verified the physical interface is now unassigned but still up:

```
Interface     IP-Address   OK? Status   Protocol
Ethernet0/0   unassigned   YES up       up
```

### Error — Tried to Set IP Before Encapsulation

Attempted to assign IP to subinterface before setting encapsulation — IOS rejected it:

```
Cafe-RTR1(config-subif)# ip address 10.0.18.1 255.255.255.224
% Configuring IP routing on a LAN subinterface is only allowed if that
subinterface is already configured as part of an IEEE 802.10, IEEE 802.1Q,
or ISL vLAN.
```

Correct sequence: **encapsulation dot1Q first, then IP address.**

### Error — `name` Is Not a Valid Subinterface Command

Tried to label the subinterface with `name` — not a valid command on subinterfaces:

```
Cafe-RTR1(config-subif)# name ADMIN-DEV
% Invalid input detected at '^' marker.
```

Used `description` instead — that's the correct command.

### Correct Subinterface Config

```
! VLAN 10 — Admin
Cafe-RTR1(config)# interface ethernet0/0.10
Cafe-RTR1(config-subif)# description ADMIN-DEV
Cafe-RTR1(config-subif)# encapsulation dot1Q 10
Cafe-RTR1(config-subif)# ip address 10.0.18.1 255.255.255.224
Cafe-RTR1(config-subif)# exit

! VLAN 20 — Patron
Cafe-RTR1(config)# interface ethernet0/0.20
Cafe-RTR1(config-subif)# description PATRON-DEV
Cafe-RTR1(config-subif)# encapsulation dot1Q 20
Cafe-RTR1(config-subif)# ip address 10.0.18.33 255.255.255.224
Cafe-RTR1(config-subif)# end
```

**Verified subinterface state:**

```
Cafe-RTR1# show ip interface brief

Interface          IP-Address    OK? Status   Protocol
Ethernet0/0        unassigned    YES up        up
Ethernet0/0.10     10.0.18.1     YES up        up
Ethernet0/0.20     10.0.18.33    YES up        up
```

Both subinterfaces up/up. Physical interface has no IP — correct for ROAS.

---

## Task 2 — Configure Per-VLAN DHCP Pools

### Remove Old Single-Pool Config

A previous pool (`Cafe-Base`) was still referencing the old combined /26 network
and causing DHCP NAKs — the router was actively rejecting DHCP requests because
the old pool's gateway didn't match any active subinterface subnet.

```
Cafe-RTR1(config)# no ip dhcp pool Cafe-Base
Cafe-RTR1(config)# no ip dhcp excluded-address 10.0.18.1 10.0.18.10
```

### DHCP NAK Messages — What They Mean

During the cleanup phase, the console showed repeated NAK messages:

```
%DHCPD-7-NAK: DHCP nak sent to client 0152.5400.c4f8.e3
%DHCPD-7-NO_LEASE: DHCP lease assignment failure, client 5254.00c4.f8e3 reason NO POOL
```

A NAK (negative acknowledgment) means the router received a DHCP request but
rejected it — either the pool was misconfigured or no matching pool existed. The
`NO POOL` message means a client requested an address but no pool covered its
subnet. Both cleared after the correct pools were defined.

### Configure Exclusions and Pools

Best practice: define exclusions before pools.

```
! Exclude gateway IPs so they're never handed to clients
ip dhcp excluded-address 10.0.18.1 10.0.18.1
ip dhcp excluded-address 10.0.18.33 10.0.18.33

! VLAN 10 pool
ip dhcp pool ADMIN-10
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
 dns-server 1.1.1.1
 exit

! VLAN 20 pool
ip dhcp pool PATRON-20
 network 10.0.18.32 255.255.255.224
 default-router 10.0.18.33
 dns-server 1.1.1.1
 exit
```

**Verified DHCP binding after clients renewed:**

```
Cafe-RTR1# show ip dhcp binding

IP address    Client-ID / Hardware address    Type       Interface
10.0.18.34    0152.5400.c4f8.e3               Automatic  Ethernet0/0.20
```

---

## Task 3 — Release and Renew on End Devices

Cleared old static/stale addresses on both Linux hosts and triggered fresh DHCP
discovery using `udhcpc`:

**Cafe-Admin1 (VLAN 10):**

```
cisco@cafe-admin1:~$ sudo ifconfig eth0 0.0.0.0 up
cisco@cafe-admin1:~$ sudo route del default 2>/dev/null
cisco@cafe-admin1:~$ sudo udhcpc -i eth0 -n -q

udhcpc: broadcasting discover
udhcpc: broadcasting select for 10.0.18.2, server 10.0.18.1
udhcpc: lease of 10.0.18.2 obtained from 10.0.18.1, lease time 86400
```

**Verified address and routing table:**

```
cisco@cafe-admin1:~$ ifconfig eth0
inet addr:10.0.18.2   Mask:255.255.255.224

cisco@cafe-admin1:~$ route -n
Destination   Gateway      Genmask           Flags
0.0.0.0       10.0.18.1    0.0.0.0           UG      ← default via ROAS gateway
10.0.18.0     0.0.0.0      255.255.255.224   U       ← local subnet route
```

**Cafe-Client1 (VLAN 20):**

```
cisco@cafe-client1:~$ sudo udhcpc -i eth0 -n -q

udhcpc: lease of 10.0.18.34 obtained from 10.0.18.33, lease time 86400
```

**Verified address and routing table:**

```
inet addr:10.0.18.34   Mask:255.255.255.224

Destination   Gateway       Genmask           Flags
0.0.0.0       10.0.18.33    0.0.0.0           UG      ← default via ROAS gateway
10.0.18.32    0.0.0.0       255.255.255.224   U
```

Each client received an address from its own VLAN's pool and its own VLAN's gateway.

---

## Task 4 — Prove Inter-VLAN Routing

**Ping from Cafe-Admin1 (VLAN 10, 10.0.18.2) → Cafe-Client1 (VLAN 20, 10.0.18.34):**

```
cisco@cafe-admin1:~$ ping 10.0.18.34
64 bytes from 10.0.18.34: seq=0 ttl=63 time=1.334 ms
64 bytes from 10.0.18.34: seq=1 ttl=63 time=1.350 ms
4 packets transmitted, 4 packets received, 0% packet loss
```

**Ping from Cafe-Client1 (VLAN 20, 10.0.18.34) → Cafe-Admin1 (VLAN 10, 10.0.18.2):**

```
cisco@cafe-client1:~$ ping 10.0.18.2
64 bytes from 10.0.18.2: seq=0 ttl=63 time=2.341 ms
64 bytes from 10.0.18.2: seq=1 ttl=63 time=1.098 ms
6 packets transmitted, 6 packets received, 0% packet loss
```

**TTL=63 confirms routing happened** — Linux starts TTL at 64, decremented by 1 at
the router hop through Cafe-RTR1. A same-VLAN ping would return TTL=64. This proves
traffic is crossing the VLAN boundary via the router, not staying local.

---

## Mistakes Made

| Mistake | What Happened | Correction |
|---------|---------------|------------|
| `name ADMIN-DEV` on subinterface | Invalid command — subinterfaces don't have a `name` command | Used `description ADMIN-DEV` instead |
| IP address before encapsulation | IOS rejected with "not configured as part of 802.1Q vLAN" error | Set `encapsulation dot1Q 10` before `ip address` |
| Old `Cafe-Base` pool still active | Router was NAKing DHCP requests — pool network didn't match subinterface subnets | Removed old pool with `no ip dhcp pool Cafe-Base` before creating new pools |
| `show dhcp binding` | Invalid command | Correct command is `show ip dhcp binding` |
| `conft t` typo | Invalid input | `conf t` |

---

## Final State After Lab

```
Cafe-RTR1:
  Et0/0          → unassigned, up/up (physical parent)
  Et0/0.10       → 10.0.18.1/27, encap dot1Q 10 (VLAN 10 gateway)
  Et0/0.20       → 10.0.18.33/27, encap dot1Q 20 (VLAN 20 gateway)
  DHCP pool ADMIN-10   → 10.0.18.0/27, gateway 10.0.18.1
  DHCP pool PATRON-20  → 10.0.18.32/27, gateway 10.0.18.33

Cafe-SW1:
  Et0/0   → trunk (802.1Q, carrying VLANs 1, 10, 20) → Cafe-RTR1
  Et0/1   → access VLAN 10 → Cafe-Admin1
  Et0/2   → access VLAN 20 → Cafe-Client1

Cafe-Admin1:  10.0.18.2/27   gateway 10.0.18.1   (DHCP, VLAN 10)
Cafe-Client1: 10.0.18.34/27  gateway 10.0.18.33  (DHCP, VLAN 20)
```

---

## Key Commands Used

```
! Switch — trunk the uplink to router
interface ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk

! Router — clear legacy IP
interface ethernet0/0
 no ip address

! Router — build subinterfaces (encapsulation before IP)
interface ethernet0/0.10
 description ADMIN-DEV
 encapsulation dot1Q 10
 ip address 10.0.18.1 255.255.255.224

interface ethernet0/0.20
 description PATRON-DEV
 encapsulation dot1Q 20
 ip address 10.0.18.33 255.255.255.224

! Router — remove old pool, add per-VLAN pools
no ip dhcp pool Cafe-Base
ip dhcp excluded-address 10.0.18.1 10.0.18.1
ip dhcp excluded-address 10.0.18.33 10.0.18.33
ip dhcp pool ADMIN-10
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
 dns-server 1.1.1.1
ip dhcp pool PATRON-20
 network 10.0.18.32 255.255.255.224
 default-router 10.0.18.33
 dns-server 1.1.1.1

! Linux — force DHCP renewal
sudo ifconfig eth0 0.0.0.0 up
sudo route del default 2>/dev/null
sudo udhcpc -i eth0 -n -q

! Verification
show ip interface brief
show ip dhcp binding
show interfaces trunk
```
