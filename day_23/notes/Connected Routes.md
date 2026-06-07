## Simple Explanation
The moment a router interface is assigned an IP address and brought up, the router automatically adds that network to its routing table as a connected route. No manual configuration required. If the interface goes down, the route disappears. The router only knows what it is directly touching.

## Why It Exists
A router needs a starting point — some baseline knowledge of where it sits in the network. Connected routes are that baseline. They are the networks the router can see without being told anything. Everything beyond that has to be learned some other way.

## How It Works

### Interface Up → Route Appears
Assign an IP address to an interface and run `no shutdown`. The router immediately adds two entries to the routing table for that interface:

```
C    192.168.1.0/24 is directly connected, Ethernet0/0
L    192.168.1.1/32 is directly connected, Ethernet0/0
```

| Code | What It Represents |
|------|-------------------|
| `C` | The network address of the subnet on that interface |
| `L` | The router's own specific IP address on that interface (/32 host route) |

### Interface Down → Route Disappears
Shut down the interface and the connected route is immediately removed from the routing table. Bring it back up and the route returns. The routing table reacts to interface state in real time.

```
! Shut the interface
Router(config-if)# shutdown
Router# show ip route
→ 192.168.1.0/24 gone from the table

! Bring it back
Router(config-if)# no shutdown
Router# show ip route
→ 192.168.1.0/24 is back
```

### What the Router Still Does Not Know
Connected routes only cover directly attached networks. A router with two interfaces knows exactly two networks — nothing more.

```
Cafe-RT1 interfaces:
  Eth0/0 → 192.168.1.0/24  (Coffee House LAN)   ← knows this
  Eth0/1 → 192.168.2.0/30  (P2P WAN link)       ← knows this
  
  192.168.3.0/24 (Fallout Shelter LAN)           ← does NOT know this
```

To reach remote networks, one of three things must happen:
- **Static route** — administrator manually adds the path
- **Default route** — catch-all pointing toward a gateway
- **Dynamic routing protocol** — routers exchange route information automatically (OSPF, EIGRP, BGP)

## Key Terms

| Term | Meaning |
|------|---------|
| **Connected Route (`C`)** | Automatically added when an interface with an IP is up; represents the subnet on that interface |
| **Local Route (`L`)** | The router's own /32 host address on that interface; added alongside every connected route |
| **Routing Table** | The router's map of known networks and how to reach them |
| **`show ip route`** | Command to view the routing table and all route entries |
| **`show ip interface brief`** | Command to see interface status and IP assignments at a glance |
| **Administrative Down** | Interface manually shut with `shutdown`; no route exists while in this state |

## Real-World Connection

### Castle Rysen — NetworkChuck Coffee
After configuring Cafe-RT1 and Fallout-RT1 with IP addresses on their active interfaces, each router's routing table showed only the networks it was directly touching:

- **Cafe-RT1** knew `192.168.1.0/24` (Coffee House LAN) and `192.168.2.0/30` (P2P link)
- **Fallout-RT1** knew `192.168.3.0/24` (Fallout Shelter LAN) and `192.168.2.0/30` (P2P link)

Neither router knew about the other's LAN. Pings across the P2P link worked. A PC on the Coffee House LAN still could not reach a server on the Fallout Shelter LAN — static routes needed next.

### Enterprise Reality
Every router in a production network starts here. Before any routing protocol runs, the router builds its connected route foundation from its own interfaces. When a link goes down in an enterprise, the connected route vanishes and traffic stops — which is exactly why network teams monitor interface state as the first step in any outage investigation.

## Exam Traps

1. **`C` and `L` are not the same thing.** `C` is the network. `L` is the router's own host address on that interface. Both appear automatically, but they represent different things. Don't confuse them.

2. **Connected routes disappear when the interface goes down.** The routing table is live, not static. A shut interface means no route — even if the IP address is still configured. Always check interface state alongside the routing table when troubleshooting.

3. **Connected routes do not solve end-to-end reachability.** Two routers pinging each other across a P2P link does not mean their LANs can communicate. Each router needs a route to the *remote* LAN — either static or dynamic. Forgetting the return route on the far router is the most common mistake.

## Commands

```
! Assign IP and bring interface up (creates connected route automatically)
Router(config)# interface ethernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown

! View full routing table (shows C and L entries)
Router# show ip route

! View only connected routes
Router# show ip route connected

! View interface status and IP assignments
Router# show ip interface brief

! Shut an interface (removes connected route)
Router(config-if)# shutdown
```

## Recall Questions

1. What two route codes appear in the routing table when you bring up an interface with an IP address? What does each one represent?
2. You configure `192.168.10.1/24` on Ethernet0/0 and run `no shutdown`. What entry appears under the `L` code in the routing table?
3. A connected route for `10.0.0.0/30` disappears from the routing table. What is the most likely cause?
4. Cafe-RT1 and Fallout-RT1 are configured and can ping each other across the P2P link. Can a PC behind Cafe-RT1 reach a server behind Fallout-RT1? What is missing?
5. Name the three ways a router can learn about a network it is not directly connected to.
