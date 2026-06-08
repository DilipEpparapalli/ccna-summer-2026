## Simple Explanation
A router only knows about networks directly connected to its own interfaces. Static routing is how you manually tell a router about remote networks — networks sitting behind another router — by giving it an explicit path: "to reach that network, send traffic through this next-hop IP."

## Why It Exists
Routers don't guess. If a packet arrives destined for a network that isn't in the routing table, the router drops it. Static routing solves the problem of remote network reachability when you don't want (or need) a dynamic routing protocol to exchange that information automatically.

## How It Works

1. Each router automatically builds **connected routes** for its own directly attached interfaces — no configuration needed.
2. Remote networks (behind another router) are invisible until you add them.
3. You add a **static route** on each router pointing toward the remote network, using the next-hop IP of the adjacent router as the exit point.
4. **Both directions must be configured.** The forward path gets traffic there; the return path gets the reply back. Miss one side and communication silently fails.

### Three-Network Example

```
[Cafe LAN]          [WAN Link]           [Fallout LAN]
192.168.1.0/24 --- 192.168.2.0/24 --- 192.168.3.0/24
     |                   |                    |
  Cafe-RT1 (.1)    (.1)--(.2)          Fallout-RT1 (.2)
```

**On Cafe-RT1** (needs to know about Fallout LAN):
```
ip route 192.168.3.0 255.255.255.0 192.168.2.2
```
→ "To reach 192.168.3.0/24, send to Fallout-RT1's WAN IP."

**On Fallout-RT1** (needs to know about Cafe LAN):
```
ip route 192.168.1.0 255.255.255.0 192.168.2.1
```
→ "To reach 192.168.1.0/24, send to Cafe-RT1's WAN IP."

Each router already knows 192.168.2.0/24 (the WAN) as a connected route — no static route needed for that.

## Key Terms

| Term | Meaning |
|------|---------|
| Connected route | A route automatically added when an interface comes up; the router owns that network |
| Static route | A manually configured route pointing to a remote network via a next-hop IP |
| Next hop | The IP address of the adjacent router that knows the next step toward the destination |
| Remote network | Any network not directly attached to the local router |
| Return route | The static route on the *far* router pointing back to the originating LAN |

## Real-World Connection
In any enterprise with multiple sites, static routes cover small, stable topologies — a branch office with one upstream router, for example. The branch router gets a default static route pointing to HQ; HQ gets a specific static route pointing back to the branch subnet. Same two-sided logic, bigger scale.

## Exam Traps

1. **One-way pings succeed, end-to-end pings fail.** A ping *from the router itself* only tests the forward path. A ping from a host behind the router also requires the return route on the far side. Always verify both directions.
2. **Next-hop IP vs. exit interface.** You *can* specify an outgoing interface instead of a next-hop IP, but the next-hop IP method is standard practice and what you'll see on the exam. Know both exist; use next-hop IP.
3. **Connected routes are automatic, static routes are not.** The exam will present scenarios where a router "should" know a network — if it's not directly connected and no route was added, it doesn't know it. No sympathy from IOS.

## Commands

```
! Add a static route (standard form — use next-hop IP)
ip route <destination-network> <subnet-mask> <next-hop-ip>

! Example: Cafe-RT1 learning about Fallout LAN
ip route 192.168.3.0 255.255.255.0 192.168.2.2

! Example: Fallout-RT1 learning about Cafe LAN
ip route 192.168.1.0 255.255.255.0 192.168.2.1

! Verify routing table
show ip route

! Static routes appear as 'S' in the routing table
! Connected routes appear as 'C'
```

## Recall Questions

1. A router has three interfaces configured and up. How many connected routes will appear in its routing table, and why?
2. You add a static route on Router A toward Router B's LAN. Pings from Router A succeed, but pings from hosts behind Router A fail. What is the most likely cause?
3. Write the static route command for a router that needs to reach `10.0.30.0/24` via next-hop `10.0.12.2`.
4. What letter does a static route show as in `show ip route` output?
