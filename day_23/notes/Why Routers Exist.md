## Simple Explanation
A switch is great at moving traffic *within* a network, but it has no idea how to reach anything *outside* it. A router is the device that connects separate networks together — and makes the decision about where a packet goes when it can't stay local.

## Why It Exists
Every device on a LAN can talk to each other through a switch. But the moment traffic needs to leave — to the internet, to another branch, to a data center — the switch hits a wall. Routers exist to cross that boundary. They also contain broadcast domains, keeping local noise from flooding the entire internet.

## How It Works

### The Core Decision Every End Device Makes First
Before any packet leaves a device, the sender checks: *is this destination on my network or not?*

- **Same network** → ARP for the destination's MAC address, send the frame directly
- **Different network** → ARP for the **default gateway's** MAC address, send the frame to the router

The router receives the frame, strips the Layer 2 header, looks at the Layer 3 destination IP, consults its routing table, and forwards the packet out the correct interface toward the destination.

```
PC ──────────────► Router ──────────────► Remote Server
      Frame to             Packet routed
      gateway MAC          by IP address
```

### The Routing Table
The router's routing table is its map of the world — a list of networks and how to reach them.

| Route Type | What It Means | Example |
|------------|--------------|---------|
| **Connected** | Network directly attached to a router interface | `192.168.1.0/24 via Gi0/0` |
| **Static** | Manually configured path to a remote network | `10.0.0.0/8 via 203.0.113.1` |
| **Default** | Catch-all — used when no specific route matches | `0.0.0.0/0 via ISP gateway` |
| **Dynamic** | Learned automatically via routing protocols (OSPF, EIGRP, BGP) | exchanged between routers |

### Rule of Specificity
When multiple routes match a destination, **the most specific route wins**.

```
Routing table has:
  10.0.0.0/8        → ISP
  10.1.1.0/24       → Branch router

Traffic to 10.1.1.50 → uses 10.1.1.0/24  ✓  (longer prefix = more specific)
Traffic to 10.2.0.1  → uses 10.0.0.0/8   ✓  (no better match exists)
```

### Broadcasts Stop at the Router
Switches forward broadcasts (`FF:FF:FF:FF:FF:FF`) to every port in the VLAN. Routers do not forward broadcasts between interfaces. This is what keeps a single ARP request in one office from flooding every device on the internet.

## Key Terms

| Term | Meaning |
|------|---------|
| **Default Gateway** | The router interface address that end devices send traffic to when the destination is off-subnet |
| **Routing Table** | The router's map — a list of known networks and how to reach them |
| **Connected Route** | A route the router knows because its own interface is on that network |
| **Static Route** | A manually configured route to a remote network |
| **Default Route** | `0.0.0.0/0` — the catch-all used when no specific route matches |
| **Dynamic Routing** | Routers automatically exchange route information using protocols like OSPF, EIGRP, or BGP |
| **Rule of Specificity** | When multiple routes match, the one with the longest prefix (most specific) wins |
| **Broadcast Domain** | The set of devices that receive a broadcast frame — routers create the boundary |

## Real-World Connection

### Castle Rysen — NetworkChuck Coffee
The `Cafe01-RTR01` router sits between the coffee shop LAN and both the internet and the fallout shelter WAN link. Without it:
- Cafe devices can talk to each other (switch handles that)
- Nothing can reach the internet
- Nothing can reach the fallout shelter server network

The router's two interfaces each represent a separate network. Configuring a static route in one direction (cafe → shelter) is not enough — the return path (shelter → cafe) must also be configured, or replies never make it back.

### Enterprise Scale
A company with 50 branch offices cannot maintain static routes manually — one missing entry breaks connectivity for an entire site. Dynamic routing protocols like OSPF let routers discover and share route information automatically, adapting when links go up or down without administrator intervention.

## Exam Traps

1. **Static routes are one-directional.** Configuring a route from A to B does not create a return route from B to A. Always verify both directions. A successful ping does not guarantee a return route exists — check the remote router's table too.

2. **Most specific route wins, not the first match.** A default route (`0.0.0.0/0`) never overrides a more specific route to the same destination. The router always picks the longest prefix match.

3. **Routers stop broadcasts — switches do not.** A broadcast stays inside a VLAN/broadcast domain. The router is the boundary. Conflating flooding (unknown unicast) with broadcasting (FF:FF:FF:FF:FF:FF) is a common slip — these are triggered by different conditions.

## Commands

```
! View the routing table
Router# show ip route

! Configure a static route
Router(config)# ip route 10.0.0.0 255.255.255.0 192.168.1.1
!                          ^network  ^mask          ^next-hop

! Configure a default route
Router(config)# ip route 0.0.0.0 0.0.0.0 203.0.113.1

! Verify an interface is up and has an IP
Router# show ip interface brief
```

## Recall Questions

1. A PC wants to send traffic to a server on a different subnet. What destination MAC address does it put in the frame, and why?
2. You configure a static route from the cafe router to the shelter network. Pings from cafe to shelter work, but not in reverse. What is missing and where do you fix it?
3. A routing table has entries for `172.16.0.0/16` and `172.16.5.0/24`. Traffic arrives for `172.16.5.100`. Which route is used and why?
4. Why don't routers forward broadcast traffic between interfaces?
5. What is the difference between a connected route and a static route?
