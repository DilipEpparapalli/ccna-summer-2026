## Simple Explanation
NAT is a mechanism that translates private IP addresses to public IP addresses at the edge of a network. It exists because private addresses are not present in any public routing table and cannot be forwarded across the internet. Without NAT, traffic from internal devices would be dropped the moment it left your network.

## Why It Exists
IPv4 gives us roughly 4.3 billion addresses — not enough for every device on Earth. RFC 1918 defined private address ranges (10.x, 172.16–31.x, 192.168.x) for internal use only. NAT lets many private devices share a small number of public IPs, which is the primary reason IPv4 is still functional decades past its expected expiration.

## How It Works

### The Core Idea
A router sitting at the network edge maintains a **NAT translation table**. When an inside device sends traffic out, the router rewrites the source IP (and sometimes port) before forwarding the packet to the internet. When the response comes back, the router looks up the table and rewrites the destination back to the original private address.

### Three Types of NAT

| Type | Mapping | Use Case |
|------|---------|----------|
| Static NAT | 1 private IP ↔ 1 public IP (permanent) | Internal server that must be reachable from outside |
| Dynamic NAT | Many private IPs → pool of public IPs (on demand) | Multiple devices needing outbound access; pool exhaustion is a real failure mode |
| NAT Overload (PAT) | Many private IPs → 1 public IP (tracked by port) | Almost every real-world deployment |

### NAT Overload — The Port Trick
When multiple devices share one public IP, the router differentiates return traffic using **TCP/UDP port numbers**. Each outbound connection gets a unique source port assigned, and the translation table maps:

```
Inside Local (private IP:port) → Inside Global (public IP:translated port)

192.168.1.10:1234 → 203.0.113.5:5001
192.168.1.20:1234 → 203.0.113.5:5002
192.168.1.30:4567 → 203.0.113.5:5003
```

Return traffic arrives at the public IP, the router checks the destination port, and forwards to the correct inside device. This is why NAT Overload is also called **PAT (Port Address Translation)**.

## Key Terms
| Term | Meaning |
|------|---------|
| Inside Local | Private IP address of an internal device |
| Inside Global | Public IP address the internal device appears as on the internet |
| Outside Global | Public IP of the destination (e.g. a web server) |
| NAT Table | Router's translation record mapping inside local ↔ inside global + ports |
| PAT | Port Address Translation — another name for NAT Overload |
| RFC 1918 | The standard defining private IPv4 address ranges |
| Dynamic NAT Pool | A manually configured set of public IPs available for dynamic assignment |

## Real-World Connection
In an enterprise branch office, every employee device uses a private 10.x or 192.168.x address internally. A single public IP from the ISP is assigned to the edge router. NAT Overload runs on that router and allows all internal devices to reach the internet simultaneously — differentiated entirely by port numbers. This is the default behavior on virtually every SOHO and branch router deployed today.

## Exam Traps
1. **Dynamic NAT ≠ automatic** — the public IP pool must be manually defined. If the pool is exhausted, new connections fail. Don't confuse "dynamic" with "unlimited."
2. **PAT and NAT Overload are the same thing** — Cisco uses both terms. NAT Overload is the IOS config keyword; PAT is the conceptual name.
3. **Private IPs aren't blocked by ACLs** — they're unroutable because no ISP advertises RFC 1918 prefixes in the public routing table. The distinction matters: it's structural absence, not a configured filter.

## Commands
```
! Define inside and outside interfaces
ip nat inside
ip nat outside

! Static NAT — one private to one public
ip nat inside source static 10.0.0.10 203.0.113.10

! Dynamic NAT — define pool and ACL
ip nat pool PUBLIC_POOL 203.0.113.1 203.0.113.5 netmask 255.255.255.248
access-list 1 permit 10.0.0.0 0.0.0.255
ip nat inside source list 1 pool PUBLIC_POOL

! NAT Overload (PAT) — many-to-one using exit interface
access-list 1 permit 10.0.0.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload

! Verify
show ip nat translations
show ip nat statistics
```

## Recall Questions
1. Why can't private IP addresses be routed across the public internet?
2. What happens in a Dynamic NAT deployment when all pool addresses are in use and a new device tries to reach the internet?
3. How does a router using NAT Overload know which internal device should receive a returning packet when all devices share one public IP?
4. What is the difference between "Inside Local" and "Inside Global" in Cisco NAT terminology?
5. Why is NAT Overload also called PAT?
