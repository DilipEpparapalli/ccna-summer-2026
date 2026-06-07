# Lab 08-01 — Coffee House ↔ Fallout Local Routing

**Platform:** Cisco CML (IOS-XE 17.16)
**Topology:** Two routers — Cafe-RT1 and Fallout-RT1 — connected via a point-to-point Ethernet link, each with a stub LAN interface.

---

## Topology

```
[Cafe LAN]                [P2P /30]               [Fallout LAN]
192.168.42.0/24          10.8.0.0/30            192.168.84.0/24
      |                  .1       .2                    |
  Cafe-RT1 ---Eth0/1-----------Eth0/1--- Fallout-RT1
  Eth0/0                                          Eth0/0
192.168.42.1                                  192.168.84.1
```
![[ccna-summer-2026/day_22/lab/Circuit_Diagram.png]]


---

## Objectives Completed

- Confirmed both routers meet Castle security standards (type 9 secrets, local console/VTY auth, `login on-success log`, `login on-failure log`)
- Verified IOS-XE version 17.16 on both routers
- Activated LAN gateways on Eth0/0 with correct addressing, preserving existing descriptions
- Lit up the point-to-point link on Eth0/1 with 10.8.0.0/30 addressing
- Installed static routes on both routers for cross-site reachability
- Proved bidirectional reachability via ping
- Saved configurations to NVRAM on both routers

---

## Task Execution

### Task 1 — Baseline

Confirmed pre-change state on both routers. All interfaces were administratively down with no IP addresses assigned. Descriptions were pre-populated and left intact throughout.

```
! Cafe-RT1 pre-change
Et0/0   admin down / down   ## Coffee House LAN - configure during lab
Et0/1   admin down / down   ## P2P-to-Fallout - configure during lab

! Fallout-RT1 pre-change
Et0/0   admin down / down   ## Fallout Shelter LAN - configure during lab
Et0/1   admin down / down   ## P2P-to-CoffeeHouse - configure during lab
Et0/2   admin down / down   ## Spare module - keep shutdown
```

Security posture confirmed on both routers:
- `enable secret 9` (type 9 hash) in place
- `username cisco privilege 15 secret 9` configured
- `login local` on console, aux, and VTY lines
- `login on-success log` and `login on-failure log` applied

### Task 2 — LAN Gateways

```
! Cafe-RT1
interface Ethernet0/0
 ip address 192.168.42.1 255.255.255.0
 no shutdown

! Fallout-RT1
interface Ethernet0/0
 ip address 192.168.84.1 255.255.255.0
 no shutdown
```

Both interfaces settled at up/up with correct addressing confirmed via `show ip interface brief`.

### Task 3 — Point-to-Point Link

```
! Cafe-RT1
interface Ethernet0/1
 ip address 10.8.0.1 255.255.255.252
 no shutdown

! Fallout-RT1
interface Ethernet0/1
 ip address 10.8.0.2 255.255.255.252
 no shutdown
```

Both ends reported up/up. Descriptions preserved on both routers.

**Note:** `ip route` typed at privileged exec level (not config mode) produced `% Invalid input detected` — this is expected IOS behavior. Command must be entered from `configure terminal`.

### Task 4 — Static Routes

```
! Cafe-RT1 — reach Fallout LAN via Fallout's P2P address
ip route 192.168.84.0 255.255.255.0 10.8.0.2

! Fallout-RT1 — reach Cafe LAN via Cafe's P2P address
ip route 192.168.42.0 255.255.255.0 10.8.0.1
```

Routing table verification — Cafe-RT1:

```
C    10.8.0.0/30        directly connected, Ethernet0/1
L    10.8.0.1/32        directly connected, Ethernet0/1
C    192.168.42.0/24    directly connected, Ethernet0/0
L    192.168.42.1/32    directly connected, Ethernet0/0
S    192.168.84.0/24    [1/0] via 10.8.0.2
```

Routing table verification — Fallout-RT1:

```
C    10.8.0.0/30        directly connected, Ethernet0/1
L    10.8.0.2/32        directly connected, Ethernet0/1
S    192.168.42.0/24    [1/0] via 10.8.0.1
C    192.168.84.0/24    directly connected, Ethernet0/0
L    192.168.84.1/32    directly connected, Ethernet0/0
```

Static entries marked `S`, resolving via next-hop across Ethernet0/1 as required.

### Task 5 — Reachability Proof and Save

```
! From Cafe-RT1
ping 192.168.84.1
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

! From Fallout-RT1
ping 192.168.42.1
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

Interface counters on both routers showed zero errors, zero drops, zero CRC — clean turn-up on all active interfaces.

Configurations saved on both routers:
```
wr
Building configuration... [OK]
```

---

## What This Proved

- A router only knows about networks directly connected to it unless told otherwise — static routes are the manual mechanism for adding that knowledge.
- The next-hop in a static route must be a directly reachable address — the far end of the point-to-point link — not the destination network itself.
- A /30 subnet (255.255.255.252) provides exactly 2 usable host addresses, purpose-built for point-to-point router links where no additional hosts will ever connect.
- `show ip route` output distinguishes connected routes (`C`), local host routes (`L`), and static routes (`S`) — exam expects you to read and interpret this table fluently.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `show ip interface brief` | Confirm IP addressing and interface state |
| `show interfaces description` | Verify descriptions survived configuration changes |
| `show interfaces ethernet0/x` | Detailed counters — errors, drops, input/output rates |
| `show ip route` | Confirm routing table entries and next-hop resolution |
| `show running-config` | Full config review before saving |
| `ip route [network] [mask] [next-hop]` | Install a static route |
| `no shutdown` | Return interface to service |
| `wr` | Save running config to NVRAM |
| `ping [address]` | Prove end-to-end reachability |

---

## Gotchas Observed

- `ip route` entered at `#` prompt (privileged exec) fails with `% Invalid input detected` — must be inside `configure terminal`
- IOS-XE on CML does not require `switchport trunk encapsulation` before trunk commands (unlike older IOSv) — platform differences matter when reading lab guides written for physical gear
- Interface counters showed 2 interface resets on Eth0/0 and Eth0/1 — normal artifact of the interface cycling through shutdown → no shutdown during configuration
