## Simple Explanation
NAT is not just about translating private IPs to public IPs — it covers several network scenarios including overlapping address spaces and policy-based routing. To describe what's happening precisely, Cisco uses four terms based on two questions: **who owns the address?** (inside vs outside) and **is it private or public?** (local vs global).

## Why It Exists
A single word like "source" or "destination" isn't enough when NAT is running, because the same packet carries different addresses at different points in the translation. Cisco's four-term system gives engineers a precise, unambiguous way to label any IP address in a NAT conversation — regardless of how complex the translation gets.

## How It Works

Two questions unlock all four terms:

1. **Who owns it?** → Inside (ours) or Outside (theirs)
2. **Is it private or public?** → Local (private) or Global (public)

Combine the answers and you get the term.

```
YOUR SIDE            |  ROUTER  |  THEIR SIDE
---------------------|----------|-------------------
192.168.1.105        |          |  142.250.80.46
inside local         |          |  outside global

98.12.45.200         |          |  192.168.99.50 (rare)
inside global        |          |  outside local
```

### The Four Terms

**Inside Local**
Our network, private address. The real IP assigned to an internal device before translation.
> Example: A workstation at `192.168.1.105`

**Inside Global**
Our network, public address. The translated public IP that represents our inside devices to the outside world. This is what the internet actually sees.
> Example: The ISP-assigned address `98.12.45.200` on the edge router

**Outside Global**
Their network, public address. The real public IP of a server or device on the internet.
> Example: Google's server at `142.250.80.46`

**Outside Local**
Their device, pretending to be local to our network. A private alias assigned to an outside device so our network can reach it — typically used when two networks have overlapping address spaces.
> Example: During a company merger, their server `10.1.1.50` is given the alias `192.168.99.50` on our side so our devices can route to it without conflict.

### The Normal NAT Flow
```
Inside Local  →  [NAT translation]  →  Inside Global  →  Outside Global
192.168.1.105                          98.12.45.200       142.250.80.46
```
With PAT/overload, many inside local addresses share one inside global address.

## Key Terms

| Term | Who Owns It | Private or Public | When You See It |
|------|-------------|-------------------|-----------------|
| Inside Local | Us | Private | Source address before translation |
| Inside Global | Us | Public | Source address after translation |
| Outside Global | Them | Public | Destination address (normal case) |
| Outside Local | Them | Private (alias) | Destination address when their IP is translated |

## Real-World Connection
In a standard enterprise network, traffic from internal workstations (`inside local`) gets translated to a single public IP (`inside global`) via PAT before hitting the internet. The web servers being reached are `outside global`. Outside local only appears in edge cases — most commonly when two organizations merge with overlapping RFC 1918 address spaces and need NAT to act as a translator between both private ranges while renumbering is sorted out.

## Exam Traps
1. **Inside global is not "the router's IP"** — it's the public address *representing* inside devices. The router holds it, but the term is about representation, not the physical device.
2. **Outside local is the least common but most tested edge case** — know the merger/overlapping address scenario cold. It's the one that trips people up because it feels backwards.
3. **Local ≠ your network, Global ≠ their network** — local/global describe private vs public, not ownership. Ownership is inside/outside. Mixing these up under exam pressure is a classic mistake.

## Commands (applicable in next lesson — NAT configuration)
```
! These terms appear in show commands during NAT config:
show ip nat translations
! Output columns: inside global, inside local, outside local, outside global
```

## Recall Questions
1. What two questions do you ask to identify any Cisco NAT address type?
2. Your laptop is `10.0.0.5` and reaches Google at `8.8.8.8` through a router with public IP `203.0.113.1`. Label all three addresses using Cisco NAT terms.
3. When would an outside local address appear? Describe the scenario in plain English.
4. What is the difference between inside global and outside global?
5. In a PAT setup, which term describes the single public IP shared by all internal devices?
