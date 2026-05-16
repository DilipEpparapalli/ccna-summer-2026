
## Simple Explanation

Ethernet copper cable carries data as electrical pulses across four twisted wire pairs. The twists reduce electromagnetic interference and maintain signal integrity. Every standard run tops out at 100 meters — the cable category determines what speed you get within that distance.

## Why It Exists

Wireless is convenient but unreliable for high-throughput, low-latency connections. Copper ethernet gives you a dedicated, predictable physical path between devices — cheap to manufacture, easy to terminate, and still fast enough for most endpoints in a building.

## How It Works

1. Data is converted to binary (0s and 1s), then represented as electrical pulses on the wire
2. Eight copper wires are arranged as four twisted pairs inside the cable jacket
3. Each pair is twisted at a different rate — the twist cancels out interference picked up from adjacent pairs (crosstalk) and external sources (EMI)
4. The signal travels up to 100m before degrading below usable levels
5. At the destination, the NIC decodes the electrical pulses back into binary

## Key Terms

|Term|Meaning|
|---|---|
|UTP|Unshielded Twisted Pair — standard copper cable, no extra shielding|
|STP|Shielded Twisted Pair — foil/braid shielding added for high-interference environments (industrial)|
|RJ45|Standard 8-pin connector used for ethernet|
|T568B|Most common wiring standard — defines pin-to-wire-color order in RJ45|
|T568A|Alternate wiring standard (used in some government/older installs)|
|Straight-through|Both ends use same standard (A–A or B–B) — used for unlike devices (PC to switch)|
|Crossover|One end T568A, other end T568B — used for like devices (switch to switch); obsolete with Auto-MDIX|
|Auto-MDIX|Hardware feature that detects the far end and flips TX/RX pins automatically — makes cable type irrelevant|
|EMI|Electromagnetic Interference — external signal noise that can corrupt data|

## Cable Categories

|Category|Max Speed|Max Distance|Notes|
|---|---|---|---|
|Cat5e|1 Gbps|100m|Still very common for endpoints|
|Cat6|10 Gbps|55m|10G capable but distance-limited|
|Cat6a|10 Gbps|100m|Best choice for new 10G runs|
|Cat7|10 Gbps|100m|Never widely adopted; skipped in most deployments|
|Cat8|40 Gbps|30m|Data center / switch uplinks only|

## Real-World Connection

In an enterprise environment, Cat5e handles most workstations and IP phones — 1 Gbps is sufficient for end-user traffic. Cat6a goes in for switch-to-switch uplinks or anywhere 10G headroom is wanted without sacrificing full 100m runs. Cat8 shows up in data center top-of-rack wiring where 40G is needed over short distances between servers and switches.

## Exam Traps

1. **T568A ≠ crossover cable.** T568A on both ends is a valid straight-through cable. A crossover requires T568A on one end _and_ T568B on the other. Many people mix this up.
2. **Cat6 is not reliable at 10 Gbps over 100m.** It tops out at ~55m for 10G. If a question says 80m run at 10 Gbps, Cat6 is wrong — Cat6a is the answer.
3. **100m is a design ceiling, not a target.** Patch panels, wall jacks, bends, and connectors eat into the budget. Plan below the limit, not at it.

## Commands (if applicable)

```
! No IOS commands for physical cabling, but useful show commands post-connection:

show interfaces GigabitEthernet0/1     ! Check link speed, duplex, errors
show interfaces GigabitEthernet0/1 counters  ! Input/output errors, CRC errors suggest bad cable
```

## Recall Questions

1. What is inside an ethernet cable, and why are the wires twisted?
2. You need a 10 Gbps run across 75 meters. Which cable category do you use and why?
3. What is the difference between a straight-through and a crossover cable? Does it matter if the switch has Auto-MDIX?
4. A cable run looks like it will be about 95 meters. What do you do?
5. What does Auto-MDIX actually do at the hardware level?