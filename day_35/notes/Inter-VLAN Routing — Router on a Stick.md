## Simple Explanation
VLANs separate traffic by design — devices in different VLANs cannot communicate
without a router. Inter-VLAN routing is the process of using a router to move traffic
between VLANs. Router on a stick (ROAS) does this with a single physical router
interface and multiple logical subinterfaces, one per VLAN, connected to the switch
via a trunk link.

## Why It Exists
VLANs create isolated broadcast domains. That isolation is the point — but it's not
absolute. A device in the admin VLAN still needs to reach the internet. A server in
one VLAN still needs to serve clients in another. Without a router sitting between
them, those conversations are impossible. ROAS solves this without requiring a
separate physical router interface per VLAN.

## How It Works

### Method 1 — One Interface Per VLAN (Legacy, Don't Use)

The brute-force approach: one physical router interface per VLAN, each interface
assigned an IP in that VLAN's subnet.

```
Router
├── Gi0/0 → 10.0.18.1  (VLAN 10 cable)
├── Gi0/1 → 10.0.18.33 (VLAN 20 cable)
└── Gi0/2 → 10.0.18.65 (VLAN 30 cable)
```

It works. It's also expensive, wasteful, and burns router ports fast. With more than
two or three VLANs it becomes impractical. Know it exists for the exam; don't design
it in real life.

### Method 2 — Router on a Stick (ROAS)

One physical interface. One trunk link to the switch. Multiple logical subinterfaces
on the router — one per VLAN. Each subinterface is tagged with 802.1Q so the router
knows which VLAN that subinterface belongs to.

```
                    Trunk link (all VLANs tagged)
Switch ────────────────────────────────── Router Gi0/1
                                           ├── Gi0/1.10  → VLAN 10 gateway
                                           └── Gi0/1.20  → VLAN 20 gateway
```

### Step 1 — Remove IP from the Physical Interface

The physical interface itself doesn't get an IP address in ROAS. The subinterfaces
handle it. If an IP was previously assigned to the physical interface, remove it:

```
interface GigabitEthernet0/1
 no ip address
```

The physical interface must still be up (`no shutdown`) — subinterfaces inherit
their state from the parent physical interface.

### Step 2 — Create Subinterfaces

Subinterfaces are created by appending a dot and a number to the physical interface
name. Convention is to match the subinterface number to the VLAN ID:

```
interface GigabitEthernet0/1.10
interface GigabitEthernet0/1.20
```

The number after the dot doesn't technically have to match the VLAN ID — but making
them different is asking for confusion. Always match them.

### Step 3 — Set Encapsulation Before IP Address

This is the step that trips people up. The router needs to know which VLAN tags to
expect on each subinterface. Without `encapsulation dot1Q`, the router rejects the
IP address assignment entirely:

```
! What happens if you skip encapsulation:
cafe01-RT01(config-subif)# ip address 10.0.18.1 255.255.255.224
% Configuring IP routing on a LAN subinterface is only allowed if that
subinterface is already configured as part of an IEEE 802.10, IEEE 802.1Q,
or ISL vLAN.
```

Correct sequence — encapsulation first, IP address second:

```
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.0.18.1 255.255.255.224

interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 10.0.18.33 255.255.255.224
```

The IP address on each subinterface becomes the **default gateway** for every device
in that VLAN.

### Step 4 — Set the Switch Port to Trunk

This is the most common missed step. The switch port connecting to the router must
be a trunk — otherwise the router receives no VLAN tags and can't route between
VLANs. Access port = only one VLAN = ROAS doesn't work.

```
! On the switch, find the port facing the router and trunk it
interface fastEthernet 0/24
 switchport mode trunk
```

The port will flap briefly when it changes mode — normal behavior.

### Verify Subinterface State

```
cafe01-RT01# show ip interface brief

Interface              IP-Address      OK? Method Status    Protocol
GigabitEthernet0/1     unassigned      YES manual up        up
GigabitEthernet0/1.10  10.0.18.1       YES manual up        up
GigabitEthernet0/1.20  10.0.18.33      YES manual up        up
```

Both subinterfaces up/up with their IPs assigned — ROAS is ready.

### Step 5 — Configure DHCP Per VLAN

Each VLAN gets its own DHCP pool. Devices in that VLAN pull addresses from their
pool and receive the correct default gateway automatically:

```
! Exclude infrastructure addresses first
ip dhcp excluded-address 10.0.18.1 10.0.18.10

! Pool for VLAN 10 (admin)
ip dhcp pool admin-devices
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
 dns-server 1.1.1.1

! Pool for VLAN 20 (patron)
ip dhcp pool patron-devices
 network 10.0.18.32 255.255.255.224
 default-router 10.0.18.33
 dns-server 1.1.1.1
```

Verify leases with:

```
show ip dhcp binding
```

### How Traffic Actually Flows

When a device in VLAN 10 sends traffic to a device in VLAN 20:

```
1. PC-A (VLAN 10, 10.0.18.11) sends packet to 10.0.18.34
2. 10.0.18.34 is not in VLAN 10's subnet → send to default gateway
3. Frame goes to switch, tagged VLAN 10, trunk port to router
4. Router receives frame on Gi0/1.10 (tagged VLAN 10)
5. Router strips tag, routes packet: destination is 10.0.18.34
6. 10.0.18.34 is in the 10.0.18.32/27 subnet → forward out Gi0/1.20
7. Router re-tags frame with VLAN 20, sends back down trunk to switch
8. Switch delivers untagged frame to VLAN 20 access port → PC-B
```

Traffic goes UP to the router and back DOWN to the switch — that's the "stick" in
router on a stick. You can prove this with traceroute, which shows the router's
subinterface IP as the first hop before the destination.

### What a 169.254.x.x Address Means

If a PC shows a 169.254.x.x address, DHCP failed completely — the client sent
requests and heard nothing back. Common causes in a ROAS setup:

```
169.254.x.x on a DHCP client → check in this order:
1. Is the switch port facing the router set to trunk? (most common cause)
2. Is the correct VLAN assigned to the PC's access port?
3. Is the DHCP pool defined for that subnet?
4. Is the default-router in the pool set correctly?
```

In this lab, the PC grabbed a 169.254 address because the switch port to the router
was still an access port. One `switchport mode trunk` command fixed it.

### Proof — Inter-VLAN Ping

```
! PC in VLAN 10 (10.0.18.11) pinging PC in VLAN 20 (10.0.18.34)
C:\> ping 10.0.18.34

Reply from 10.0.18.34: bytes=32 time<1ms TTL=127
Reply from 10.0.18.34: bytes=32 time=11ms TTL=127
Reply from 10.0.18.34: bytes=32 time<1ms TTL=127
Reply from 10.0.18.34: bytes=32 time<1ms TTL=127
```

TTL=127 confirms the packet was routed (started at 128, decremented by 1 at the
router hop). A same-VLAN ping would show TTL=128 — no router involved.

## Key Terms

| Term | Meaning |
|------|---------|
| Inter-VLAN routing | Using a router or Layer 3 device to move traffic between VLANs |
| Router on a stick (ROAS) | Inter-VLAN routing design using one trunk link and subinterfaces |
| Subinterface | Logical division of a physical interface; created with `interface Gi0/1.10` syntax |
| `encapsulation dot1Q` | Binds a subinterface to a specific VLAN tag; must be set before IP address |
| Default gateway | The router IP that devices use to leave their local subnet |
| 169.254.x.x | APIPA address — device tried DHCP and got no response; signals a broken path |
| DHCP pool | Per-VLAN address range the router hands out automatically |
| TTL | Time to Live; decrements by 1 at each router hop; useful for confirming routing occurred |

## Real-World Connection

Router on a stick is common in small and medium branch environments where you have
one router, one or two switches, and a handful of VLANs. It's also a stepping stone
to understanding Layer 3 switching, where the routing happens inside the switch
itself instead of requiring a separate router. The concepts are identical —
subinterfaces on ROAS become SVIs (Switch Virtual Interfaces) on a Layer 3 switch.
Understanding ROAS deeply makes Layer 3 switching intuitive.

In a real deployment, every VLAN addition requires three things to stay in sync:
a new subinterface on the router, a new DHCP pool, and the correct VLAN on the
switch. Miss any one and devices in that VLAN get 169.254 addresses and no
connectivity.

## Exam Traps

1. **`encapsulation dot1Q` must come before the IP address** — IOS rejects the IP
   assignment on a subinterface if encapsulation hasn't been set. The error message
   is clear but the fix is easy to miss under pressure.

2. **The switch port to the router must be a trunk** — ROAS fails silently if this
   port is left as an access port. Devices pull 169.254 addresses and nothing works.
   This is the most common ROAS misconfiguration.

3. **The physical interface needs no IP but must be up** — subinterfaces inherit
   state from the parent. If `Gi0/1` is shut down, all subinterfaces go down with it.

4. **TTL=127 on a ping means routing happened** — TTL starts at 128 on Windows
   (64 on Linux), decrements by 1 per router hop. If you expect a direct reply
   (TTL=128) but see TTL=127, a router is in the path. Useful for confirming ROAS
   is working or diagnosing unexpected routing.

5. **Subinterface number doesn't have to match VLAN ID — but always match them** —
   technically `Gi0/1.99` could carry VLAN 10 if `encapsulation dot1Q 10` is set.
   In practice, mismatching these creates unnecessary confusion and is a maintenance
   nightmare.

## Commands

```
! Remove IP from physical interface (if previously set)
interface GigabitEthernet0/1
 no ip address

! Create subinterfaces with encapsulation and IP
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.0.18.1 255.255.255.224

interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 10.0.18.33 255.255.255.224

! Set switch port facing router to trunk
interface fastEthernet 0/24
 switchport mode trunk

! DHCP per VLAN
ip dhcp excluded-address 10.0.18.1 10.0.18.10
ip dhcp pool admin-devices
 network 10.0.18.0 255.255.255.224
 default-router 10.0.18.1
 dns-server 1.1.1.1

ip dhcp pool patron-devices
 network 10.0.18.32 255.255.255.224
 default-router 10.0.18.33
 dns-server 1.1.1.1

! Verification
show ip interface brief
show ip dhcp binding
show interfaces trunk
```

## Recall Questions

1. A PC in VLAN 10 pings a PC in VLAN 20 and succeeds. The reply shows TTL=127.
   What does TTL=127 tell you about the path the packet took?
2. You configure ROAS subinterfaces and assign IPs, but PCs are getting 169.254
   addresses. What is the most likely cause and what command fixes it?
3. What happens if you try to assign an IP address to a subinterface before setting
   `encapsulation dot1Q`? What error do you see?
4. Why does the physical router interface in ROAS have no IP address, and what
   happens to the subinterfaces if that physical interface is shut down?
5. Walk through the exact path a frame takes from a PC in VLAN 10 to a PC in VLAN 20
   in a ROAS setup — from the access port to the router and back.
```
