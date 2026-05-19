
## 1. Why Models Exist

Two types of models in networking:

- **Communication models** — shared vocabulary for engineers (OSI, TCP/IP)
- **Design models** — blueprints for building networks before touching hardware

Without a common framework, two engineers facing a broken network have no shared language.
Models fix that. "Layer 3 issue" instantly means IP problem to any engineer on the planet.

---

## 2. OSI vs TCP/IP — The Big Picture

|         | OSI Model                                                | TCP/IP Model                                       |
| ------- | -------------------------------------------------------- | -------------------------------------------------- |
| Layers  | 7                                                        | 5                                                  |
| Purpose | Communication framework — how we **talk** about networks | Actual protocol suite — what **runs** the internet |
| Status  | Theoretical / troubleshooting language                   | Live implementation                                |

**The key insight:**
> OSI describes. TCP/IP runs. Engineers speak OSI, networks run TCP/IP.

TCP/IP won the protocol war — not because it was better, but because it shipped first.
The internet built itself on top of it. OSI became the language we use to describe it.

---

## 3. The OSI Model — 7 Layers

### Layer Map (Bottom to Top)

| Layer | Name | PDU | Job |
|-------|------|-----|-----|
| 1 | Physical | Bits | Transmits raw bits — copper, fiber, radio |
| 2 | Data Link | Frame | Hop-to-hop delivery using MAC addresses |
| 3 | Network | Packet | End-to-end delivery using IP addresses |
| 4 | Transport | Segment | TCP/UDP + port numbers — app-to-app delivery |
| 5 | Session | Data | Opens, manages, closes sessions per application |
| 6 | Presentation | Data | Formatting, encryption, compression |
| 7 | Application | Message | User-facing apps — HTTP, DNS, SMTP, etc. |

### The Two Zones

**Top 3 (Layers 5–7)** — Application team's live.
TCP/IP collapses all three into a single "Application" layer.

**Bottom 4 (Layers 1–4)** — Where network engineers live.
This is where switching, routing, and transport happen.

---

## 4. TCP/IP Model — 5 Layers

| TCP/IP Layer   | Maps to OSI    | What lives here             |
| -------------- | -------------- | --------------------------- |
| Application    | Layers 5, 6, 7 | HTTP, HTTPS, DNS, SMTP, SSH |
| Transport      | Layer 4        | TCP, UDP, port numbers      |
| Network        | Layer 3        | IP addressing, routing      |
| Data Link      | Layers 2       | MAC addresses               |
| Physical Layer | Layer 1        | Physical media              |

---

## 5. Encapsulation & Decapsulation

### What It Is

As data travels **down** the stack, each layer wraps it with a header.
As data travels **up** the stack on the receiving end, each layer strips its header.

- Going down = **Encapsulation**
- Going up = **Decapsulation**

### The Journey — One Click to the Wire

```
User clicks a link (e.g., HTTPS request)

Layer 7 - Application   → Message     — Browser generates HTTP request
Layer 6 - Presentation  → Data        — TLS encryption applied
Layer 5 - Session       → Data        — Session created, tracked per app
Layer 4 - Transport     → Segment     — TCP chosen; source + destination port added
Layer 3 - Network       → Packet      — Source + destination IP address added
Layer 2 - Data Link     → Frame       — Source + destination MAC address added
Layer 1 - Physical      → Bits        — Converted to electrical/light/radio signals
```

### MAC vs IP — The Critical Distinction

```
[PC] ──────> [Router A] ──────> [Router B] ──────> [Server]

IP:  192.168.1.10 ─────────────────────────────> 93.184.216.34
     (stays constant end to end — final destination)

MAC: AA:BB → CC:DD    CC:DD → EE:FF    EE:FF → GG:HH
     (changes at every hop — next stop only)
```

> If the IP address changed at every hop, the packet would never know where it was ultimately going. IP is the constant. MAC is the variable.

---

## 6. Port Numbers

- Live at **Layer 4 (Transport)** — not Layer 3
- 65,536 total ports available
- **Well-known ports (0–1023):**

| Port | Protocol |
|------|----------|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 53 | DNS |
| 25 | SMTP |

**How multiple apps work simultaneously:**
One NIC receives a frame → decapsulate → check **destination port number** → OS routes segment to the correct application (Chrome vs Spotify vs email).

**Socket** = IP address + port number combined , uniquely identifies a connection.

---

## 7. TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Reliability | Guaranteed — receiver acknowledges every segment | No guarantee — fire and forget |
| Speed | Slower (overhead of acknowledgements) | Faster |
| Use case | Web browsing, file transfer, email | Streaming, VoIP, gaming, DNS |

> Streaming chooses UDP because a dropped packet from 5 seconds ago is useless. You don't want the network to pause and go back for it.

---

## 8. Real-World Connections

| Scenario | Layer | Why |
|---|---|---|
| Cable unplugged | Layer 1 | Physical medium |
| Wrong MAC / switch issue | Layer 2 | Data Link |
| IP misconfiguration | Layer 3 | Network |
| Firewall blocking port 443 | Layer 4 | Transport |
| HTTPS not loading | Layer 6/7 | Presentation/Application |

When an engineer says "Layer 2 problem" — they mean MAC, switching, frames.
When they say "Layer 3 problem" — they mean IP, routing, packets.
You will almost never hear someone say "Internet layer problem" — OSI language dominates.

---

## 9. Exam Traps

1. **Port numbers = Layer 4, not Layer 3.**
   Packets carry IP addresses only. Ports are in the segment.

2. **MAC changes at every hop. IP does not.**
   Know exactly why — this is a classic exam question.

3. **TCP/IP original model** had 4 layers with Physical + Data Link merged into "Link."
   The updated model splits them. Know both versions.

4. **OSI layers 5–7 are tested individually** even though TCP/IP collapses them.
   Don't confuse the two models' layer counts.

5. **Cat5e supports 1Gbps** — not 100Mbps. That's Cat5 (no "e").
   Cat6 adds 10Gbps at distances up to ~55m.

---

## 10. Recall Questions

1. What are the 7 OSI layers in order, bottom to top?
2. What is the PDU name at each layer?
3. Why does MAC change at every hop but IP stays constant?
4. Two apps running simultaneously — one packet arrives. How does the OS route it correctly?
5. When would you choose UDP over TCP? Give a real example.
6. An engineer says "it's a Layer 2 loop." Where do you look and why?
7. What layer does a firewall blocking port 443 operate at?
8. TCP/IP won over OSI — what does each model actually do today?
