## The Big Picture

A switch is not just a traffic forwarder. It is a detective tool.
Every device that connects to a switch leaves evidence — and if you know how to read it,
you can find any device on the network, trace any problem, and fix it fast.

Before routers, firewalls, or VLANs mean anything, this foundation has to be solid.

---

## MAC Addresses

### What Is a MAC Address?

A **MAC address** (Media Access Control) is the hardware identity of a network interface.
Every NIC — laptop, printer, phone, camera, access point — has one burned in at the factory.

- Also called: **physical address**, **hardware address**, **Layer 2 address**
- 48 bits long, written as 12 hexadecimal characters
- Hexadecimal uses digits 0–9 and letters A–F

**How it looks:**

```
Windows:    00-1A-2B-3C-4D-5E
Linux/Mac:  00:1A:2B:3C:4D:5E
Cisco IOS:  001a.2b3c.4d5e
```
Same address, three different formats. The value is identical.

**How to find it:**
```
Windows:   ipconfig /all
Linux/Mac: ip link   or   ifconfig
Cisco:     show interfaces   or   show arp
```

---

### OUI — The First Half Tells a Story

A MAC address is split into two 24-bit halves:

```
[ OUI — first 24 bits ]  [ Device ID — last 24 bits ]
   00:1A:2B                    3C:4D:5E
   "Who made it"               "Which one"
```

**OUI = Organizationally Unique Identifier**
Assigned by IEEE to manufacturers. Every vendor owns one or more OUIs.

| OUI Prefix | Vendor |
|------------|--------|
| `00:50:56` | VMware |
| `B8:27:EB` | Raspberry Pi Foundation |
| `00:1A:2B` | (example — look yours up) |

> **Real-world use:** Unknown device on the network? Grab its MAC, look up the OUI.
> If it comes back Roku on a business LAN — that's a problem to investigate.
> If it comes back Ubiquiti — probably a camera or AP. Narrows the hunt immediately.

---

## Frames — The Layer 2 Wrapper

### What Is a Frame?

When your laptop sends data, it travels *down* the OSI model through **encapsulation** —
each layer wraps the data with its own addressing and control information.

By the time it hits **Layer 2 (Data Link)**, the device builds a **frame**:

```
+------------------+------------------+----------+-----+
| Destination MAC  |   Source MAC     |   Data   | FCS |
|   (6 bytes)      |   (6 bytes)      |          |     |
+------------------+------------------+----------+-----+
```

| Field | Purpose |
|-------|---------|
| Destination MAC | Where this frame is going *on the local network* |
| Source MAC | Who sent it |
| Data | The payload (IP packet lives inside here) |
| FCS / CRC | Frame Check Sequence — detects corruption in transit |

### IP vs MAC — Both Are Always in Play

This is the concept that trips most people up:

> **IP address** = end-to-end logical destination (Layer 3)
> **MAC address** = local delivery label for this one hop (Layer 2)

Both are present in every transmission. They operate at different layers simultaneously.

```
Laptop → Pi-hole DNS query

Layer 3 (IP):   Source: 192.168.1.50    Dest: 192.168.1.2   ← stays the same end to end
Layer 2 (MAC):  Source: AA:BB:CC:11:22:33   Dest: DD:EE:FF:44:55:66   ← changes at each hop
```

The laptop knows Pi-hole's *IP* address. But to build the frame, it needs Pi-hole's *MAC* address.
That's where **ARP** comes in.

---

## ARP — The Bridge Between Layer 3 and Layer 2

**ARP = Address Resolution Protocol**

When a device knows a destination IP but not its MAC address, it broadcasts:
> "Who has 192.168.1.2? Tell 192.168.1.50."

The owner of that IP replies with its MAC address. Now the sender can build the frame.

```
[Laptop]  →  ARP Request (broadcast to FF:FF:FF:FF:FF:FF)
             "Who has 192.168.1.2?"

[Pi-hole] →  ARP Reply (unicast back to laptop)
             "192.168.1.2 is at DD:EE:FF:44:55:66"

[Laptop]  →  Now builds frame with correct destination MAC
             Sends DNS query
```

The resolved mapping is cached in the **ARP table** so it doesn't broadcast every time.

```
# View ARP table
Windows:   arp -a
Linux:     arp -n   or   ip neigh
Cisco:     show arp
```

> **Key insight:** The switch never sees the ARP conversation as anything special.
> It just sees frames — and it learns from every single one.

---

## How a Switch Thinks — The CAM Table

### The Switch's Job

A switch connects devices on a LAN and forwards frames intelligently.
It does **not** look at IP addresses. It only reads **Layer 2 — MAC addresses**.

The switch's entire decision-making process lives in one place:

### The CAM Table (MAC Address Table)

**CAM = Content Addressable Memory**

The switch builds a table that maps MAC addresses to physical ports.
It learns this automatically — no manual config required.

```
DS-07-SW1# show mac address-table

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
   1    aa:bb:cc:11:22:33  DYNAMIC    Et0/1
   1    dd:ee:ff:44:55:66  DYNAMIC    Et0/2
   1    00:1a:2b:3c:4d:5e  DYNAMIC    Et0/3
```

### How the Switch Learns (Self-Learning)

```
Step 1 — Frame arrives on port Et0/1
Step 2 — Switch reads the SOURCE MAC address
Step 3 — Records: "AA:BB:CC:11:22:33 is reachable via Et0/1"
Step 4 — Looks up DESTINATION MAC in the CAM table
Step 5 — Forwards frame out the correct port (or floods if unknown)
```

No administrator intervention. No configuration. Fully automatic.

### Three Forwarding Decisions

| Situation | Action |
|-----------|--------|
| Destination MAC is in the table | **Forward** — send out that specific port only |
| Destination MAC is NOT in the table | **Flood** — send out all ports except the one it came in on |
| Source MAC = Destination MAC (same port) | **Filter** — drop it, device is talking to itself |

> **Flooding is normal, not a problem** — it's how the switch learns.
> Once the destination device replies, its MAC gets added to the table and flooding stops.

### CAM Table Aging

Entries don't live forever. Default aging timer: **300 seconds (5 minutes)**.
If no frame arrives from a MAC within that window, the entry is removed.
Next frame from that device triggers the learning process again.

---

## Finding Devices on the Network

This is the job-saving skill. When something is "causing issues" and you need to find it:

### Step 1 — Get the MAC address of the device
```
# If you have the IP, use ARP to resolve it
show arp | include 192.168.1.50
```

### Step 2 — Find which switch port it's on
```
show mac address-table | include aa:bb:cc:11:22:33
```

### Step 3 — Trace upward if needed
If the port leads to another switch (trunk), log into that switch and repeat.
Eventually you'll hit the access port where the device physically connects.

### Step 4 — Now you know
- Which switch
- Which port
- Which cable
- Which desk/room/wall jack

A vague "network feels slow" complaint just became a physical location.

---

## ASCII Topology — What the Switch Sees

```
                    [DS-07-SW1]
                   /     |     \
                  /      |      \
             Et0/1    Et0/2    Et0/3
               |        |        |
           [Laptop]  [Printer] [Camera]
         AA:BB:CC:   DD:EE:FF:  00:1A:2B:
         11:22:33    44:55:66   3C:4D:5E

CAM Table after all three send a frame:
  Et0/1 → AA:BB:CC:11:22:33
  Et0/2 → DD:EE:FF:44:55:66
  Et0/3 → 00:1A:2B:3C:4D:5E

Laptop sends to Printer:
  Dest MAC = DD:EE:FF:44:55:66
  Switch looks up table → Et0/2
  Forwards ONLY to Et0/2
  Camera on Et0/3 sees nothing
```

---

## Key Terms

| Term | Meaning |
|------|---------|
| MAC Address | 48-bit hardware identity of a network interface |
| OUI | First 24 bits of MAC — identifies the manufacturer |
| Frame | Layer 2 data unit — includes src/dest MAC and FCS |
| FCS / CRC | Frame Check Sequence — detects corruption |
| Encapsulation | Each OSI layer wraps data with its own header |
| ARP | Protocol that resolves IP addresses to MAC addresses |
| CAM Table | Switch's MAC-to-port mapping table |
| Flooding | Forwarding a frame out all ports when destination MAC is unknown |
| Aging Timer | How long a CAM entry lives without activity (default 300s) |

---

## Exam Traps

1. **Switches forward based on MAC, not IP.**
   A switch has no idea what IP address is inside the frame. That's Layer 3.
   If a question asks "what does the switch use to forward traffic" — MAC address, always.

2. **Flooding is not broadcasting.**
   Flooding is what happens when the destination MAC is unknown — the switch sends
   out all ports except the incoming one. Broadcasting is a deliberate all-destinations
   transmission (like ARP requests). They look similar but mean different things.

3. **ARP is required before the first frame can be sent.**
   A device can't build a frame without a destination MAC. If it only knows the IP,
   it must ARP first. This is why the first ping to a new device is sometimes slower.

4. **MAC addresses change at each Layer 3 hop — IP addresses don't.**
   Across a routed network, the IP source/destination stays the same end to end.
   The MAC addresses are re-written at every router hop.
   This is one of the most tested concepts in CCNA.

5. **The CAM table is built from SOURCE MAC addresses, not destination.**
   The switch learns where devices live by reading who is *sending*, not who is
   being sent to. This is how self-learning works.

6. **Unknown unicast = flood. Known unicast = forward. Same subnet broadcast = flood.**
   Three different triggers, three similar outcomes. Know the difference.

---

## Useful Commands

```
# View the MAC address table
show mac address-table

# Filter for a specific MAC
show mac address-table | include aa:bb:cc:11:22:33

# View ARP table (on router or Layer 3 switch)
show arp

# Clear the MAC address table (forces re-learning)
clear mac address-table dynamic

# Check interface details including MAC
show interfaces Ethernet0/1

# Find MAC on a Windows host
ipconfig /all

# Find MAC on Linux
ip link show
```

---

## Recall Questions

1. What are the two halves of a MAC address and what does each represent?
2. A device knows the IP address of its destination but not the MAC. What happens before the first frame is sent?
3. A frame arrives at a switch with a destination MAC not in the CAM table. What does the switch do, and why?
4. What is the difference between flooding and broadcasting?
5. Your laptop sends a packet to a web server on the internet. At the router, what happens to the source/destination IP? What happens to the source/destination MAC?
6. What command shows you which port a specific MAC address is connected to on a Cisco switch?
7. Why does the switch learn from SOURCE MAC addresses and not destination MAC addresses?
8. You see an unknown device on the network. Walk through the steps to physically locate it.
