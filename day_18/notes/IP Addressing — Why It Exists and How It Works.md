
## Simple Explanation

Every device on a network needs two kinds of addresses doing two different jobs simultaneously. MAC addresses handle local delivery — getting a frame to the next hop. IP addresses handle end-to-end delivery — identifying the final destination across any number of networks. You can't do one job with the other's tool.

## Why MAC Addresses Alone Don't Scale

Switches forward frames using MAC addresses, and that works beautifully inside a local network. The problem is MAC addresses have no hierarchy — they're flat unique identifiers that say *what* a device is, not *where* it is. There's no location information encoded in a MAC address, so a router has no way to build a map of where things are or how to reach them.

Try to run a global network on MAC addresses alone and two things collapse immediately. First, every device trying to find another device has to broadcast to the entire network — and with no routers stopping those broadcasts, they propagate everywhere. The network drowns in its own noise. Second, even if the broadcasts were manageable, there's no way to build routing tables from MAC addresses. Routing requires hierarchy: "this range of addresses lives over there." MAC addresses can't provide that.

IP addresses solve both problems. They encode network location (which neighborhood a device lives in), and they give routers the structure needed to build maps and make forwarding decisions.

## The Two Layers Working Together

When a laptop sends a packet to a server on a different network, both address types are active at the same time doing separate jobs:

- The **IP address** carries the identity of the final destination and doesn't change for the entire journey from source to destination.
- The **MAC address** carries the identity of the *next hop* — whoever needs to receive this frame right now. It gets rewritten at every router along the path.

So locally, the switch looks at the MAC address and delivers the frame to the router. The router opens it, reads the IP address, rewrites the MAC address to point at the *next* router (or the final destination), and forwards it again. This repeats until the packet arrives. Two address types, two jobs, running simultaneously — neither replaces the other.

## The Default Gateway

When a device wants to reach something outside its own network, it compares the destination IP address against its own subnet. If the destination is on a different network, the device knows it can't deliver directly — it needs to hand the packet to the router.

The default gateway is the router's local IP address — the exit door out of the local network. The device sends the frame to the router's MAC address (local delivery), while the packet inside still carries the remote server's IP address (end-to-end destination). The router then takes over and forwards it toward the destination network.

Routers do two critical things here: they connect different networks together, and they stop broadcasts. ARP requests and other broadcasts never cross a router — they die at the boundary. That's why a broadcast storm on one network doesn't take down every network connected to it.

## Key Terms

| Term            | Meaning                                                                                                                           |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| MAC address     | Physical address burned into a NIC; identifies a device on the local network; changes at each hop                                 |
| IP address      | Logical address assigned to a device; identifies the final destination; stays constant end-to-end                                 |
| Default gateway | The router's local IP address; the exit point out of a local network                                                              |
| Broadcast       | A frame or packet sent to all devices on a network segment; stopped by routers                                                    |
| ARP             | Address Resolution Protocol; maps an IP address to a MAC address on the local network; broadcast-based and does not cross routers |
| Subnet mask     | Tells a device which part of an IP address is the network and which part is the host                                              |

## Real-World Connection

On a coffee shop network with registers, laptops, cameras, and a back-office server — every device needs an IP address the moment it needs to talk to anything outside its immediate local segment. A POS register talking to a payment gateway is crossing networks; that journey needs IP addresses to navigate and MAC addresses to move frame by frame. If either layer breaks, the transaction fails. Troubleshooting starts with three checks: IP address, subnet mask, default gateway. If any of those are wrong, nothing else matters.

## Exam Traps

**MAC addresses change at each hop; IP addresses don't.** A common misconception is that MAC addresses travel all the way to the destination. They don't — they're local only and get rewritten at every router.

**ARP does not cross routers.** If a device ARPs for something on a different network, the router drops the broadcast. The device has to send traffic to its default gateway instead and let routing take over.

**Default gateway is not optional.** A device with no default gateway configured can communicate on its local network just fine, but it cannot reach anything outside. This is one of the most common causes of "I can ping my neighbor but not the internet."

**MAC addresses have no hierarchy.** This is why routing with MAC addresses is architecturally impossible, not just inefficient. IP addresses encode location; MAC addresses don't.

## Recall Questions

1. Why can't routers use MAC addresses to build routing tables?
2. What happens to a device's IP address as a packet travels from source to destination across multiple routers? What happens to the MAC address?
3. A device can reach other devices on its local network but cannot reach anything on the internet. What are the three things you check first?
4. Why do routers stop broadcasts, and why does that matter?
