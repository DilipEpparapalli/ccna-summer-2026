# TCP/IP and OSI Models

## Simple Explanation
Both are layered models that describe how network communication works. TCP/IP is what devices actually use. OSI is the vocabulary engineers use to troubleshoot. Know both — they complement each other.

## Why It Exists
Early networks were proprietary — vendor A's devices couldn't talk to vendor B's. Standardized models gave everyone a shared rulebook. Now a Raspberry Pi can talk to a Cisco router to an iPhone without anyone thinking twice about it.

## The Models Side by Side

```
OSI (7 layers)          TCP/IP (5 layers)      What's happening
─────────────────────────────────────────────────────────────────
7. Application    ──┐
6. Presentation   ──┼──► Application           Browser, HTTPS, DNS
5. Session        ──┘
4. Transport      ──────► Transport             TCP / UDP, Ports
3. Network        ──────► Network               IP addresses, Routing
2. Data Link      ──────► Data Link             MAC addresses, Frames
1. Physical       ──────► Physical              Cables, signals, NICs
```

## How Encapsulation Works (Data Going Down the Stack)

```
Application data
    + Transport header (TCP/UDP + port)  → SEGMENT
    + Network header (IP addresses)      → PACKET/DATAGRAMS  
    + Data Link header+trailer (MACs)    → FRAME
    + Physical (bits on the wire)        → BITS
```

Each layer wraps the previous layer's data — like envelopes inside envelopes. At the destination, each layer unwraps its own envelope and passes the rest up.

## TCP vs UDP

| | TCP | UDP |
|--|-----|-----|
| Connection | Yes (3-way handshake) | No |
| Reliable delivery | Yes (ACKs + retransmit) | No |
| Speed | Slower | Faster |
| Use cases | Web, email, banking, SSH | Video streaming, gaming, VoIP, DNS |

**3-way handshake (TCP):**
```
Client ──► SYN       ──► Server
Client ◄── SYN-ACK   ◄── Server
Client ──► ACK       ──► Server
          [data flows]
```

## Common Port Numbers
| Port | Protocol | Use |
|------|----------|-----|
| 80 | HTTP | Unencrypted web |
| 443 | HTTPS | Encrypted web |
| 22 | SSH | Secure remote CLI |
| 53 | DNS | Domain name resolution |
| 21 | FTP | File transfer |
| 3389 | RDP | Remote desktop |

## What Each Layer Device Cares About

```
[End Device] → reads all layers (it's the destination)
[Router]     → reads up to Layer 3 (IP) — strips and rebuilds Layer 2
[Switch]     → reads only Layer 2 (MAC) — never touches IP
```

YouTube uses **both TCP and UDP** simultaneously — TCP for page elements (reliability matters), UDP for the video stream (speed matters more than perfection).
## Exam Traps
1. **OSI has 7 layers, TCP/IP has 4 or 5** depending on the version — know both and don't mix them up on the exam
2. **HTTPS runs over TCP, not UDP** — it's secure AND reliable. Never confuse encryption with transport protocol.
3. **UDP is not "worse" than TCP** — it's the right tool for real-time applications. Wrong answer: "TCP is always better."
4. **Switches don't read Layer 3.** If a switch sees an IP packet, it only cares about the Layer 2 frame wrapping it.

## Troubleshooting with the OSI Model
```
"Layer 1 issue" → check cable, NIC, physical connection
"Layer 2 issue" → check MAC table, VLAN config, switch port
"Layer 3 issue" → check IP address, subnet mask, default gateway
"Layer 4 issue" → check TCP/UDP, firewall port rules
"Layer 7 issue" → check the application, DNS, server config
```

## Recall Questions
1. A packet travels from your Pi to a server across 3 routers. What changes at each router? What never changes?
2. Why does YouTube use TCP for the webpage but UDP for the video?
3. You can ping a server but can't load its website. What layer(s) would you investigate first?
