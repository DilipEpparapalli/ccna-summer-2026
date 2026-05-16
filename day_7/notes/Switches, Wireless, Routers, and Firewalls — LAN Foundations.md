
## Simple Explanation
Four devices build the LAN and connect it to the world: a switch connects wired devices locally, an AP extends that LAN over WiFi, a router moves  traffic between networks (LAN to WAN), and a firewall decides what traffic is allowed or blocked. One box can do all four jobs — but separation is how you scale.

## Why It Exists
Each device solves a specific problem. Combining them works at small scale.  As traffic, users, and security requirements grow, dedicated devices handle each job better, fail independently, and are easier to troubleshoot.

## How It Works

### The Switch
- Connects all wired LAN devices through Ethernet ports (interfaces)
- Lives in a network closet — not under desks
- Ethernet cable limit: 100 meters before signal degrades
- Beyond 100m: add another switch — it fully regenerates the signal
  (a repeater just amplifies noise; a switch starts fresh)
- Standard business sizes: 24-port or 48-port managed switches

### The Wireless Access Point (AP)
- Plugs into the switch — extends the LAN wirelessly, does NOT create a new network
- Omnidirectional antennas: broad coverage bubble around the AP
- Directional antennas: focused beam — can bridge two buildings without running cable ("thumb over the hose" effect)

### The Router
- Moves traffic between networks — one foot in the LAN, one in the WAN
- Core question it asks: **"where does this traffic go?"**
- Fewer ports than a switch — usually just LAN side and WAN side
- Think: border crossing at the edge of your network

### The Firewall
- Inspects traffic and enforces security rules
- Core question it asks: **"should this traffic be allowed at all?"**
- Can block or permit based on source, destination, port, protocol
- Often combined with a router in one device — but born with a 
  different purpose

### All-in-One vs Purpose-Built
| Scenario | All-in-One | Purpose-Built |
|----------|-----------|---------------|
| Home / small office | Fine | Overkill |
| Growing business | Bottleneck | Scales cleanly |
| Device failure | Everything dies | One function affected |
| Security control | Limited | Per-device tuning |

## Key Terms
| Term | Meaning |
|------|---------|
| Switch | Connects wired LAN devices; regenerates signal |
| AP | Wireless Access Point — extends LAN over WiFi |
| Router | Moves traffic between networks (LAN ↔ WAN) |
| Firewall | Permits or blocks traffic based on security rules |
| LAN | Local Area Network — devices inside the building |
| WAN | Wide Area Network — the internet or link to another site |
| Interface | A port on a network device |
| Single point of failure | One device whose death kills everything it serves |
| 100m rule | Max copper Ethernet run before signal degrades |
| MDF | Main Distribution Frame — where core network gear lives |

## Real-World Connection
In an enterprise branch, a dedicated firewall sits at the WAN edge inspecting all inbound and outbound traffic. Behind it, a router handles path decisions. A 48-port managed switch connects servers, APs, and workstations. APs mount in the ceiling, cabled back to the switch. If the firewall is upgraded, routing continues unaffected. If an AP fails, only that coverage zone drops. No single failure takes down the whole site.

## Exam Traps
1. **Router vs firewall is about purpose, not hardware** — one box can  do both, but a router routes and a firewall filters. Know which question each one is answering.
2. **APs do not create a new network** — they extend the existing LAN wirelessly. A WiFi client's traffic still flows through the switch.
3. **A switch regenerates signal; a repeater amplifies it** — amplifying also amplifies noise. Switches are the modern answer to the 100m limit.
4. **All-in-one devices are not wrong** — they're right-sized for small deployments. The exam tests whether you know *when* to separate functions, not whether combo boxes are evil.
5. **Routers have fewer ports than switches** — in general. But modern devices combine roles, so don't identify a device by port count alone. Ask what function it's performing.

## Recall Questions
1. A wireless client sends a file to a wired server on the same LAN. Name every device the traffic touches in order.
2. Your cable run is 140 meters. What do you do, and why not use a repeater?
3. What is the one-sentence difference between what a router does and what a firewall does?
4. The office all-in-one box dies. List every function that goes down simultaneously. Why is that worse than a dedicated switch failing?
5. When does it make sense to move from an all-in-one device to purpose-built gear?