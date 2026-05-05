# What is a Router?

## Simple Explanation
A router connects different networks together. When your device wants to reach something outside its own local network, it sends the traffic to the router — the router checks its routing table and forwards the packet toward the destination.

## Why It Exists
Switches are great for local delivery, but they only understand MAC addresses and can't move traffic between different IP networks. Routers solve this — they operate at Layer 3 and make decisions based on IP addresses.

## How It Works

**The key decision your device makes every time it sends traffic:**
> "Is the destination on my network or somewhere else?"

- **Same subnet** → talk directly via the switch (ARP to find the MAC, no router involved)
- **Different subnet** → send to the default gateway (the router's local interface)

**What the router actually does:**
1. Receives a frame — checks if the destination MAC is its own → yes, it is
2. Strips the Layer 2 frame (MAC addresses) — looks at the Layer 3 packet (IP addresses)
3. Checks the **routing table** — finds the best path to the destination network
4. **Re-encapsulates** the packet into a brand new frame with new source/destination MACs for the next hop
5. Forwards it out the correct interface

```
[Johnny 10.1.1.10] ──► [Router 10.1.1.1 | 23.x.x.x] ──► [Server 23.x.x.x]

Frame hop 1:    SRC MAC=Johnny    DST MAC=Router
Frame hop 2:    SRC MAC=Router    DST MAC=Server

IP packet:      SRC IP=Johnny     DST IP=Server  ← never changes end-to-end
```

**IP addresses stay the same end-to-end. MAC addresses change at every hop.**

## Key Terms
| Term | Meaning |
|------|---------|
| Default Gateway | The router interface on your local network — your exit door |
| Routing Table | The router's map of known networks and where to send traffic |
| Layer 3 | The network layer — where IP addresses and routing decisions live |
| Packet | Layer 3 unit of data — contains source/destination IP addresses |
| Re-encapsulation | Router strips old Layer 2 frame and builds a new one for the next hop |
| ARP | Address Resolution Protocol — how devices discover MAC addresses from IP addresses |

## Exam Traps
1. **MAC addresses change hop-to-hop. IP addresses don't.** This trips up almost everyone early on.
2. **No default gateway = no internet.** Device can talk locally but has no exit door for remote traffic.
3. **Routers don't connect devices — they connect networks.** The switch handles device-to-device. The router handles network-to-network.

## Commands
```
! View the routing table
show ip route

! Check interface IP addresses
show ip interface brief

! Ping to test connectivity
ping 10.1.1.1
```

## Recall Questions
1. Johnny pings `10.1.1.50` from `10.1.1.10/24`. Does this hit the router? Why?
2. What changes and what stays the same as a packet crosses three routers?
3. What happens if a device has the wrong default gateway configured?
