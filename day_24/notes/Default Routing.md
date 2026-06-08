## Simple Explanation
A default route is the router's fallback — the path it uses when no other route in the table matches the destination. Instead of dropping the packet, the router sends it to a designated next hop (usually the ISP) and lets the rest of the internet figure out the journey from there.

## Why It Exists
A router can only route what it knows. Static routes cover specific remote networks, but the internet has millions of networks — you can't type them all by hand. The default route solves this with one entry: "For anything you don't specifically know about, send it here."

## How It Works

1. A packet arrives at the router destined for some unknown address (e.g., a website on the internet).
2. The router checks its routing table for a match, working from most specific to least specific (rule of specificity / longest prefix match).
3. If no specific route matches, the router reaches the default route — `0.0.0.0/0` — which matches *every* destination.
4. The packet is forwarded to the configured next hop, typically the ISP router.
5. From there, the ISP and the broader internet carry the packet the rest of the way.

### Rule of Specificity in Action

```
Routing Table:
  S  192.168.3.0/24 via 192.168.2.2   ← specific static route
  S* 0.0.0.0/0      via 216.0.5.1     ← default route (least specific)

Traffic to 192.168.3.50  → matches 192.168.3.0/24 → uses static route
Traffic to 8.8.8.8       → no specific match      → uses default route
```

The more specific route always wins. The default route only fires when nothing else matches.

### Topology Example

```
[Cafe LAN]        [WAN]         [ISP]          [Internet]
192.168.1.0/24 ---192.168.2.0/30--- 216.0.5.0/30 ---→ 0.0.0.0/0
     |                  |               |
  Cafe-RT1          Eth0/1          ISP-RT1
  Eth0/0: .1.1    Eth0/2: 216.0.5.2   216.0.5.1
```

Default route on Cafe-RT1:
```
ip route 0.0.0.0 0.0.0.0 216.0.5.1
```
→ "For everything you don't know, send to the ISP router."

## Key Terms

| Term | Meaning |
|------|---------|
| Default route | A catch-all route matching any destination; used when no specific route exists |
| Gateway of last resort | Cisco's name for the next hop used by the default route; shown at top of `show ip route` output |
| `0.0.0.0 0.0.0.0` | Network + mask meaning "any destination" — the least specific possible match |
| Rule of specificity | Longer (more specific) prefix matches always win over shorter ones |
| BGP | The routing protocol that actually carries full internet routing tables between ISPs — not your job at the edge |
| /30 | A tiny 4-address subnet (2 usable) used for point-to-point links between two routers |

## Real-World Connection
Every edge router in every office, café, or home connects to an ISP via a default route. The ISP's router carries a full BGP routing table with every internet prefix — your router doesn't need to. It just needs to know: "I don't know where this goes, but my ISP does." One route replaces millions.

## Exam Traps

1. **Default route vs. no route.** Without a default route, traffic to unknown destinations is silently dropped — the router doesn't guess or forward anyway. No route, no trip.
2. **Specificity always beats default.** A static route to `192.168.3.0/24` will always win over `0.0.0.0/0` for traffic destined to that subnet. The default route is the *last* resort, not a wildcard override.
3. **`show ip route` notation.** The default route appears as `S*` — `S` for static, `*` marking it as the candidate default. The phrase "Gateway of last resort is X" appears at the top of the output when a default route is set. If it says "Gateway of last resort is not set," there is no default route.

## Commands

```
! Configure a default route pointing to ISP next hop
ip route 0.0.0.0 0.0.0.0 <next-hop-ip>

! Example: Cafe-RT1 pointing to ISP
ip route 0.0.0.0 0.0.0.0 216.0.5.1

! Verify — look for S* and "Gateway of last resort" line
show ip route

! Example output snippet:
! Gateway of last resort is 216.0.5.1 to network 0.0.0.0
! S*  0.0.0.0/0 [1/0] via 216.0.5.1
```

## Recall Questions

1. What happens to a packet when the router has no matching route and no default route configured?
2. You see `Gateway of last resort is not set` in `show ip route`. What does that tell you?
3. A router has a default route to the ISP and a static route to `10.10.10.0/24`. Traffic arrives for `10.10.10.5`. Which route is used and why?
4. Why is `0.0.0.0 0.0.0.0` described as "least specific" rather than "no match"?
