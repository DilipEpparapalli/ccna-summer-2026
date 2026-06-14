## Simple Explanation
NAT Overload lets many inside private devices share one (or a few) public IP addresses simultaneously. The router keeps each session unique not by assigning different public IPs, but by assigning different source port numbers. This is the NAT method running on virtually every home router and small business edge device in the world.

## Why It Exists
Dynamic NAT solves the manual-mapping problem but still burns one public IP per active inside device. Public IPv4 addresses are scarce and expensive. NAT Overload stretches a single public IP to serve hundreds or thousands of inside devices at once — using port numbers as the differentiator instead of IP addresses.

At Castle Rysen, the cafe has laptops, POS systems, tablets, phones, and IoT gear all needing internet. Buying a public IP for each device would be absurd. One public IP with PAT handles all of it.

## How It Works

```
Inside Devices              Router (cafe01-RT01)              Internet
192.168.1.50  ──────►  216.0.5.2 : port 1024  ──────►  1.1.1.1
192.168.1.51  ──────►  216.0.5.2 : port 1025  ──────►  1.1.1.1
192.168.1.52  ──────►  216.0.5.2 : port 1026  ──────►  1.1.1.1
              all appear as the SAME public IP, different ports
```

The router maintains a NAT table that tracks:
- Inside local IP + port
- Inside global IP + port (the translated one)
- Outside destination

When a reply comes back to `216.0.5.2:1025`, the router knows exactly which inside device it belongs to.

**Configuration (interface method — most common):**

```bash
# Step 1 — interfaces already marked inside/outside from before
# Step 2 — ACL to identify inside addresses (same as dynamic NAT)
access-list 1 permit 192.168.1.0 0.0.0.255
access-list 2 permit 192.168.3.0 0.0.0.255

# Step 3 — bind to interface with overload keyword
ip nat inside source list 1 interface GigabitEthernet0/0 overload
ip nat inside source list 2 interface GigabitEthernet0/0 overload
```

The `overload` keyword is the entire difference from dynamic NAT. Without it, you get one-to-one. With it, you get many-to-one via port multiplexing.

**Pool method (when you have multiple public IPs):**
```bash
ip nat pool CAFE_PUBLIC 216.0.5.50 216.0.5.60 netmask 255.255.255.0
ip nat inside source list 1 pool CAFE_PUBLIC overload
```
Router fills port space on the first pool IP, then spills to the next. Useful when one IP's ~65,000 ports aren't enough.

## Key Terms

| Term | Meaning |
|------|---------|
| PAT | Port Address Translation — the formal name for NAT Overload |
| NAT Overload | Cisco's name for PAT — many-to-one using port numbers |
| Inside global | The public IP all inside devices share (e.g. 216.0.5.2) |
| Source port | The port number the router assigns to track each unique session |
| `overload` keyword | The single word that converts dynamic NAT into PAT |
| Interface method | Using the router's outside interface IP as the public address instead of a static pool |

## Real-World Connection
Your home router is doing this right now. Every device on your home network — phone, laptop, Pi, smart TV — shares one public IP from your ISP. The router tracks every session by port number. When your ISP changes your public IP (because most residential ISPs use dynamic addressing), the interface method means your NAT config keeps working automatically — no pool to update, no manual adjustment.

At Castle Rysen: `show ip nat translations` shows multiple inside devices (`192.168.1.50`, etc.) all translating to `216.0.5.2` with different port numbers. That's PAT in action.

## Changing NAT in Production — The Right Sequence
NetworkChuck flagged this as real-world critical. Removing a NAT rule while active translations exist will fail — the router protects in-use translations.

**Correct sequence:**
```bash
# 1. Clear the translation table
clear ip nat translation *

# 2. Verify it's actually clear
show ip nat translations

# 3. Remove the old NAT rule
no ip nat inside source list 1 pool CAFE_PUBLIC

# 4. If traffic keeps rebuilding translations instantly → bring interface down
interface GigabitEthernet0/0
 shutdown

# 5. Make config changes, then bring interface back up
 no shutdown
```
In production: plan a maintenance window and communicate the outage first. Something is always talking — cameras, card readers, cloud apps, monitoring agents — even at 3am.

## Exam Traps

1. **`overload` is the only difference.** `ip nat inside source list 1 pool MYPOOL` is dynamic NAT. `ip nat inside source list 1 pool MYPOOL overload` is PAT. One word changes everything. The exam will test whether you know which command produces which behavior.

2. **Interface method ≠ always one IP.** The interface method uses whatever IP is currently on that interface. If the ISP changes the address, NAT adapts automatically. This is why it's preferred over hard-coded pools for environments with dynamic public IPs.

3. **Each ACL needs its own overload statement.** If you have two inside networks and only configure overload for ACL 1, traffic from ACL 2's network gets no translation. Easy to miss in a multi-segment lab.

4. **`clear ip nat translation *` before removing rules.** Trying to `no` a NAT rule while sessions are active fails silently or throws an error. Clear first, then remove.

5. **PAT is not the same as Dynamic NAT.** Both use ACLs. Both can reference pools. The distinguishing factor is `overload` and port-based session tracking. Don't let the similar config syntax blur the distinction.

## Commands Reference

```bash
# Configure PAT using outside interface IP
ip nat inside source list 1 interface GigabitEthernet0/0 overload

# Configure PAT using a pool (for higher-volume environments)
ip nat inside source list 1 pool CAFE_PUBLIC overload

# Verify translations (note: same public IP, different ports)
show ip nat translations

# Check NAT statistics (hit/miss counts, pool utilization)
show ip nat statistics

# Clear translation table (required before removing NAT rules)
clear ip nat translation *

# Remove a PAT rule
no ip nat inside source list 1 interface GigabitEthernet0/0 overload
```

## NAT Comparison Summary

| Type | Mapping | Uses Ports? | Config keyword |
|------|---------|-------------|----------------|
| Static NAT | 1-to-1, permanent | No | `ip nat inside source static` |
| Dynamic NAT | many-to-many (pool) | No | `ip nat inside source list X pool Y` |
| NAT Overload / PAT | many-to-one (or few) | Yes | `... overload` |

## Recall Questions

1. What single keyword converts a Dynamic NAT rule into PAT? Where does it go in the command?
2. You have 3 inside networks but only configure overload for 2 of them. What happens to traffic from the third?
3. Why is the interface method preferred over a hard-coded pool for small businesses with ISP-assigned public IPs?
4. You try to remove a NAT rule and the router won't let you. What's happening and what do you do first?
5. Your NAT table shows `216.0.5.2:13`, `216.0.5.2:14`, `216.0.5.2:15` — all different inside locals. What type of NAT is this and how do you know?
