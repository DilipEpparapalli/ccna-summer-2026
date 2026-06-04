## Simple Explanation
Every port on a switch is a configurable interface. Beyond just plugging in a cable, you can label ports with descriptions, lock down speed and duplex settings, and assign the switch a management IP so you can reach it remotely. These aren't optional niceties — they're what separates a network you can operate from one you're just guessing at.

## Why It Exists
A switch with no interface configuration works, but only until something breaks. Without descriptions you're tracing cables by hand. Without speed/duplex control you get silent performance degradation. Without a management IP you're physically walking to every device to configure it. Interface configuration gives you visibility, control, and remote access.

## How It Works

### Interface Descriptions
Descriptions are human-readable labels attached to a port. They show up in `show ip interface brief` and `show interfaces status`, turning "Ethernet0/7" into "WAP-Floor2" or "POS-Terminal-3."

Configure a single port:
```
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Uplink-to-Router
```

Configure a range of ports at once:
```
SW1(config)# interface range FastEthernet0/6 - 10
SW1(config-if-range)# description End-User-Devices
```

`interface range` is the move for bulk work — don't touch 14 ports individually when they all serve
the same purpose.

### Speed and Duplex
**Speed** = link rate (10 / 100 / 1000 Mbps).  
**Duplex** = how traffic flows:
- **Full duplex** — both sides transmit simultaneously (normal on modern switched networks)
- **Half duplex** — one side transmits at a time (walkie-talkie model; legacy gear only)

By default, interfaces use **auto-negotiation** — both sides agree on speed and duplex
automatically. This works fine when both sides are modern. Problems appear when one side is hard coded and the other is on auto, especially on older 10/100 links: the auto side may settle on the correct speed but default to **half duplex**. The link stays up, but performance degrades and you'll see collisions — in a modern switched network, collisions should not exist at all.

To hardcode both values (both sides must match):
```
SW1(config)# interface FastEthernet0/1
SW1(config-if)# speed 100
SW1(config-if)# duplex full
```

> ⚠️ Changing speed or duplex causes the interface to bounce (down → up). Don't do this during business hours unless you're already in an outage.

### Reading Interface Statistics
`show interfaces Ethernet0/1` gives the full counter breakdown for a port. The two counters that matter most for duplex diagnosis:

| Counter              | What it means                                                                                                  |
| -------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Collisions**       | Should be zero on a switched network. Any collisions point to a duplex mismatch or a Layer 1 problem.          |
| **Late collisions**  | Collisions happening after the first 64 bytes of a frame — a strong indicator of duplex mismatch specifically. |
| **Interface resets** | Port is recovering from errors repeatedly. Users experience this as random disconnects.                        |

### Management IP — The SVI
A Layer 2 switch doesn't need an IP address to forward frames — it operates purely on MAC addresses. But to manage it remotely (SSH, Telnet, ping), it needs one.

That IP doesn't go on a physical port. It goes on the **SVI — Switch Virtual Interface** — a
logical interface tied to a VLAN. On most switches, VLAN 1 is the default management VLAN.

```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.0.2 255.255.255.0
SW1(config-if)# no shutdown
```

Verify with:
```
SW1# show ip interface brief
```

The SVI shows up in that output like any other interface. If it shows `up/up`, the switch is
reachable at that IP.

### Useful Verification Commands

| Command                       | What it shows                                      |
| ----------------------------- | -------------------------------------------------- |
| `show ip interface brief`     | All interfaces — IP, status, protocol at a glance  |
| `show interfaces status`      | Speed, duplex, VLAN, description per port          |
| `show interfaces Ethernet0/x` | Full stats for one port — errors, counters, duplex |

## Key Terms

| Term                               | Meaning                                                                                  |
| ---------------------------------- | ---------------------------------------------------------------------------------------- |
| **Interface description**          | Human-readable label on a port; shows in `show` output                                   |
| **interface range**                | Configure multiple ports in one command block                                            |
| **Full duplex**                    | Simultaneous two-way transmission; standard on switched networks                         |
| **Half duplex**                    | One direction at a time; legacy gear or misconfiguration                                 |
| **Duplex mismatch**                | One side full, other half — link stays up but collisions appear and performance degrades |
| **Auto-negotiation**               | Switch and connected device agree on speed/duplex automatically                          |
| **SVI (Switch Virtual Interface)** | Logical interface (e.g., `interface vlan 1`) that holds the switch management IP         |
| **Collision**                      | Two devices transmitting simultaneously; should never occur on a modern switched port    |

## Real-World Connection
Imagine a branch office where the coffee shop's POS terminals started running slow. The link looks up, there are no alarms, but transactions are taking 3x longer than normal. Running `show interfaces` on the switch reveals collisions ticking up on the POS port. The upstream device was hardcoded to full duplex, but the switch port was left on auto and negotiated half. The link was technically up the whole time — but fighting itself on every transmission. Hardcoding both sides to full/100 stops the collisions immediately. That's a duplex mismatch in production.

## Exam Traps

1. **"The link is up so it must be fine."** A duplex mismatch doesn't take a link down — it just degrades performance silently. The interface stays connected. You have to check the collision counters to catch it.

2. **"I'll assign the management IP to a physical port."** You can't. Layer 2 switch ports don't hold IP addresses. The IP goes on the SVI (`interface vlan X`), which is a logical interface.

3. **`interface range` syntax is IOS-version sensitive.** Some versions require a space before the dash, some don't. If the command rejects, check spacing: `FastEthernet0/1 - 5` vs `FastEthernet0/1-5`. When in doubt, test on the device you're actually using.

## Commands

```ios
! -- Describe a single interface --
SW1(config)# interface FastEthernet0/1
SW1(config-if)# description Uplink-to-CoreSwitch

! -- Describe a range of interfaces --
SW1(config)# interface range FastEthernet0/6 - 10
SW1(config-if-range)# description End-User-Devices

! -- Hardcode speed and duplex (both sides must match) --
SW1(config)# interface FastEthernet0/1
SW1(config-if)# speed 100
SW1(config-if)# duplex full

! -- Assign management IP via SVI --
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.0.2 255.255.255.0
SW1(config-if)# no shutdown

! -- Verification --
SW1# show ip interface brief
SW1# show interfaces status
SW1# show interfaces FastEthernet0/1
```

## Recall Questions
1. You see collisions on a switched port. The link is up. What's the most likely cause, and what command do you run to confirm it?
2. Why can't you assign a management IP directly to a physical switch port?
3. You need to apply the same description to ports 11 through 24. What command do you use?
4. What's the difference between `show interfaces status` and `show ip interface brief`?
5. You hardcode a switch port to full/100 but forget to configure the connected device. What happens to the link?
