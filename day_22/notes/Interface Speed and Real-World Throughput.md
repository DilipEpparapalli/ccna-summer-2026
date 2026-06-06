## Simple Explanation

Network speed is measured in **bits per second**, but file sizes are measured in **bytes**. Since 1 byte = 8 bits, a "1 Gbps" link can never actually deliver 1 GB of useful data per second — headers, protocols, and TCP behavior all eat into that number before your data even arrives.

## Why It Exists

When you design or troubleshoot a network, you need to answer real questions: *How long will this backup take? Will this uplink handle peak traffic? Why is my 1 Gbps link only showing 600 Mbps of throughput?* Without understanding the gap between theoretical speed and real-world throughput, you'll buy the wrong gear, set wrong expectations, and blame the wrong things.

## How It Works

### Bits vs. Bytes — Get This Right First

| Unit | What It Measures | Abbreviation |
|------|-----------------|--------------|
| Bit | Single binary value (0 or 1) | b (lowercase) |
| Byte | 8 bits — one "character" of data | B (uppercase) |
| Megabit | 1,000,000 bits | Mb |
| Megabyte | 1,000,000 bytes (= 8,000,000 bits) | MB |
| Gigabit | 1,000,000,000 bits | Gb |
| Gigabyte | 1,000,000,000 bytes (= 8,000,000,000 bits) | GB |

> **The trap:** A "10 Gbps" link and a "10 GBps" link are not the same thing. The second is 8× faster. On the job, always check the case of the B.

**Conversion rule:** To convert file size (bytes) into network language (bits), multiply by 8.

---

### Transfer Time Math

**Formula:**

```
Transfer Time = (File Size in bits) ÷ (Link Speed in bits per second)
```

**Example — 100 GB file over a 1 Gbps link:**

```
100 GB  →  100,000,000,000 bytes
        ×  8
        =  800,000,000,000 bits  (800 Gb)

Link speed: 1 Gbps = 1,000,000,000 bits/sec

Transfer time = 800,000,000,000 ÷ 1,000,000,000
              = 800 seconds
              ≈ 13 minutes (theoretical)
              ≈ 16 minutes (real-world, with ~20% overhead)
```

**Always pad your estimate by ~20%** for real-world conditions.

---

### Why Real Throughput Is Always Less Than the Spec

A "1 Gbps" link is a physical capacity — like a 4-lane highway. But several things eat into that capacity before your data arrives:

#### 1. Protocol Overhead (Headers)

Data doesn't travel raw. Every packet is wrapped in headers at multiple layers. Think of it like mailing a letter — the envelope, stamp, and address all take up space and cost money, but none of it is your message.

```
[ Ethernet Header | IP Header | TCP Header | YOUR DATA | Ethernet Trailer ]
      14 bytes        20 bytes    20 bytes    ~1,460 bytes     4 bytes
```

A standard 1,500-byte Ethernet frame carries only ~1,460 bytes of actual payload. That's roughly **3% overhead per packet** — just from headers, before anything else.

#### 2. TCP Slow Start

TCP doesn't open a connection and immediately blast at full speed. It starts slow — sending a small number of packets, waiting for acknowledgment, then doubling its rate each round — until it either hits the network's limit or a packet gets dropped.

```
Round 1:  send 1 packet   → ACK → OK
Round 2:  send 2 packets  → ACK → OK
Round 3:  send 4 packets  → ACK → OK
Round 4:  send 8 packets  → ACK → OK
...continues doubling until limit or drop
```

**Why it exists:** TCP has no idea how congested the network is when the connection opens. Slow start protects routers from being overwhelmed by every connection blasting simultaneously.

**The consequence:** Short transfers (small files, web requests) may finish *before TCP ever reaches full speed*. This is why your first file in a session often feels slower than subsequent ones.

**When a packet drops:** TCP backs off — cuts its rate significantly — and begins ramping again from a lower point. On a congested network this creates a repeating cycle of ramp → drop → ramp → drop, which is why video streams sometimes buffer in waves.

#### 3. Upstream Bottlenecks

Your local link speed is only one segment of the path. If the sender has a 100 Mbps upload connection and you have a 1 Gbps download, your effective speed is capped at 100 Mbps. The slowest link in the chain sets the ceiling — every time.

```
Sender (100 Mbps upload) ──→ Internet ──→ You (1 Gbps download)
                  ↑
         This is your actual limit
```

#### 4. Network Congestion

Shared infrastructure (internet circuits, uplinks, ISP backbones) carries traffic from many sources simultaneously. High utilization = more queuing at routers = higher latency and reduced effective throughput, even if no packets are dropped.

---

### Quick Reference: Transfer Time by Link Speed

| File Size | 100 Mbps | 1 Gbps | 10 Gbps |
|-----------|----------|--------|---------|
| 1 GB | ~80 sec | ~8 sec | <1 sec |
| 10 GB | ~13 min | ~80 sec | ~8 sec |
| 100 GB | ~2.2 hrs | ~13 min | ~80 sec |
| 1 TB | ~22 hrs | ~2.2 hrs | ~13 min |

*Theoretical values. Add ~20% for real-world overhead.*

---

## Key Terms

| Term | Meaning |
|------|---------|
| Bit | Smallest unit of data — a single 1 or 0 |
| Byte | 8 bits; the unit used for file sizes |
| Throughput | Actual data delivered per second (always less than link speed) |
| Bandwidth | Theoretical maximum capacity of a link |
| Overhead | Non-payload data (headers, trailers) that consumes link capacity |
| Payload | The actual user data carried inside a packet |
| TCP Slow Start | TCP's ramp-up mechanism for new connections |
| Bottleneck | The slowest link in a path — sets the ceiling for the whole transfer |
| MTU | Maximum Transmission Unit — largest frame size a link carries (typically 1,500 bytes on Ethernet) |

---

## Real-World Connection

**Castle Rysen scenario:** The coffee roastery needs to replicate high-res product photography and video content between the Central Office and District Shops over a WAN link. Before sizing that link, you'd calculate: *How large are those files? How often do they transfer? What's the transfer window?* A 500 MB video file over a 10 Mbps WAN link takes ~400 seconds — almost 7 minutes per file. If they're pushing 20 files a night, that's over 2 hours just for content sync, before any other traffic runs.

**Home lab tie-in:** When you pull a large Docker image or push a backup to Nextcloud over WireGuard, the slowest segment — WireGuard encryption overhead, your ISP's upload cap, or the Pi's CPU handling encryption — is what you actually feel. The "1 Gbps" on your LAN switch means nothing if the bottleneck is elsewhere.

---

## Exam Traps

1. **Bits vs. bytes unit confusion** — The exam will give you a file size in GB and a link speed in Mbps and expect you to convert correctly. If you forget to multiply by 8, your answer is 8× off. Always convert to the same unit before dividing.

2. **"Faster link = proportionally faster transfer" — not always true** — If the bottleneck is upstream (sender's ISP, a slow WAN segment, CPU processing), upgrading your local link changes nothing. The exam loves scenarios where you identify *where* the bottleneck actually lives.

3. **Theoretical vs. real throughput** — A question might describe a 1 Gbps link showing 750 Mbps of throughput and ask if this indicates a problem. It doesn't — overhead, TCP behavior, and traffic patterns routinely keep real throughput well below the physical maximum.

---

## Commands

*No IOS commands apply to this concept — it is a mathematical and protocol-behavior topic. Relevant commands appear in Week 6 interface configuration (speed, duplex) and Week 8 (bandwidth statements for routing protocols).*

---

## Recall Questions

1. A colleague says "I upgraded our link from 100 Mbps to 1 Gbps but transfers only got 6x faster, not 10x." Name two reasons this is normal and expected.
2. You need to transfer a 50 GB database backup over a 500 Mbps link. What is the theoretical transfer time? What would you estimate in practice?
3. Why does TCP slow start exist, and what happens to TCP's transmission rate when a packet is dropped mid-transfer?
4. Your download speed is 1 Gbps but a file from a friend takes twice as long as expected. What is the first thing you check, and why?
5. A packet on an Ethernet network has a 1,500-byte MTU. How many bytes of that are actual payload, and what consumes the rest?
