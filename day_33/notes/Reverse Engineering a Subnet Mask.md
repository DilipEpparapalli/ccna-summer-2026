## Simple Explanation
Reverse engineering a mask means taking an existing IP address and subnet mask and working
backward to identify the network address, broadcast address, and valid host range. You are
not designing a subnet — you are decoding one that already exists.

## Why It Exists
In the real world you rarely design networks from scratch. You inherit somebody else's
configuration and need to troubleshoot it. Reverse engineering lets you quickly answer:
what network is this host in, is its gateway valid, and are two devices actually on the
same subnet?

## How It Works

**Step 1 — Identify the interesting octet**
Find the octet in the subnet mask that is neither 255 nor 0. That octet is doing the
subnetting work.

**Step 2 — Find the increment**
Convert the interesting octet to binary. The increment is the place value of the first
zero bit.

| Mask octet | Binary    | Increment |
|------------|-----------|-----------|
| 240        | 11110000  | 16        |
| 224        | 11100000  | 32        |
| 192        | 11000000  | 64        |
| 128        | 10000000  | 128       |

**Step 3 — Map the ranges and locate the host**
Count up from 0 in increments until you bracket the host address.

Example: **192.168.5.22 / 255.255.255.240**

- Mask 240 → /28 → increment 16
- Ranges in the last octet: 0–15, **16–31**, 32–47, ...
- .22 falls in 16–31
- Network address: **192.168.5.16**
- Broadcast address: **192.168.5.31**
- Valid hosts: 192.168.5.17 – 192.168.5.30
- .22 is a valid host in the **192.168.5.16/28** subnet

## Key Terms

| Term | Meaning |
|------|---------|
| Increment | The block size determined by the first zero bit in the interesting octet |
| Network address | First address in a subnet block — not assignable to a host |
| Broadcast address | Last address in a subnet block — not assignable to a host |
| Valid host range | All addresses between network and broadcast, inclusive |
| Interesting octet | The octet in the mask that is not 255 or 0 |

## The Gateway Validity Check
A host determines whether to send a packet directly or via the default gateway using a
bitwise AND operation:

```
Source IP   AND Subnet Mask  = Source Network
Dest IP     AND Subnet Mask  = Dest Network

If Source Network == Dest Network → send directly (ARP for destination)
If Source Network != Dest Network → send to default gateway (ARP for gateway MAC)
```

**The broken scenario — why gateway subnet membership matters:**

```
PC:      192.168.5.10  /28  → network 192.168.5.0/28   (range .0–.15)
GW:      192.168.5.33  /28  → network 192.168.5.32/28  (range .32–.47)
Server:  192.168.5.17  /28  → network 192.168.5.16/28  (range .16–.31)
```

All three are on the same physical switch. All three are syntactically valid IP addresses.
None of them can communicate correctly.

When the PC at .10 tries to reach the gateway at .33:
1. AND operation: .33 is in the 192.168.5.32/28 network — different from PC's .0/28
2. PC concludes: "remote host, send to default gateway"
3. PC ARPs for .33 inside its own subnet — no response, because .33 is logically elsewhere
4. Connection fails — the PC cannot reach the gateway it needs to route off-subnet

> A configuration can look valid because nothing immediately errors. The device accepts
> the IP. But the network is still fundamentally broken.

## Real-World Connection
This is the skill you use when a user reports "internet is down." You pull up the IP
configuration, run the reverse engineering steps, and immediately know whether the host,
subnet, and gateway are self-consistent — before you chase any other gremlins.

In an enterprise environment, mismatched subnets on the same VLAN are a common
misconfiguration after manual IP assignments or botched migrations. Reverse engineering
the mask lets you spot the problem in seconds.

## Exam Traps

1. **"Valid IP" does not mean "correct subnet."** A host will accept any syntactically
   valid address. The exam will give you devices with valid-looking IPs that are in
   different /28 or /27 blocks and ask you why they can't communicate.

2. **The gateway must be in the same subnet as the host — not just the same VLAN.**
   If the gateway IP lands in a different subnet block, the host's AND operation sends
   traffic toward the gateway via the gateway — a circular dead end.

3. **Always name the network by its network address, not its position.**
   "The second /28 block" is imprecise. "192.168.5.16/28" is exact. Exam questions and
   troubleshooting scenarios both require the precise network address.

## Commands (for verification in IOS)

```
! View IP address and mask on a router interface
show interfaces GigabitEthernet0/0

! View IP address configured on a host (from router or L3 switch)
show ip interface brief

! Verify ARP table — useful for confirming gateway reachability
show arp
```

## Recall Questions

1. Given 10.0.0.77 / 255.255.255.192 — what is the network address, broadcast, and
   valid host range?
2. A PC at 172.16.4.50/28 has a default gateway of 172.16.4.65. Will this work? Why
   or why not? Trace the AND operation.
3. Two hosts are on the same physical switch: 192.168.1.20/27 and 192.168.1.70/27.
   Can they communicate directly? What are their respective networks?
4. What is the difference between an IP address being "syntactically valid" and being
   "correctly placed in a subnet"?
