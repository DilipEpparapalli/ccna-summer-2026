
### Simple Explanation

Network speed is measured in **bits per second**. File size is measured in **bytes**. Since 1 byte = 8 bits, you divide by 8 to convert link speed into real-world transfer speed. Confuse the two and every calculation will be wrong.

### Why It Exists

Bits are the fundamental unit of digital data — a 0 or a 1. Bytes group 8 bits together into something more human-usable (a character, a color value, etc.). Networks transmit bits; humans and operating systems think in bytes. You need both, and you need to know when each applies.

### How It Works

#### The Units

**Data size (storage) — bytes:**

|Unit|Size|Example|
|---|---|---|
|Byte (B)|8 bits|One character|
|Kilobyte (KB)|~1,000 bytes|Small text file|
|Megabyte (MB)|~1,000,000 bytes|A photo|
|Gigabyte (GB)|~1,000,000,000 bytes|A movie|
|Terabyte (TB)|~1,000,000,000,000 bytes|Large storage drive|

> ⚠️ Technically 1 KB = 1,024 bytes (binary), but for network calculations use 1,000 — it's close enough and is what most speed tests and ISPs use.

**Network speed — bits per second:**

|Unit|Speed|
|---|---|
|Kbps|Kilobits per second|
|Mbps|Megabits per second|
|Gbps|Gigabits per second|

#### The Critical Notation Difference

```
b = bit  (lowercase)  → network speed   → Mbps, Gbps
B = Byte (uppercase)  → file size       → MB, GB
```

A **1 Gbps** link ≠ moving **1 GB/sec**. It moves **1 Gb/sec** = **125 MB/sec**.

#### The Core Formula

```
Transfer time (seconds) = File size (bits) ÷ Link speed (bits per second)

Steps:
1. Convert file size to bits  →  multiply bytes by 8
2. Divide by link speed in bps
3. Result = transfer time in seconds
```

#### Worked Example

**Scenario:** Move a 2 GB file over a 1 Gbps link.

```
Step 1: Convert file to bits
  2 GB = 2 × 10⁹ bytes
  2 × 10⁹ × 8 = 16 × 10⁹ bits = 16 Gbits

Step 2: Divide by link speed
  16 Gbits ÷ 1 Gbps = 16 seconds

Answer: ~16 seconds (ideal conditions)
```

**Scenario 2:** What is the real-world transfer rate of a 1 Gbps link in MB/s?

```
1 Gbps = 1,000 Mbps
1,000 Mbps ÷ 8 = 125 MB/s
```

> ⚠️ Real-world speeds are always lower due to protocol overhead, latency, retransmissions, and other traffic sharing the link.

#### Why Multiply by 8 (Not Divide)?

When converting file size to match link speed units, you're going from **bytes → bits** (making the number bigger). Multiply by 8. If you divided, you'd be going bits → bytes, which would give you the wrong unit for the comparison.

### Key Terms

|Term|Meaning|
|---|---|
|Bit|Smallest unit of data — a 0 or 1|
|Byte|8 bits grouped together|
|Mbps|Megabits per second — network speed|
|MB/s|Megabytes per second — transfer rate|
|Throughput|Actual observed data transfer rate (always ≤ theoretical max)|
|Bandwidth|Maximum theoretical capacity of a link|
|Overhead|Protocol metadata that consumes bandwidth but isn't your data|

### Real-World Connection

An enterprise WAN link is sold as "100 Mbps." A server needs to replicate a 50 GB database nightly.

```
50 GB = 50 × 10⁹ × 8 = 400 Gbits
400 Gbits ÷ 0.1 Gbps = 4,000 seconds ≈ 67 minutes
```

That's just the math. Factor in overhead and competing traffic — you're probably looking at 90+ minutes. This is why network engineers size links and schedule replication windows.

### Exam Traps

1. **The lowercase/uppercase trap** — Cisco questions will write "Mbps" and "MBps" intentionally. Know which is which before reading the question options.
2. **Bandwidth ≠ throughput** — Bandwidth is the theoretical max. Throughput is what you actually get. Overhead, retransmissions, and congestion reduce throughput below the rated bandwidth.
3. **ISPs advertise in bits** — "You have 500 Mbps internet" means ~62.5 MB/s actual download speed. Don't let a question trick you into thinking 500 MB/s.
