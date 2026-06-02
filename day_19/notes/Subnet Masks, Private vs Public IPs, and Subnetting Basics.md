## Subnet Masks — The Network Definer

An IP address alone is incomplete. Without a subnet mask, a device has no way to answer the most fundamental routing question: is this destination on my network, or do I need a router?

The subnet mask draws the line. It tells a device which bits of the IP address identify the network (the neighborhood) and which bits identify the host (the specific house). Every IP address must be paired with a subnet mask — the operating system won't accept one without the other.

### How a Device Uses the Mask to Make Decisions

When a device wants to send traffic somewhere, it doesn't guess — it runs a binary AND operation between its subnet mask and both IP addresses (its own and the destination's) to extract just the network portions. Then it compares them:

- **Network portions match** → destination is local → deliver directly using MAC addressing
- **Network portions differ** → destination is remote → send to the default gateway

Example: Device is `172.20.5.10/16` (mask `255.255.0.0`). Destination is `172.20.99.200`.

- Device network: `172.20.x.x`
- Destination network: `172.20.x.x`
- Match → local delivery, no router needed

Change the destination to `172.21.5.10` and now the network portion differs (`172.21` vs `172.20`) — off to the default gateway it goes.

The mask is not optional decoration. It is the mechanism that drives every local vs. remote forwarding decision a device makes.

### Reading the Mask

Anywhere you see `255` in a subnet mask, those bits belong to the network. Anywhere you see `0`, those bits belong to the host.

| Subnet Mask   | Network Bits       | Host Bits         | CIDR |
| ------------- | ------------------ | ----------------- | ---- |
| 255.0.0.0     | First octet        | Last three octets | /8   |
| 255.255.0.0   | First two octets   | Last two octets   | /16  |
| 255.255.255.0 | First three octets | Last octet        | /24  |

---

## IP Address Classes (Historical Context)

Classful networking is largely obsolete in modern design, but you still need to recognize it because it explains where default subnet masks came from and why private IP ranges look the way they do.

| Class | First Octet Range | Default Mask        | Usable Hosts  | Intended For         |
| ----- | ----------------- | ------------------- | ------------- | -------------------- |
| A     | 1–126             | 255.0.0.0 (/8)      | ~16.7 million | Large enterprises    |
| B     | 128–191           | 255.255.0.0 (/16)   | ~65,000       | Medium organizations |
| C     | 192–223           | 255.255.255.0 (/24) | 254           | Small networks       |

The problem with classful networking: the network sizes are rigid and don't match reality. A company needing 500 hosts had to take a Class B (65,000 addresses) and waste the rest. This is why CIDR and subnetting exist.

---

## Why Default Network Sizes Are a Problem

A Class A network like `10.0.0.0/8` gives over 16 million host addresses in a single network. Two problems with using it as-is:

**1. Massive address waste.** No real organization puts 16 million devices on one flat network. The unused address space is gone — you can't loan it to anyone else while it's allocated to you.

**2. Broadcast collapse.** Every device on a network segment receives every broadcast. ARP requests, DHCP discovery, and other broadcast traffic hits all 16 million hosts simultaneously. The network drowns in its own noise long before the address space fills up. Routers stop broadcasts at network boundaries — the more you subnet, the smaller each broadcast domain becomes, and the healthier the network.

Subnetting solves both: it carves large address blocks into smaller, right-sized networks that match actual deployment needs and keep broadcast traffic contained.

---

## Private vs Public IP Addresses

### The Three Private Ranges (memorize on sight)

| Range                         | Class Origin | Addresses     |
| ----------------------------- | ------------ | ------------- |
| 10.0.0.0 – 10.255.255.255     | A            | ~16.7 million |
| 172.16.0.0 – 172.31.255.255   | B            | ~1 million    |
| 192.168.0.0 – 192.168.255.255 | C            | ~65,000       |

These ranges come from RFC 1918. The internet does not route them. They exist for use inside private networks only.

> The `172.16–172.31` range is the one people forget. Memorize it intentionally.

### Why Private Addresses Don't Break the Internet

Every home network in the world can use `192.168.1.10` simultaneously — millions of devices sharing the same private address — and none of them conflict. This works because private addresses never appear on the public internet. Each network is isolated behind its own router.

The mechanism that makes this work is **NAT (Network Address Translation)**. When a private device sends traffic to the internet, the router intercepts it and rewrites the source IP address from the private address to the ISP-assigned public IP. It tracks each session using port numbers (16-bit, giving 65,535 possible values) so it knows how to reverse-translate replies back to the right internal device. The mapping lives in the NAT table on the router.

Result: many internal devices share one public IP address, each distinguished by a unique port number. The internet only ever sees the public IP.

### When to Use Which Private Range

| Range               | Good For                                                      |
| ------------------- | ------------------------------------------------------------- |
| `192.168.x.x`       | Home networks, small offices — fits easily in /24 subnets     |
| `172.16–172.31.x.x` | Medium organizations needing more room than Class C           |
| `10.x.x.x`          | Large enterprises, multi-site organizations with many subnets |

---

## Subnetting Basics — Carving Networks to Fit

CIDR (Classless Inter-Domain Routing) replaced classful networking by allowing the subnet mask to be set independently of the address class. Instead of accepting a /8 because an address starts with 10, you choose the mask that fits your actual needs.

### The Castle Rysen Addressing Pattern

A clean real-world approach: take the `192.168.0.0` range and allocate four /24 subnets per location — three active segments (internal, voice, guest) plus one spare for growth.

```
Shop 1:  192.168.0.0/24   (internal)
         192.168.1.0/24   (voice)
         192.168.2.0/24   (guest)
         192.168.3.0/24   (spare)

Shop 2:  192.168.4.0/24
         192.168.5.0/24
         192.168.6.0/24
         192.168.7.0/24
```

Grouping in predictable blocks serves two purposes: it leaves room for growth without renumbering, and it supports **route summarization** later — representing multiple subnets with a single routing advertisement, reducing routing overhead.

### Why the Spare Network Matters

Networks grow. A camera system gets added. Guest WiFi gets split from contractor WiFi. A new service requires isolation. Reserving spare address space at design time prevents painful renumbering later. A good addressing plan solves today's problem and anticipates tomorrow's.

---

## Key Terms

| Term                | Meaning                                                                                    |
| ------------------- | ------------------------------------------------------------------------------------------ |
| Subnet mask         | Defines which bits of an IP address are network vs host                                    |
| CIDR                | Classless Inter-Domain Routing — allows flexible mask lengths independent of address class |
| Broadcast domain    | The set of devices that receive a broadcast; bounded by routers                            |
| RFC 1918            | The standard defining the three private IPv4 address ranges                                |
| NAT                 | Network Address Translation — maps private IPs to a public IP using port numbers           |
| NAT table           | The router's record of active private↔public address mappings                              |
| Route summarization | Representing multiple subnets with one routing advertisement                               |

---

## Exam Traps

**Private ranges don't route on the internet — ever.** If you see a private IP as a source or destination in an internet-facing context, the problem is NAT misconfiguration or a routing error. Don't try to trace private IPs across the internet.

**The 172.16–172.31 range is the one people miss.** Not just `172.16.x.x` — the entire range through `172.31.255.255` is private. `172.32.x.x` is public.

**Subnet mask drives local vs remote decisions, not the IP address alone.** Two devices can have IPs that look close together and still be on different networks if the mask says so. Always check the mask before assuming devices can communicate directly.

**Subnetting solves two problems, not one.** Address waste is the obvious one. Broadcast domain size is equally important — and on the exam, both are valid reasons to subnet.

**NAT uses port numbers to track sessions, not just IP addresses.** One public IP can represent thousands of private devices simultaneously because each session gets a unique source port in the NAT table.

---

## Recall Questions

1. A device has IP `10.5.20.1/8`. Another device has IP `10.200.50.1`. Does traffic go locally or to the default gateway? What is the device actually comparing?
2. What are the three private IPv4 ranges? Which one do people most often forget?
3. Why is a 16-million-host flat network a problem even if you only have 500 devices on it?
4. What does NAT use to distinguish between multiple private devices sharing one public IP address?
5. Why does a network design leave a spare /24 subnet per location even when it's not immediately needed?
