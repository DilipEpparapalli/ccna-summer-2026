## Simple Explanation
Dynamic NAT translates a group of private inside IP addresses to a pool of public outside IP addresses — automatically, without manual per-device configuration. When an inside device sends traffic out, the router picks an available public IP from the pool and creates a translation entry on the fly. When the session ends, that public IP goes back into the pool for someone else.

## Why It Exists
Static NAT requires a hand-built, permanent mapping for every inside device. At small scale that's manageable. At cafe scale — dozens of devices coming and going via DHCP — it becomes impossible to maintain. Dynamic NAT automates the assignment so the router handles translation without human intervention per device.

## How It Works

```
Inside Network          Router (cafe01-RT01)         Internet
192.168.1.0/24    →    NAT Table (dynamic)    →    216.0.5.x
192.168.3.0/24    ↗    pool: .50 – .60        →    1.1.1.1
```

**Step-by-step:**

1. **Mark interfaces** — tell the router which side is inside and which is outside
   ```
   interface GigabitEthernet0/1
    ip nat inside
   interface GigabitEthernet0/0
    ip nat outside
   ```

2. **Create an ACL** — identify which inside addresses are eligible for translation (this is a *selector*, not a firewall rule)
   ```
   access-list 1 permit 192.168.1.0 0.0.0.255
   access-list 2 permit 192.168.3.0 0.0.0.255
   ```

3. **Define a pool** — the range of public IPs the router can assign
   ```
   ip nat pool CAFE_PUBLIC 216.0.5.50 216.0.5.60 netmask 255.255.255.0
   ```

4. **Bind ACL to pool** — wire the selector to the translation rule
   ```
   ip nat inside source list 1 pool CAFE_PUBLIC
   ip nat inside source list 2 pool CAFE_PUBLIC
   ```

5. **Traffic flows** — router assigns a pool IP dynamically, builds a NAT table entry, and forwards the packet

## Key Terms

| Term | Meaning |
|------|---------|
| Inside local | Private IP of the source device (e.g. 192.168.1.50) |
| Inside global | Public IP the router assigns from the pool (e.g. 216.0.5.50) |
| Outside local | Destination IP as seen from inside (usually same as outside global) |
| Outside global | Actual public destination IP (e.g. 1.1.1.1) |
| NAT pool | Named range of public IPs the router draws from |
| ACL (as NAT selector) | Access list used to identify eligible inside addresses — not for filtering, just matching |
| Wildcard mask | Inverse of subnet mask — 0 means "must match", 255 means "don't care" |

## Real-World Connection
In a cafe or office, inside devices get addresses from DHCP and churn constantly. Dynamic NAT handles this without any per-device config — the router just needs to know the eligible inside subnets (via ACL) and the available public IPs (via pool). In the Castle Rysen lab, both the cafe network (`192.168.1.0/24`) and the fallout shelter network (`192.168.3.0/24`) share the same `CAFE_PUBLIC` pool, demonstrating that multiple inside segments can NAT through a single outside interface.

## The Hard Limit
Dynamic NAT pool size = maximum simultaneous translated sessions.

Pool `216.0.5.50 – 216.0.5.60` = **11 public IPs** = max 11 concurrent inside devices.

Device #12 tries to reach the internet → no pool address available → translation fails → no internet. The router doesn't queue or retry. It silently drops the translation attempt. This is the fundamental limitation that makes Dynamic NAT a stepping stone toward PAT/NAT Overload.

## Exam Traps

1. **ACL = selector, not firewall.** An access list used in a NAT rule is just identifying addresses to translate. A `permit` statement doesn't mean traffic is allowed through — it means "match this for NAT." Don't confuse it with security ACLs applied to interfaces.

2. **Interface must be marked inside/outside.** You can build a perfect NAT pool and ACL, but if you forget `ip nat inside` or `ip nat outside` on the interfaces, nothing translates. The router won't warn you — it just silently does nothing.

3. **`interface` keyword ≠ overload.** `ip nat inside source list 1 interface Gi0/0` still does one-to-one dynamic NAT using the interface IP as a pool-of-one. Adding `overload` is what enables PAT — many-to-one with port multiplexing. Easy to confuse on the exam.

4. **Pool exhaustion is silent.** When the pool runs out, inside devices get no error message — their traffic just fails. `show ip nat translations` will show the pool is fully allocated.

## Commands Reference

```bash
# Mark interfaces
ip nat inside                          # on inside-facing interface
ip nat outside                         # on outside-facing interface

# Define eligible inside addresses (ACL as selector)
access-list 1 permit 192.168.1.0 0.0.0.255

# Create public IP pool
ip nat pool CAFE_PUBLIC 216.0.5.50 216.0.5.60 netmask 255.255.255.0

# Bind ACL to pool (the translation rule)
ip nat inside source list 1 pool CAFE_PUBLIC

# Verify
show ip nat translations               # live NAT table
show ip nat statistics                 # pool utilization, hit/miss counts
show ip access-lists                   # confirm ACL entries
```

## Recall Questions

1. Why does Dynamic NAT use an access list, and what does a `permit` entry in that list actually mean in a NAT context?
2. You have a pool of 10 public IPs and 15 inside devices all try to reach the internet simultaneously. What happens to the last 5?
3. What's the difference between `ip nat inside source list 1 pool MYPOOL` and `ip nat inside source list 1 interface Gi0/0`? What keyword makes the second one PAT?
4. You configure Dynamic NAT but translations aren't building. What are the two most likely things you forgot?
5. Can two different ACLs map to the same NAT pool? What does that enable?
