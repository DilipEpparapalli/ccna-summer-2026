
## Simple Explanation
A LAN is a network contained within a single location a building, office, or campus. 
A WAN connects that local network to the outside world, typically the internet. 
Inside any network, communication follows the client/server model: one device requests, 
another provides.

## Why It Exists
Without a structured network, every device would need a direct connection to every other 
device — unscalable chaos. LAN/WAN gives us a clean boundary between "inside" and 
"outside." The client/server model gives us a pattern for how devices actually talk.

## How It Works

### LAN
- A switch connects all local wired devices (servers, APs, workstations)
- A wireless access point extends the LAN over WiFi  but plugs back into the switch
- Traffic that stays inside the LAN never touches the router

### WAN
- The router is the gateway between the LAN and the WAN
- One interface faces inward (the LAN), one faces outward (the ISP/internet)
- All internet-bound traffic exits through the router

### Client/Server Model
- **Client**: any device that initiates a request (laptop, phone, IP camera)
- **Server**: any device that responds with data or a service (file server, DNS, media)
- A device can be both simultaneously  a DNS resolver is a server to LAN clients 
  and a client to upstream DNS servers

## Key Terms
| Term | Meaning |
|------|---------|
| LAN | Local Area Network — devices within a single location |
| WAN | Wide Area Network — connects across large distances, typically the internet |
| Switch | Central device that connects all local wired endpoints |
| AP | Wireless Access Point — extends the LAN over WiFi |
| Router | Connects the LAN to the WAN; the boundary device |
| Client | Device that makes a request |
| Server | Device that responds with data or a service |
| Endpoint | Any device at the edge of the network (clients and servers alike) |
| ISP | Internet Service Provider — delivers the WAN connection |
| Staging | Pre-deployment setup in a safe environment to catch failures before going live |

## Real-World Connection
In an enterprise branch office, a 48-port switch connects workstations, APs, printers, 
and servers. The APs serve wireless clients; those clients still reach the file server 
through the switch — entirely within the LAN. The router connects the branch to HQ or 
the internet via MPLS or a broadband WAN link. A DNS resolver in the branch is a server 
to local clients and a client to the corporate DNS hierarchy upstream.

## Exam Traps
1. **Router is not required for LAN traffic** — devices on the same LAN communicate 
   through the switch only. The router is only involved when traffic leaves the LAN.
2. **Wireless doesn't mean no wires** — the AP is wireless to clients but wired back 
   to the switch. WiFi is the last hop, not the whole network.
3. **Client and server are roles, not device types** — a server can be a client 
   simultaneously (e.g. a proxy, a DNS forwarder, a monitoring agent sending data upstream).

## Recall Questions
1. A user's laptop streams a video from an on-premises media server. Does that traffic 
   touch the router? Why or why not?
2. A security camera sends footage to a local NVR (network video recorder). Which is 
   the client and which is the server?
3. What is the purpose of staging network equipment before an on-site deployment?
4. An AP is plugged into a switch. A phone connects to that AP. What path does traffic 
   take from the phone to a file server also plugged into the switch?