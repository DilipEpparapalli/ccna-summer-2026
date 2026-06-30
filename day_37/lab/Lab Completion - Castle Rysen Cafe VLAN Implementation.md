
## Objective
Complete the VLAN rollout at Castle Rysen Cafe (District Shop 01). Subinterfaces were pre-configured on Cafe-RTR1. The job was to bring up the router parent interface, configure trunk and access ports on both switches, rehome the Plex server into VLAN 10, and lock down all unused ports.

---

## Topology

```
                  [Cafe-RTR1]
                  Et0/0 (trunk to SW1)
                      |
              [Cafe-SW1] Et0/0
              Et0/1 ---|--- Et0/1 [Cafe-SW2]
              Et0/2 → Admin device (VLAN 10)
              Et0/3 → Patron device (VLAN 20)
              Et1/0 → Cafe-WAP1 (trunk)
                              Et0/3 → Admin device (VLAN 10)
                              Et1/0 → Cafe-WAP2 (trunk)
                              Et1/2 → Patron device (VLAN 20)
                              Et6/0 → Cafe-PLEX1 (access, VLAN 10)
```
![[ccna-summer-2026/day_37/lab/Network_Diagram.png]]
---

## Baseline State (Task 0)

Both switches had VLANs 10 and 20 created but using default names (`VLAN0010`, `VLAN0020`). All ports were in VLAN 1. `show interfaces trunk` returned immediately to prompt — no trunks active yet.

Cafe-RTR1 subinterfaces were pre-configured but the parent `Ethernet0/0` was administratively down, which meant both subinterfaces were also down:

```
Ethernet0/0            unassigned      YES unset  administratively down down
Ethernet0/0.10         10.0.18.1       YES TFTP   administratively down down
Ethernet0/0.20         10.0.18.33      YES TFTP   administratively down down
```

---

## Task 1 — Enable Router Interface and Configure Trunks

### Cafe-RTR1
Brought up the parent interface. Both subinterfaces came up automatically:

```
Cafe-RTR1(config)# interface ethernet0/0
Cafe-RTR1(config-if)# no shutdown
```

Post-change:
```
Ethernet0/0            unassigned      YES unset  up   up
Ethernet0/0.10         10.0.18.1       YES TFTP   up   up
Ethernet0/0.20         10.0.18.33      YES TFTP   up   up
```

### Cafe-SW1
Named VLANs and configured trunks on Et0/0 (to RTR1), Et0/1 (to SW2), and Et1/0 (to WAP1):

```
Cafe-SW1(config)# vlan 10
Cafe-SW1(config-vlan)# name ADMIN-DEV
Cafe-SW1(config)# vlan 20
Cafe-SW1(config-vlan)# name PATRON-DEV

Cafe-SW1(config)# interface range ethernet 0/0-1, ethernet 1/0
Cafe-SW1(config-if-range)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if-range)# switchport mode trunk
Cafe-SW1(config-if-range)# switchport trunk allowed vlan 10,20
```

Trunk verification:
```
Port     Mode   Encapsulation  Status     Native vlan
Et0/0    on     802.1q         trunking   1
Et0/1    on     802.1q         trunking   1
Et1/0    on     802.1q         trunking   1

Port     Vlans allowed on trunk
Et0/0    10,20
Et0/1    10,20
Et1/0    10,20
```

### Cafe-SW2
Same VLAN naming, trunks on Et0/1 (to SW1) and Et1/0 (to WAP2):

```
Cafe-SW2(config)# vlan 10
Cafe-SW2(config-vlan)# name ADMIN-DEV
Cafe-SW2(config)# vlan 20
Cafe-SW2(config-vlan)# name PATRON-DEV

Cafe-SW2(config)# interface range ethernet 0/1, ethernet 1/0
Cafe-SW2(config-if-range)# switchport trunk encapsulation dot1q
Cafe-SW2(config-if-range)# switchport mode trunk
Cafe-SW2(config-if-range)# switchport trunk allowed vlan 10,20
```

---

## Task 2 — Rehome the Plex Server

Cafe-PLEX1 was connected to Cafe-SW2 `Ethernet6/0`, which was sitting in VLAN 1. Despite having the correct IP (10.0.18.6/27), it had no path to the router because VLAN 1 has no subinterface on Cafe-RTR1.

```
Cafe-SW2(config)# interface ethernet6/0
Cafe-SW2(config-if)# description PLEX-SERVER
Cafe-SW2(config-if)# switchport mode access
Cafe-SW2(config-if)# switchport access vlan 10
```

Post-change on Cafe-SW2:
```
10   ADMIN-DEV    active    Et0/3, Et6/0
```

### Plex Server Verification

```
cisco@Cafe-PLEX1:~$ ifconfig eth0
inet addr:10.0.18.6  Bcast:10.0.18.31  Mask:255.255.255.224

cisco@Cafe-PLEX1:~$ ping 10.0.18.1   ← first attempt
64 bytes from 10.0.18.1: seq=12 ttl=255 time=2029.634 ms
...
18 packets transmitted, 6 packets received, 66% packet loss

cisco@Cafe-PLEX1:~$ ping 10.0.18.1   ← second attempt
64 bytes from 10.0.18.1: seq=0 ttl=255 time=1.188 ms
5 packets transmitted, 5 packets received, 0% packet loss
```

---

## Task 3 — Unused Port Lockdown

### Cafe-SW1
Shut down all ports except Et0/0–0/3 and Et1/0 using a long interface range chain (IOL doesn't support cross-slot range shorthand):

```
Cafe-SW1(config)# interface range ethernet 1/1-3, ethernet 2/0-3, ethernet 3/0-3, ethernet 4/0-3, ethernet 5/0-3, ethernet 6/0-3
Cafe-SW1(config-if-range)# shutdown
```

### Cafe-SW2
Same approach, leaving Et0/1, Et0/3, Et1/0, Et1/2, and Et6/0 active:

```
Cafe-SW2(config)# interface range ethernet 0/2, ethernet 1/1, ethernet 1/3, ethernet 2/0-3, ethernet 3/0-3, ethernet 4/0-3, ethernet 5/0-3, ethernet 6/1-3
Cafe-SW2(config-if-range)# shutdown
```

Post-lockdown on Cafe-SW2 confirmed via `show ip interface brief`:
```
Ethernet0/2    unassigned    YES unset  administratively down  down
Ethernet1/1    unassigned    YES unset  administratively down  down
Ethernet6/0    unassigned    YES unset  up                     up      ← Plex port, still live
```

---

## Mistakes Made

### 1. `switchport trunk allowed vlan 10 20` — syntax error
Tried to separate VLAN IDs with a space. IOS requires comma-separated values with no spaces between the command and the list.

```
❌  switchport trunk allowed vlan 10 20
✅  switchport trunk allowed vlan 10,20
```

### 2. IOL interface range across slots doesn't work with a dash
Attempted `interface range ethernet 1/1 - 6/3` expecting it to span all slots. IOL requires each slot to be listed separately.

```
❌  interface range ethernet 1/1 - 6/3
✅  interface range ethernet 1/1-3, ethernet 2/0-3, ethernet 3/0-3, ...
```

### 3. First ping showed 66% packet loss — left it there initially
The first ping to 10.0.18.1 lost most packets. Root cause: ARP. The Plex server had no MAC entry for the gateway and had to resolve it via ARP broadcast before ICMP could flow. The high latency on seq=12 (2029ms) was the ARP resolution completing mid-ping. Second ping hit clean from seq=0 because the ARP cache was already populated. Always run a second ping to confirm — a lossy first ping after a port change is expected behavior, not a misconfiguration.

---

## Verification Summary

| Check | Result |
|---|---|
| Cafe-SW1 trunks (Et0/0, Et0/1, Et1/0) | Trunking, VLANs 10,20 allowed and active |
| Cafe-SW2 trunks (Et0/1, Et1/0) | Trunking, VLANs 10,20 allowed and active |
| Cafe-SW2 Et6/0 | Access port, VLAN 10, Plex server reachable |
| Plex ping to 10.0.18.1 (second attempt) | 5/5 packets, 0% loss, ~1ms RTT |
| Unused ports both switches | Administratively down |

---

## Key Takeaways

- **Subinterfaces inherit state from the parent.** A pre-configured subinterface does nothing until the parent physical interface is brought up with `no shutdown`.
- **IOL `interface range` syntax is slot-strict.** You cannot span across slot boundaries with a single dash. List each slot explicitly.
- **ARP explains the lossy first ping.** After a port moves to a new VLAN, the first ping will often lose packets while ARP resolves the gateway MAC. Run a second ping before declaring a problem.
- **VLAN 1 is a dead end in a router-on-a-stick design.** If no subinterface exists for VLAN 1, anything left there has no routed path — correct IP address or not.
