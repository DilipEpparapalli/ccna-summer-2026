
## Simple Explanation
A switch learns which devices live on which ports by reading the source MAC
address of every incoming frame. When a device needs to reach another device
on the same network but doesn't know its MAC address, it uses ARP to find it.
Once ARP completes, the switch knows where both devices live and forwards
traffic directly.

## Why It Exists
Devices communicate using MAC addresses at Layer 2. IP addresses alone aren't
enough to build a frame — you need the destination MAC too. ARP is the bridge
between the IP world you configure and the MAC world the switch actually uses.

## How It Works

### The First Ping (cold start)
```
PC1 wants to ping PC2 (192.168.1.51)
        │
        ▼
Does PC1 know PC2's MAC address?
        │
        ├── YES → build frame, send ping
        │
        └── NO  → send ARP broadcast first
                        │
                        ▼
                ARP Request (broadcast, dest MAC = FF:FF:FF:FF:FF:FF)
                "Who has 192.168.1.51? Tell me your MAC."
                        │
                        ▼
                Switch receives ARP request on Fa0/1
                → Learns PC1's MAC on Fa0/1 (source MAC on ingress)
                → Floods out all ports except Fa0/1
                        │
                        ▼
                PC2 receives it → "That's me" → sends ARP Reply (unicast)
                        │
                        ▼
                Switch receives reply → Learns PC2's MAC on Fa0/2
                → Forwards reply to Fa0/1 (PC1's port, now known)
                        │
                        ▼
                PC1 now has PC2's MAC → builds ICMP frame → sends ping
```

### Frame Structure (what's inside every frame)
| Field | Value | Purpose |
|-------|-------|---------|
| Source IP | 192.168.1.50 | Who sent it |
| Destination IP | 192.168.1.51 | Final destination |
| Source MAC | PC1's MAC | This hop's sender |
| Destination MAC | PC2's MAC | This hop's receiver |

IP addresses = logical, end-to-end, never change across the journey.
MAC addresses = physical, hop-by-hop, change at every router.

## Reaching the Internet (off-LAN traffic)

When the destination is outside the local subnet (e.g. google.com):
- PC1 knows Google's IP (via DNS)
- Google is NOT on the LAN → PC1 does NOT ARP for Google's MAC
- PC1 ARPs for the **default gateway (router)** MAC instead
- Frame is built with:
  - Destination IP = Google's IP
  - Destination MAC = Router's MAC

The router receives it, strips the Layer 2 frame, builds a new one for the
next hop, and forwards it. This repeats across every router on the path.
IP stays fixed on Google. MAC changes every single hop.

```
PC1 ──[MAC: Router]──► Router ──[MAC: Next Hop]──► ... ──► Google
     IP dst: Google           IP dst: Google
```

## Key Terms

| Term | Meaning |
|------|---------|
| ARP | Address Resolution Protocol — resolves IP → MAC on local network |
| ARP Request | Broadcast asking "Who has this IP?" |
| ARP Reply | Unicast response with the MAC address |
| ARP Cache | Local table on each device storing IP-to-MAC mappings |
| Broadcast | Frame sent to all devices (dest MAC = FF:FF:FF:FF:FF:FF) |
| Unicast | Frame sent to one specific device |
| Default Gateway | Router interface on the local network — exit point to the internet |
| CAM Table | Switch's map of MAC addresses to ports |

## Why Flooded Frames Don't Go Back Out the Source Port

When a switch floods a broadcast, it sends it out every port **except** the
one it arrived on. Reason: the source device is on that port — sending the
frame back is useless. It would never find the destination there, wastes
bandwidth, and forces the source device to process its own frame.

## Exam Traps

1. **ARP happens before the ping** — The first ping always takes longer
   because ARP must complete first. The ICMP packet can't be built without
   the destination MAC.

2. **Switch learns on ingress, from the SOURCE MAC** — The switch doesn't
   learn destination MACs. It learns source MACs the moment a frame arrives.
   It learns PC1's MAC from the ARP *request*, before any reply happens.

3. **Off-subnet traffic → ARP for the gateway, not the destination** —
   PC1 never ARPs for Google's MAC. It ARPs for the router. Destination IP
   stays Google's; destination MAC is the router's. MAC is always the
   *next hop*, not the final destination.

4. **IP stays the same end-to-end; MAC changes every hop** — This is the
   most important IP vs MAC distinction on the exam.

5. **ARP cache vs CAM table** — ARP cache lives on the end device (IP→MAC).
   CAM table lives on the switch (MAC→port). Two different tables, two
   different devices, two different purposes.

## Commands

```ios
show mac address-table          ! See what the switch has learned
clear mac address-table dynamic ! Wipe dynamic entries, watch switch relearn
```

```dos
arp -a                          ! (on PC) View local ARP cache
ping 192.168.1.51               ! Triggers ARP if MAC not yet cached
```

## Recall Questions

1. PC1 pings PC2 for the first time. Walk through every step from the moment
   PC1 realizes it doesn't know PC2's MAC to the moment the ping succeeds.
2. When exactly does the switch learn PC1's MAC address during the ARP process?
3. PC1 wants to reach a server at 8.8.8.8. What MAC address goes in the
   destination MAC field of the frame, and why?
4. What is the difference between the ARP cache and the CAM table?
5. Why doesn't the switch flood a frame back out the port it came in on?
