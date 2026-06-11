
## Topology
```
[PC1]─────[Cafe-Rtr Eth0/0]───[Cafe-Rtr Eth0/1]─────WAN─────[ISP-Rtr Eth0/0]
      192.168.1.1/24                  216.0.5.2/30              216.0.5.1/30
                                                                      |
                                                          Loopback1: 1.1.1.1/32
                                                          Loopback2: 8.8.8.8/32
```

![[ccna-summer-2026/day_27/lab/Network_diagram.png]]
---

## ISP-Rtr Configuration

### Task 0 — Verify Initial State
```
show ip interface brief
```
```
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            unassigned      YES TFTP   administratively down down
Ethernet0/1            unassigned      YES TFTP   administratively down down
Ethernet0/2            unassigned      YES TFTP   administratively down down
Ethernet0/3            unassigned      YES TFTP   administratively down down
```
All interfaces down before configuration. Hostname still showing default `inserthostname-here`.

### Task 1 — Harden ISP-Rtr
```
configure terminal

hostname ISP-Rtr

line console 0
 password Cisco
 login
exit

line vty 0 4
 password Cisco
 login
exit

enable secret Cisco

service password-encryption
```
> Note: `service password-encryption` was added beyond the lab minimum — applies Type 7 obfuscation to line passwords.

### Task 2 — Provision WAN Interface
```
interface ethernet0/0
 ip address 216.0.5.1 255.255.255.252
 no shutdown
exit
```
Interface came up immediately:
```
%LINK-3-UPDOWN: Interface Ethernet0/0, changed state to up
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/0, changed state to up
```

Verified with `show ip interface brief`:
```
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            216.0.5.1       YES manual up                    up
```

Ping to Cafe-Rtr WAN address:
```
ISP-Rtr#ping 216.0.5.2
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/1/1 ms
```
> First packet dropped while interface was still coming up — normal behavior. 80% success on first attempt is expected; a repeat ping would return 100%.

### Task 3 — Simulate Public DNS Targets
```
interface Loopback 1
 ip address 1.1.1.1 255.255.255.255
 description Cloudflare_dns
exit

interface Loopback 2
 ip address 8.8.8.8 255.255.255.255
 description google_dns
exit
```

Both loopbacks came up immediately (loopbacks are always up by default).

Final `show ip interface brief`:
```
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            216.0.5.1       YES manual up                    up
Loopback1              1.1.1.1         YES manual up                    up
Loopback2              8.8.8.8         YES manual up                    up
```

Saved with `write memory`.

---

## Cafe-Rtr Verification

Pre-existing state confirmed — Cafe-Rtr was already configured with addressing and a default route:
```
show ip interface brief
```
```
Interface              IP-Address      OK? Method Status                Protocol
Ethernet0/0            192.168.1.1     YES TFTP   up                    up
Ethernet0/1            216.0.5.2       YES TFTP   up                    up
```

Routing table confirmed default route toward ISP:
```
Gateway of last resort is 216.0.5.1 to network 0.0.0.0
S*    0.0.0.0/0 [1/0] via 216.0.5.1
```

Pings to simulated DNS targets succeeded:
```
Cafe-Rtr#ping 1.1.1.1
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

Cafe-Rtr#ping 8.8.8.8
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```
> Cafe-Rtr can reach the simulated internet because it sources packets from 216.0.5.2 — a public address ISP-Rtr knows via its connected route.

---

## PC1 Reachability Test

```
cisco@pc1:~$ ping 1.1.1.1
--- 1.1.1.1 ping statistics ---
60 packets transmitted, 0 packets received, 100% packet loss
```

**100% packet loss — expected.** PC1 sources traffic from 192.168.1.x. ISP-Rtr has no route to 192.168.1.0/24, so return traffic is dropped. The NAT gap is confirmed.

ISP-Rtr routing table at this point contains only:
- `216.0.5.0/30` — connected (WAN link)
- `1.1.1.1/32` — connected (Loopback1)
- `8.8.8.8/32` — connected (Loopback2)

No entry for `192.168.1.0/24` — ISP has no path back to the internal network.

---

## Key Observation
The fix is **not** to add a static route on ISP-Rtr pointing to 192.168.1.0/24. In the real world, an ISP will never hold a route to your private address space. NAT solves this by making PC1's traffic appear to originate from 216.0.5.2 — a public address ISP-Rtr already has a connected route for — so return traffic has a valid path back.

---

## Completion Checklist
- [x] ISP-Rtr Ethernet0/0 — 216.0.5.1/30, up/up
- [x] Loopback1 — 1.1.1.1/32, up/up (Cloudflare DNS)
- [x] Loopback2 — 8.8.8.8/32, up/up (Google DNS)
- [x] Cafe-Rtr pings 1.1.1.1 and 8.8.8.8 — 100% success
- [x] PC1 pings 1.1.1.1 — 100% packet loss (NAT gap confirmed)
- [x] ISP-Rtr `show ip route` shows no 192.168.1.0/24 entry
- [x] Configuration saved with `write memory`
