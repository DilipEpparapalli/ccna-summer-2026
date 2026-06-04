## Task 0 — Label the Active Ports

Applied descriptions to active ports on both switches so field staff can identify circuits
without tracing cables.

### Cafe-SW1

```
Cafe-SW1(config)# interface ethernet0/0
Cafe-SW1(config-if)# description ## Cafe-SW2 Uplink
Cafe-SW1(config-if)# interface ethernet0/1
Cafe-SW1(config-if)# description ## BaristaPOS
```

### Cafe-SW2

```
Cafe-SW2(config)# interface ethernet0/0
Cafe-SW2(config-if)# description ## Cafe-SW1 Uplink
Cafe-SW2(config-if)# interface ethernet0/1
Cafe-SW2(config-if)# description ## Cam01
Cafe-SW2(config-if)# interface ethernet0/3
Cafe-SW2(config-if)# description ## Thermostat
```

### Verification — `show interfaces status`

**Cafe-SW1:**
```
Port    Name               Status     Vlan  Duplex  Speed
Et0/0   ## Cafe-SW2 Uplink connected  1     full    auto
Et0/1   ## BaristaPOS      connected  1     full    auto
```

**Cafe-SW2:**
```
Port    Name               Status     Vlan  Duplex  Speed
Et0/0   ## Cafe-SW1 Uplink connected  1     full    auto
Et0/1   ## Cam01           connected  1     full    auto
Et0/3   ## Thermostat      connected  1     full    auto
```

Descriptions confirmed on correct ports. Et2 on both switches had no description — not part of
the labeled circuit plan for this deployment.

---

## Task 1 — Normalize Duplex Settings

Hardcoded duplex to full on the uplink and IoT drop ports to eliminate any auto-negotiation
risk. Speed was left on auto (fixed by the IOL platform).

### Cafe-SW1

```
Cafe-SW1(config)# interface ethernet0/0
Cafe-SW1(config-if)# duplex full
```

### Cafe-SW2

```
Cafe-SW2(config)# interface ethernet0/0
Cafe-SW2(config-if)# duplex full
Cafe-SW2(config)# interface ethernet0/1
Cafe-SW2(config-if)# duplex full
Cafe-SW2(config)# interface ethernet0/3
Cafe-SW2(config-if)# duplex full
```

### Verification — filtered `show interfaces`

```
Cafe-SW1# show interface ethernet0/0 | include line|duplex|coll
Ethernet0/0 is up, line protocol is up (connected)
  Full-duplex, Auto-speed
  0 output errors, 0 collisions, 1 interface resets
  0 babbles, 0 late collision, 0 deferred
```

```
Cafe-SW2# show interface ethernet0/1 | include line|duplex|coll
Ethernet0/1 is up, line protocol is up (connected)
  Full-duplex, Auto-speed
  0 output errors, 0 collisions, 1 interface resets
  0 babbles, 0 late collision, 0 deferred
```

**Collisions: 0. Late collisions: 0.** Both sides running full duplex — no mismatch.

The 1 interface reset visible on each port is from the duplex change causing a brief link bounce.
This is expected behavior when duplex is reconfigured mid-operation.

---

## Task 2 — Assign Management IPs

Put both switches on VLAN 42 (192.168.42.0/24) and verified end-to-end reachability.

### Sequence that matters

The SVI (`interface vlan 42`) will not come up unless:
1. VLAN 42 exists in the local VLAN database
2. At least one port is active and carrying VLAN 42 (via trunk or access)

On Cafe-SW1 I initially ran `no shutdown` on the SVI before completing the trunk configuration —
the SVI stayed `down/down`. Once the trunk was carrying VLAN 42 and the VLAN existed in the
database, the `no shutdown` took effect.

### Cafe-SW1

```
! Step 1 — Create VLAN in database
Cafe-SW1(config)# vlan 42

! Step 2 — Configure uplink as trunk carrying VLAN 42
Cafe-SW1(config)# interface ethernet0/0
Cafe-SW1(config-if)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if)# switchport mode trunk
Cafe-SW1(config-if)# switchport trunk allowed vlan add 42

! Step 3 — Assign IP to SVI and bring it up
Cafe-SW1(config)# interface vlan 42
Cafe-SW1(config-if)# ip address 192.168.42.1 255.255.255.0
Cafe-SW1(config-if)# no shutdown

! Step 4 — Set default gateway for management traffic
Cafe-SW1(config)# ip default-gateway 192.168.42.254
```

### Cafe-SW2

```
Cafe-SW2(config)# vlan 42
Cafe-SW2(config)# interface ethernet0/0
Cafe-SW2(config-if)# switchport trunk encapsulation dot1q
Cafe-SW2(config-if)# switchport mode trunk
Cafe-SW2(config-if)# switchport trunk allowed vlan add 42
Cafe-SW2(config)# interface vlan 42
Cafe-SW2(config-if)# ip address 192.168.42.2 255.255.255.0
Cafe-SW2(config-if)# no shutdown
Cafe-SW2(config)# ip default-gateway 192.168.42.254
```

### Verification

```
Cafe-SW2# show ip interface brief | include Vlan42
Vlan42    192.168.42.2    YES manual up    up

Cafe-SW1# show ip interface brief | include Vlan42
Vlan42    192.168.42.1    YES manual up    up
```

```
Cafe-SW2# ping 192.168.42.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms

Cafe-SW1# ping 192.168.42.2
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/3 ms
```

Both SVIs up/up. Bidirectional ping succeeds. Management reachability confirmed.

---

## What This Proved

- Interface descriptions appear in `show interfaces status` and act as a cable map —
  no physical tracing needed
- Hardcoding duplex eliminates auto-negotiation risk; 0 collisions confirms no mismatch
- The pipe filter (`| include`) is the right tool for pulling specific counters out of
  verbose `show interfaces` output
- An SVI will not come up until the VLAN exists in the database **and** an active port
  is carrying that VLAN — order of operations matters
- `switchport trunk allowed vlan add 42` adds VLAN 42 to the allowed list without
  wiping other VLANs; using `switchport trunk allowed vlan 42` (no `add`) would
  replace the entire allowed list with only VLAN 42
