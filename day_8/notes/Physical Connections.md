
### Simple Explanation

Physical connections are the actual roads your data travels on. Before any configuration happens, devices must be connected — physically and correctly. The medium you choose (copper, fiber, wireless) determines speed, distance, and reliability.

### Why It Exists

Different scenarios demand different connections. A short office run is fine on copper. A cross-building backbone needs fiber. Wireless solves mobility but introduces tradeoffs. Choosing wrong means a network that "kind of works" until it doesn't — usually at the worst moment.

### How It Works

#### Copper (Twisted Pair)

- Uses electrical signals to carry data as 0s and 1s
- Categorized by capability — Cat5e, Cat6, Cat6a, etc.
- Higher category = higher speed + less interference
- Max distance: ~100 meters per segment before signal degrades
- Terminated with **RJ45** connectors
- Susceptible to **EMI** (electromagnetic interference)

**Common categories:**

|Category|Max Speed|Max Distance|Notes|
|---|---|---|---|
|Cat 3|10 Mbps|100m|Legacy — don't use|
|Cat 5e|1 Gbps|100m|Minimum for modern installs|
|Cat 6|1–10 Gbps|55m (10G) / 100m (1G)|Standard today|
|Cat 6a|10 Gbps|100m|Best for 10G runs|

> ⚠️ Cat 3 is the classic trap — it may "work" at low load, then collapse under real traffic. Don't inherit bad cabling without auditing it.

#### Fiber Optic

- Uses **pulses of light** through glass or plastic strands — no electrical signal
- Immune to EMI
- Much longer distances than copper
- Two main types:

|Type|Core Size|Distance|Use Case|
|---|---|---|---|
|**Single-mode (SMF)**|~9 microns|Up to 100km+|WAN, campus backbone, ISP|
|**Multi-mode (MMF)**|50–62.5 microns|Up to ~550m|Data center, building runs|

> Single-mode = laser light, long reach, expensive  
> Multi-mode = LED light, shorter reach, cheaper  
> "Mode" refers to how many paths light takes through the core

#### Wireless

- Data travels as **radio waves** (no physical medium)
- Convenient but introduces tradeoffs: interference, range limits, shared bandwidth
- Governed by 802.11 standards (a/b/g/n/ac/ax)
- Best for endpoints — not for uplinks or backbone connections where reliability matters

### Key Terms

|Term|Meaning|
|---|---|
|RJ45|Standard connector for twisted pair copper cabling|
|EMI|Electromagnetic Interference — disrupts copper signals|
|Single-mode fiber|Long-distance fiber using laser light, narrow core|
|Multi-mode fiber|Shorter-distance fiber using LED light, wider core|
|Cat 6a|Augmented Cat 6 — supports 10 Gbps at full 100m|
|Bandwidth|Maximum capacity of a connection (how wide the road is)|

### Real-World Connection

An enterprise building uses Cat 6a copper from wall ports to the access layer switch (short runs, cheap, reliable). Then **fiber uplinks** from the access switch to the distribution layer — those runs are longer and carry aggregated traffic. Wireless APs handle endpoint mobility but connect back to the switch via copper or fiber, not wirelessly.

### Exam Traps

1. **Cat 6 does NOT do 10 Gbps at 100m** — only 55m. At 100m it drops to 1 Gbps. Cat 6a does 10G at full 100m.
2. **Single-mode vs multi-mode** — Cisco loves testing which goes farther. Single-mode wins every time (laser, narrow core, long distance).
3. **Fiber is immune to EMI** — copper is not. If a question mentions interference, the answer usually involves fiber.