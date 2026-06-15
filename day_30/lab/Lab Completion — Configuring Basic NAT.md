## Objective
Rebuild the full NAT stack on Cafe-Rtr from scratch — progressing through Static NAT, Dynamic NAT with a pool, and NAT Overload (PAT) — while standing up a simulated ISP edge with public DNS loopbacks.

## Topology

```
[PC1: 192.168.1.50]──┐
                      ├──[Cafe-Rtr Eth0/0: 192.168.1.1]──Eth0/1: 216.0.5.2──[ISP-Rtr Eth0/0: 216.0.5.1]
[PC2: 192.168.1.51]──┘                                   (WAN /30)               │
                                                                               Lo0: 1.1.1.1/32
                                                                               Lo1: 8.8.8.8/32
```
![[ccna-summer-2026/day_30/lab/Network_Diagram.png]]
---

## Task 1 — Rebuild the ISP Edge

### ISP-Rtr Configuration

```
hostname ISP-Rtr

line console 0
 password Cisco
 login

line vty 0 4
 password Cisco
 login

enable secret Cisco
service password-encryption

interface Ethernet0/0
 ip address 216.0.5.1 255.255.255.252
 no shutdown

interface Loopback0
 ip address 1.1.1.1 255.255.255.255
 description cloudflare_dns

interface Loopback1
 ip address 8.8.8.8 255.255.255.255
 description google_dns
```

### Verification

```
! From ISP-Rtr
show ip interface brief
! Expected: Eth0/0 up/up, Lo0 1.1.1.1 up/up, Lo1 8.8.8.8 up/up

! From Cafe-Rtr
ping 1.1.1.1   ! success
ping 8.8.8.8   ! success
```

**Note:** ISP-Rtr had a static route `216.0.5.0/24 via 216.0.5.2` pre-staged in the topology, which is what allowed Cafe-Rtr to reach the loopbacks without additional routing config. ISP-Rtr could NOT reach 192.168.1.x — that's intentional; private addresses have no return path without NAT.

---

## Task 2 — Static NAT

Expose PC1 with a one-to-one public mapping. PC2 remains unadvertised.

```
! Mark interface roles
interface Ethernet0/0
 ip nat inside

interface Ethernet0/1
 ip nat outside

! Create static mapping
ip nat inside source static 192.168.1.50 216.0.5.20
```

### Verification

```
show ip nat translations
! Expected:
! --- 216.0.5.20   192.168.1.50   ---   ---

! From ISP-Rtr
ping 216.0.5.20   ! hits PC1 — success
ping 216.0.5.21   ! no mapping for PC2 — fails (expected)
```

---

## Task 3 — Dynamic NAT with Pool

Replace static mapping with a pool that serves the entire LAN subnet.

```
! Remove static entry
no ip nat inside source static 192.168.1.50 216.0.5.20

! Define which inside addresses can be translated
access-list 1 permit 192.168.1.0 0.0.0.255

! Create the public pool
ip nat pool Cafe-Public 216.0.5.50 216.0.5.100 netmask 255.255.255.0

! Bind ACL to pool
ip nat inside source list 1 pool Cafe-Public
```

### Verification

```
show ip access-lists
! Expected: ACL 1 — permit 192.168.1.0 wildcard 0.0.0.255

show ip nat translations
! PC1 gets 216.0.5.50, PC2 gets 216.0.5.51 — sequential pool allocation

show ip nat statistics
! pool Cafe-Public: 51 total addresses, 2 allocated (3%)
```

---

## Task 4 — NAT Overload (PAT)

Collapse all inside hosts behind the WAN interface IP (216.0.5.2), differentiated by port number.

```
! Step 1 — Clear active translations first (required before removing a rule in use)
clear ip nat translation *

! Step 2 — Remove dynamic pool rule
no ip nat inside source list 1 pool Cafe-Public

! Step 3 — Shut interfaces to prevent new sessions during reconfiguration
interface Ethernet0/0
 shutdown
interface Ethernet0/1
 shutdown

! Step 4 — Apply overload rule (ACL 1 already exists, reuse it)
ip nat inside source list 1 interface Ethernet0/1 overload

! Step 5 — Restore interfaces
interface Ethernet0/0
 no shutdown
interface Ethernet0/1
 no shutdown
```

### Verification

```
show ip nat translations
! Both PC1 and PC2 now appear under the same inside global: 216.0.5.2
! Each session gets a unique port number (1024, 1025, etc.)
! Example:
! icmp 216.0.5.2:1024   192.168.1.50:736   1.1.1.1:736   1.1.1.1:1024
! icmp 216.0.5.2:1025   192.168.1.51:741   1.1.1.1:741   1.1.1.1:1025
```

---

## Mistakes Made

### 1. Tried to remove dynamic NAT rule while translations were still active

**What happened:** Ran `no ip nat inside source list 1 pool Cafe-Public` while PC sessions were still in the NAT table. IOS rejected it:
```
%Dynamic mapping in use, cannot remove
```
Then shut down both interfaces hoping that would clear the table — it didn't. IOS still blocked the removal.

**Fix:** The correct sequence is:
1. `clear ip nat translation *` — flush the table first
2. Then `no ip nat inside source list 1 pool Cafe-Public`

Shutting interfaces does NOT automatically clear the NAT translation table.

---

## Key Observations

- **Static NAT** = one inside host gets one permanent public IP. Outside can initiate inbound. No ACL needed.
- **Dynamic NAT** = pool of public IPs handed out on demand. No port tracking. Pool exhaustion silently drops new sessions.
- **PAT/Overload** = single public IP shared by all inside hosts, distinguished by source port. What your home router does.
- The inside/outside interface designations (`ip nat inside` / `ip nat outside`) persist across all three NAT types — you set them once in Task 2 and never touched them again.
- ACL 1 was defined in Task 3 and reused directly in Task 4. No need to redefine it.

---

## Commands Reference

```bash
# Interface roles
ip nat inside
ip nat outside

# Static NAT
ip nat inside source static <inside-local> <inside-global>
no ip nat inside source static <inside-local> <inside-global>

# Dynamic NAT
access-list 1 permit <network> <wildcard>
ip nat pool <name> <start-ip> <end-ip> netmask <mask>
ip nat inside source list 1 pool <name>
no ip nat inside source list 1 pool <name>

# PAT / Overload
ip nat inside source list 1 interface <outside-interface> overload

# Verification and troubleshooting
show ip nat translations
show ip nat statistics
show ip access-lists
clear ip nat translation *
```
