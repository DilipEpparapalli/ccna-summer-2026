## Simple Explanation
Subnetting is the practice of dividing a large network into smaller, custom-sized subnetworks. Instead of accepting a one-size-fits-all default network, you use a subnet mask to precisely control how many devices can live in each network segment — fitting the network to the business, not the other way around.

## Why It Exists
Two reasons:

**1. Broadcasts don't scale.**
Every device on a network must interrupt its CPU to process every broadcast frame — ARP requests, DHCP discovery, and other normal network chatter — even if the broadcast has nothing to do with that device. One device doing this is fine. Thousands of devices doing this constantly is a performance problem. Subnetting shrinks the broadcast domain, so fewer devices are exposed to traffic they don't care about.

Important distinction: **routers block broadcasts** — they never cross a router boundary. **Switches flood broadcasts** out every port. The victims are the **end hosts**, not the infrastructure.

**2. IP addresses are a finite resource — don't waste them.**
A point-to-point link between two routers has exactly two endpoints. Assigning a /24 (254 usable addresses) to that link wastes 252 addresses. A /30 gives 4 total addresses, 2 usable — a perfect fit. In public IP space especially, that waste is expensive and inexcusable.

## How It Works
A subnet mask tells devices which part of an IP address identifies the network and which part identifies the host.

```
IP Address:    192.168.1.50
Subnet Mask:   255.255.255.0  (/24)
               └─ network ──┘ └host┘
```

By changing the subnet mask, you change the boundary between network and host — making subnets larger or smaller as needed.

At NetworkChuck Coffee, different parts of the business get different sized subnets:

```
Guest WiFi        → large subnet  (many customers, high turnover)
Employee devices  → medium subnet (stable, known headcount)
Camera network    → small subnet  (fixed number of devices)
Router WAN links  → /30           (exactly 2 endpoints, no waste)
```

## Key Terms
| Term | Meaning |
|------|---------|
| Subnet | A smaller network carved out of a larger address space |
| Subnet mask | The value that defines where the network portion ends and the host portion begins |
| Broadcast domain | The set of devices that receive each other's broadcast traffic |
| /30 | A subnet with 4 total addresses, 2 usable — standard for point-to-point links |
| Classful addressing | The old system of fixed network sizes (Class A/B/C) with no flexibility |
| VLSM | Variable Length Subnet Masking — using different subnet sizes within the same address space |

## Real-World Connection
Your Pi homelab already does this. When you set up VLANs with separate subnets for trusted devices, IoT, and guests — that's subnetting in action. You didn't want your guest devices broadcasting into your trusted network, and you didn't want a flat mess where everything talks to everything. The principle at Castle Rysen is identical: POS systems, guest WiFi, cameras, and back-office devices each need their own segment, and subnetting is what creates those clean boundaries.

## Exam Traps
1. **Broadcasts stop at routers, not switches.** Switches flood broadcasts everywhere. If an exam question asks what device contains broadcast traffic, the answer is a router (or a Layer 3 device), not a switch.
2. **The first and last address in any subnet are unusable.** First = network address, last = broadcast address. A /30 has 4 total addresses but only 2 usable hosts. Don't forget this when counting.
3. **Subnetting isn't just math — it's design.** Exam scenarios often describe a business requirement ("this link has 2 devices") and ask you to pick the right subnet size. Always ask "how many hosts actually need to live here?" before choosing a mask.

## Recall Questions
1. What are the two main reasons subnetting exists?
2. Why does a /24 subnet assigned to a point-to-point link waste addresses — and what should you use instead?
3. A switch receives a broadcast frame. What does it do? What does a router do with the same frame?
4. A /30 subnet has how many total addresses? How many are usable for hosts, and why?
5. At NetworkChuck Coffee, why shouldn't guest WiFi and POS systems share the same subnet?
