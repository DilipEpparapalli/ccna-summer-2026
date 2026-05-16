
### Simple Explanation

Ethernet is not the cable. Ethernet is a **communication standard**, a shared set of rules that allows devices from different manufacturers to talk to each other over a network. The cable is the road; Ethernet is the traffic law.

### Why It Exists

Without a standard, every vendor would invent their own communication method. A Cisco switch couldn't talk to a Dell server. An HP laptop couldn't connect to a Netgear AP. Standards created interoperability — the reason the entire internet scales.

> Ethernet was invented at Xerox PARC in the 1970s, then standardized by the **IEEE** (Institute of Electrical and Electronics Engineers) as **IEEE 802.3**.

### How It Works

Ethernet defines two critical things at Layer 2:

#### 1. MAC Addresses

- **Media Access Control** address
- **48-bit** hardware identifier assigned to every NIC (Network Interface Card)
- Written as 12 hexadecimal characters
- Two common notations:
    - `AA:BB:CC:11:22:33` — Linux/Unix style (colons)
    - `AABB.CC11.2233` — Cisco style (dotted groups of 4)
- Split into two halves:

```
[ OUI - 24 bits ]  [ Device ID - 24 bits ]
  AA:BB:CC           11:22:33
  ↑ Manufacturer     ↑ Assigned by manufacturer to this specific NIC
```

- **OUI** = Organizationally Unique Identifier — identifies the manufacturer
- **BIA** = Burned-In Address — the MAC hardcoded into the NIC at the factory
- ⚠️ MACs can be **spoofed in software** — a MAC is not a security mechanism

#### 2. Frame Format

Ethernet defines how data is packaged for Layer 2 transmission. A frame wraps the payload with a destination MAC, source MAC, type field, and FCS (error checking). This is what a switch reads to make forwarding decisions.

#### 3. CSMA/CD (Historical — but tested)

Before switches, networks used **hubs**. A hub repeated every signal out every port — dumb broadcast. If two devices transmitted simultaneously, signals collided.

**CSMA/CD** = Carrier Sense Multiple Access with Collision Detection:

1. **Listen** before transmitting (Carrier Sense)
2. **Everyone can transmit** when the line is clear (Multiple Access)
3. **Detect collision** if two transmit simultaneously
4. **Back off** and retransmit after a random wait

Modern **switched** networks eliminated collisions in normal operation — each switch port is its own collision domain. But CSMA/CD still appears on the exam.

#### How a Switch Uses Ethernet Standards

A switch maintains a **CAM table** (Content Addressable Memory) — also called the **MAC address table**:

```
Switch CAM Table:
+--------+-------------------+-----------+
| Port   | MAC Address       | VLAN      |
+--------+-------------------+-----------+
| Gi0/1  | AA:BB:CC:11:22:33 | 1         |
| Gi0/2  | AA:BB:CC:44:55:66 | 1         |
| Gi0/3  | AA:BB:CC:77:88:99 | 10        |
+--------+-------------------+-----------+
```

- Switch **learns** MACs by reading the **source MAC** of incoming frames
- Switch **forwards** frames by looking up the **destination MAC**
- If destination MAC is unknown → switch **floods** out all ports (except the one it came in on)
- If destination MAC is known → switch **unicasts** to the correct port only

**Why this matters:** Without this intelligence, every frame would go everywhere — exactly like a hub. That's wasted bandwidth and wasted CPU on every device that has to reject frames not meant for it.

### Key Terms

|Term|Meaning|
|---|---|
|IEEE 802.3|The official Ethernet standard|
|MAC Address|48-bit Layer 2 hardware identifier|
|OUI|First 24 bits of MAC — identifies manufacturer|
|BIA|Burned-In Address — factory-assigned MAC|
|CAM Table|Switch's MAC-to-port mapping table|
|CSMA/CD|Collision avoidance/detection method used with hubs|
|Collision Domain|Network segment where collisions can occur|
|Unicast|Frame sent to one specific destination|
|Flood|Frame sent out all ports when destination MAC is unknown|
|Frame|Layer 2 unit of data — wraps payload with MAC headers|

### Real-World Connection

Every switch port in an enterprise network is its own collision domain. A 48-port switch has 48 collision domains — no device competes with any other for the wire. The CAM table is constantly being updated as devices come and go. In large environments, MAC tables can hold tens of thousands of entries.

### Exam Traps

1. **Ethernet ≠ the cable.** If asked "what is Ethernet?", the answer is a communication standard (IEEE 802.3), not twisted pair cabling.
2. **Switch floods unknown unicasts** — not just broadcasts. If the CAM table doesn't have the destination MAC, it floods. This catches people off guard.
3. **MAC spoofing is possible** — MACs are not authentication. Don't confuse "burned in" with "immutable."

### Commands (if applicable)

```
! View the MAC address table on a Cisco switch
show mac address-table

! View MAC address table for a specific VLAN
show mac address-table vlan 10

! View MAC address table for a specific interface
show mac address-table interface gigabitEthernet 0/1

! Clear the MAC address table (forces relearning)
clear mac address-table dynamic
```

