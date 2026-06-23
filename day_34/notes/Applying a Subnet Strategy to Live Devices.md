## Simple Explanation
A subnet plan on paper does nothing until it's pushed into real device configuration.
Applying a subnet strategy means translating your address allocations into router
interface IPs, DHCP pools, and updated NAT rules — and doing it in a way that's
logical, predictable, and easy to maintain as the network grows.

## Why It Exists
Subnetting produces a design. Operationalizing that design is a separate skill. You
need to know how to assign addresses systematically, update dependent services like NAT
when the subnet changes, and automate address distribution via DHCP — because manually
assigning every endpoint doesn't scale and creates errors.

## How It Works

### Step 1 — Translate Slash Notation to Dotted Decimal

IOS commands require dotted decimal subnet masks, not slash notation. You need to move
between them fluently.

```
/26 → 255.255.255.192   (increment 64, host bits in last octet)
/27 → 255.255.255.224   (increment 32)
/30 → 255.255.255.252   (increment 4, for point-to-point links)
```

When you look at a subnet like `10.0.18.0/26`, you should immediately see:
- Mask: 255.255.255.192
- Range: 10.0.18.0 – 10.0.18.63
- Usable hosts: 10.0.18.1 – 10.0.18.62
- Broadcast: 10.0.18.63

### Step 2 — Assign Addresses with a Predictable Pattern

Random address assignment makes future troubleshooting painful. Use a consistent
reservation pattern across every site:

```
.1 – .10    Infrastructure (router, switch SVIs, WAP, printers, servers)
.11 – .50   DHCP pool (clients — laptops, tablets, IoT devices, POS terminals)
.51+        Reserved for future static assignments or specialty devices
```

Typical infrastructure assignment order:

| Address      | Device              |
|--------------|---------------------|
| x.x.x.1     | Router LAN interface (default gateway) |
| x.x.x.2     | Switch 1 management SVI |
| x.x.x.3     | Switch 2 management SVI |
| x.x.x.4     | Wireless access point |
| x.x.x.5     | Printer              |
| x.x.x.6     | Server               |

This pattern means any engineer walking into that site can predict where to find
infrastructure devices without looking anything up.

### Step 3 — Configure the Router LAN Interface

```
interface GigabitEthernet0/0
 ip address 10.0.18.1 255.255.255.192
 no shutdown
```

The subnet mask here is the dotted decimal translation of /26. The router's LAN
interface address becomes the default gateway for all hosts in the subnet.

### Step 4 — Update NAT When the Subnet Changes

NAT uses an ACL to identify which inside addresses should be translated. When the
subnet changes, the ACL must be updated to match the new range — otherwise NAT
silently drops traffic from addresses that don't match.

The ACL uses a **wildcard mask**, which is the bitwise inverse of the subnet mask:

```
Subnet mask:   255.255.255.192  →  /26
Wildcard mask: 0.0.0.63

Formula: 255.255.255.255 − 255.255.255.192 = 0.0.0.63
```

The wildcard mask works like a "don't care" filter: `0` bits must match, `63` bits
(the last 6) are free to be anything — which is exactly the host portion of a /26.

Updated NAT ACL:

```
ip access-list standard INSIDE_HOSTS
 permit 10.0.18.0 0.0.0.63
```

**Critical:** If the old NAT rule is still active when you apply the new subnet
config, clear active translations first:

```
clear ip nat translation *
```

Then remove the old ACL before adding the new one. Leaving stale NAT config
alongside a new subnet is a common source of mysterious connectivity failures.

### Step 5 — Configure DHCP

DHCP lets the router hand out IP addresses automatically to client devices. Without
it, every endpoint needs a manual static assignment — unmanageable at scale.

```
! Exclude infrastructure addresses from the pool
ip dhcp excluded-address 10.0.18.1 10.0.18.10
ip dhcp excluded-address 10.0.18.51 10.0.18.62

! Define the pool for client devices
ip dhcp pool DISTRICT-SHOP
 network 10.0.18.0 255.255.255.192
 default-router 10.0.18.1
 dns-server 8.8.8.8
```

When a client device boots and requests an address, the router assigns the next
available IP from the pool, records the binding (IP ↔ MAC address), and tells the
client where its default gateway and DNS server are. Verify bindings with:

```
show ip dhcp binding
```

### Full Picture — What Changes Together

When you re-subnet a site, these four things must all move in sync:

```
┌─────────────────────────────────────────────────────────────┐
│  Subnet changes from OLD /27  →  NEW /26                    │
├─────────────────────┬───────────────────────────────────────┤
│  Router interface   │  New IP + new mask                    │
│  NAT ACL            │  New wildcard mask to match new range │
│  DHCP excluded      │  Updated to match new infrastructure  │
│  DHCP pool          │  New network + mask                   │
└─────────────────────┴───────────────────────────────────────┘
```

Updating only the router interface and forgetting NAT or DHCP is the most common
mistake. The router comes up fine, clients get addresses — but nothing reaches the
internet because NAT is still matching the old subnet.

## Key Terms

| Term | Meaning |
|------|---------|
| Dotted decimal | Subnet mask written as four octets (e.g., 255.255.255.192) |
| Wildcard mask | Inverse of subnet mask; used in ACLs and routing protocols |
| DHCP pool | Range of addresses the router hands out automatically |
| DHCP excluded-address | Addresses the router will never assign dynamically |
| DHCP binding | The recorded IP ↔ MAC mapping for an active lease |
| Infrastructure range | Low-end addresses reserved for routers, switches, and servers |
| NAT ACL | Access list identifying which inside addresses are eligible for translation |

## Real-World Connection

In enterprise environments, this pattern — infrastructure in the low end of the
subnet, DHCP pool in the middle, reserved space at the top — is standard operating
procedure. A network engineer onboarding to any well-run company will find the same
structure at every branch site. The consistency isn't aesthetic; it's operational.
When something breaks at 2am, you want to know exactly where to look without reading
documentation first.

The same principle applies to DHCP exclusions. If you forget to exclude the router's
own IP from the DHCP pool, the router may hand out its own address to a client,
creating an IP conflict that takes down the entire subnet's gateway. That exclusion
is not optional.

## Exam Traps

1. **Wildcard mask vs. subnet mask** — ACLs use wildcard masks (inverted), not
   subnet masks. A /26 subnet uses mask 255.255.255.192 on the interface but wildcard
   0.0.0.63 in the ACL. Getting these swapped is a classic exam gotcha.

2. **Forgetting to clear NAT translations before changing the ACL** — Active
   translations hold references to the old ACL. Removing the ACL while translations
   are live causes unpredictable behavior. Always `clear ip nat translation *` first.

3. **DHCP pool includes the excluded range** — The `ip dhcp excluded-address` command
   does not shrink the pool definition; it just marks those addresses as off-limits
   for dynamic assignment. The `network` command in the pool still covers the full
   subnet. The exclusion is enforced separately.

4. **Default gateway must be the router's LAN interface IP** — The DHCP
   `default-router` value must exactly match the IP configured on the router's
   interface that faces the subnet. A mismatch silently breaks routing for all DHCP
   clients.

## Commands

```
! Interface config
interface GigabitEthernet0/0
 ip address 10.0.18.1 255.255.255.192
 no shutdown

! DHCP exclusions and pool
ip dhcp excluded-address 10.0.18.1 10.0.18.10
ip dhcp pool SITE-NAME
 network 10.0.18.0 255.255.255.192
 default-router 10.0.18.1
 dns-server 8.8.8.8

! NAT ACL (wildcard mask = inverse of subnet mask)
ip access-list standard INSIDE_HOSTS
 permit 10.0.18.0 0.0.0.63

! Verification
show ip dhcp binding
show ip dhcp pool
show ip nat translations
show running-config | section dhcp
```

## Recall Questions

1. A site is re-subnetted from /27 to /26. List every configuration element on the
   router that must be updated, and explain why each one breaks if you miss it.
2. What is the wildcard mask for a /25 subnet? Show the math.
3. A host gets a DHCP address but can't reach the internet. NAT is configured. What
   is the first thing you check, and why?
4. Why should infrastructure addresses always be excluded from the DHCP pool — and
   what failure mode occurs if they aren't?
5. You need to reserve .1–.10 for infrastructure and .51–.62 for future use in a /26.
   Write the two `ip dhcp excluded-address` commands.
