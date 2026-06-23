## Simple Explanation
A VLAN (Virtual LAN) lets you take one physical switch and divide it into multiple
logical networks. Devices in different VLANs can't talk to each other without a
router — even if they're plugged into the same physical switch. Same hardware,
multiple isolated networks.

## Why It Exists

Out of the box, a switch is one giant broadcast domain. Every device hears every
broadcast from every other device. That creates three real problems as a network grows:

- **Noise** — broadcasts from 200 devices hit every endpoint, wasting CPU cycles on
  devices that don't care
- **Security** — guest laptops, payment terminals, cameras, and employee PCs all share
  the same network space by default
- **Scale** — a flat network has no structure to grow into; adding devices makes the
  problem worse

VLANs solve all three by creating boundaries. Each VLAN is its own broadcast domain,
its own IP subnet, and its own security zone — using the same physical switches you
already have.

## How It Works

### Broadcast Domains and VLANs

A default 24-port switch has one broadcast domain. A broadcast sent by any device
floods out all 23 other ports — every device hears it.

When you create VLANs, you assign ports to them:

```
Ports 1–8   → VLAN 10 (Sales)
Ports 9–16  → VLAN 20 (Guest)
Ports 17–24 → VLAN 30 (Cameras)
```

Now a broadcast from a Sales device only floods ports 1–8. Guest and Camera devices
never hear it. You've created three logical switches inside one physical switch.

```
Before VLANs:                    After VLANs:
┌─────────────────────┐          ┌──────────┐ ┌──────────┐ ┌──────────┐
│  24-port switch     │          │  VLAN 10 │ │  VLAN 20 │ │  VLAN 30 │
│  1 broadcast domain │   →      │  Sales   │ │  Guest   │ │  Cameras │
│  everyone hears     │          │  8 ports │ │  8 ports │ │  8 ports │
│  everything         │          └──────────┘ └──────────┘ └──────────┘
└─────────────────────┘          3 broadcast domains, 1 physical switch
```

### The VLAN-to-Subnet Relationship

Each VLAN maps to its own IP subnet. They are not the same thing, but they go
together 1:1 in practice:

| VLAN | Name    | Subnet             |
|------|---------|--------------------|
| 10   | Sales   | 192.168.10.0/24    |
| 20   | Guest   | 192.168.20.0/24    |
| 30   | Cameras | 192.168.30.0/24    |

Because devices in different VLANs are in different subnets, they cannot communicate
without a router. This is intentional — the separation is the point.

### Access Ports vs Trunk Ports

There are two kinds of switch ports and they handle frames completely differently:

**Access Port** — connects to an end device (PC, phone, printer)
- Carries traffic for exactly **one VLAN**
- Receives an untagged frame from the end device → **adds** the VLAN tag before
  sending it toward the trunk
- Receives a tagged frame from the trunk → **strips** the tag before delivering to
  the end device
- The end device never sees a tag. Ever.

**Trunk Port** — connects switch-to-switch, or switch-to-router
- Carries traffic for **multiple VLANs** simultaneously
- Passes frames with their 802.1Q tags **intact**
- Does not add or strip tags — it just forwards tagged frames to the next device

```
End Device                Switch                        Switch
(no tags)                 Access Port    Trunk Port     Trunk Port    Access Port
                          adds tag  →    carries tag →  receives tag  strips tag
[PC - VLAN 10] ────────→ [tag: VLAN 10] ──────────────────────────→ [PC - VLAN 10]
                                         802.1Q trunk
```

### 802.1Q Tagging — How the Switch Knows Where a Frame Belongs

When a frame crosses a trunk link, the sending switch inserts a **4-byte 802.1Q tag**
into the frame header. This tag carries one critical piece of information: the VLAN ID.

The tag is a **membership label** — it answers one question only:
> "Which VLAN does this frame belong to?"

It is not a source address. It is not a destination address. It is purely a label
that tells the receiving switch which broadcast domain this frame lives in.

```
Normal frame:   [Dest MAC | Src MAC | Type | Payload]
Tagged frame:   [Dest MAC | Src MAC | 802.1Q Tag | Type | Payload]
                                      └── 4 bytes ──┘
                                          VLAN ID lives here
```

**Exact sequence for a frame crossing two switches:**

```
1. PC-A sends an untagged frame (PC knows nothing about VLANs)
2. Switch1 access port receives it → sees the port is in VLAN 10
3. Switch1 adds 802.1Q tag: "VLAN 10"
4. Tagged frame travels across the trunk link to Switch2
5. Switch2 trunk port receives the tagged frame
6. Switch2 reads the tag → "this is a VLAN 10 frame"
7. Switch2 strips the tag
8. Switch2 delivers the untagged frame out its VLAN 10 access port to PC-B
```

802.1Q is an industry standard — Cisco, Dell, Ubiquiti, and every other vendor that
supports it can exchange tagged frames across a trunk. It is not vendor-specific.

### Routing Between VLANs — Router on a Stick

VLANs isolate traffic by design. When you need cross-VLAN communication — Guest
reaching the internet, Sales reaching a server in a different VLAN — you need a
router.

**Router-on-a-stick** is the common solution for smaller environments:

- One trunk link runs from the switch to a single router interface
- The router creates **subinterfaces** — one per VLAN — on that physical interface
- Each subinterface needs two things configured:
  1. `encapsulation dot1Q <vlan-id>` — tells the subinterface which tagged traffic
     belongs to it
  2. An IP address — becomes the default gateway for that VLAN

```
                    Trunk link (carries all VLANs)
Switch ────────────────────────────────→ Router
                                         ├── Gi0/0.10  encap dot1Q 10
                                         │   ip addr 192.168.10.1 /24
                                         │   (gateway for VLAN 10)
                                         │
                                         └── Gi0/0.20  encap dot1Q 20
                                             ip addr 192.168.20.1 /24
                                             (gateway for VLAN 20)
```

When a VLAN 10 device sends traffic to a VLAN 20 device, the frame goes to the
router (its default gateway), the router evaluates it, and either forwards it to
VLAN 20, blocks it, or sends it to the internet. The router is where policy lives.

## Key Terms

| Term | Meaning |
|------|---------|
| VLAN | A logical broadcast domain created on a switch; one VLAN = one broadcast domain |
| Broadcast domain | The area a broadcast can reach; VLANs create boundaries for these |
| Access port | Switch port assigned to one VLAN; adds/strips tags; connects to end devices |
| Trunk port | Switch port carrying multiple VLANs; passes tagged frames intact |
| 802.1Q | Industry standard for VLAN tagging; inserts a 4-byte tag into the frame |
| VLAN tag | 4-byte membership label on a frame that identifies which VLAN it belongs to |
| Subinterface | Logical division of a physical router interface; one per VLAN in RoaS |
| Router-on-a-stick | Inter-VLAN routing design using one trunk link and subinterfaces |
| Encapsulation dot1Q | IOS command on a subinterface that binds it to a specific VLAN tag |
| Inter-VLAN routing | Using a router or Layer 3 device to move traffic between VLANs |

## Real-World Connection

In enterprise networks, VLANs are one of the first design decisions made when
building out a site. A typical branch office might have:

- VLAN 10 — Employee data (laptops, desktops)
- VLAN 20 — VoIP phones
- VLAN 30 — Guest WiFi
- VLAN 40 — Security cameras
- VLAN 99 — Management (switch SVIs, router management interfaces)

The physical switch infrastructure is shared across all of them. The VLANs provide
the logical separation. This is why VLANs appear early in network design — they
define the security and routing boundaries everything else is built around.

## Exam Traps

1. **VLANs and subnets are not the same thing** — a VLAN is a Layer 2 broadcast
   domain; a subnet is a Layer 3 address space. They map 1:1 in practice but are
   distinct concepts. The exam will test whether you know which layer each lives on.

2. **End devices never see 802.1Q tags** — tags exist only on trunk links. Access
   ports add tags toward the trunk and strip them toward the end device. If a question
   implies an end device is tag-aware, that's a red flag.

3. **Missing `encapsulation dot1Q` on a subinterface breaks everything** — the
   subinterface won't know which tagged traffic belongs to it. The IP address alone
   is not enough. Both commands are required.

4. **A trunk port does not strip tags** — it forwards frames with tags intact. Only
   access ports strip tags. Confusing these two is a common source of wrong answers
   on switching questions.

5. **VLANs don't route — routers route** — devices in different VLANs cannot reach
   each other without a Layer 3 device. A switch alone, no matter how many VLANs it
   has, cannot move traffic between them.

## Commands

```
! Create a VLAN and name it
vlan 10
 name Sales

! Assign an access port to a VLAN
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10

! Configure a trunk port
interface FastEthernet0/24
 switchport mode trunk

! Verify VLANs and port assignments
show vlan brief

! Verify trunk status
show interfaces trunk

! Router-on-a-stick subinterface config
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0

! Bring up the physical interface (subinterfaces won't work if this is down)
interface GigabitEthernet0/0
 no shutdown
```

## Recall Questions

1. A switch has 24 ports, all defaults. How many broadcast domains exist? What
   changes if you create three VLANs and assign 8 ports to each?
2. A frame leaves PC-A on VLAN 10 heading to PC-B on VLAN 10 across two switches.
   Trace exactly what happens to the 802.1Q tag at each step.
3. What is the VLAN tag and what single question does it answer? What is it NOT?
4. What two things must be configured on a router subinterface for router-on-a-stick
   to work, and what breaks if either is missing?
5. A device in VLAN 10 can't reach a device in VLAN 20. The VLANs are configured
   correctly on the switch. What's the most likely cause?
```
