
## Simple Explanation

Ethernet devices have dedicated pins for transmitting (TX) and receiving (RX). For two devices to communicate, one side's TX must land on the other side's RX. The cable type — straight-through or crossover — determines whether that happens correctly.

## Why It Exists

Different device types have opposite TX/RX pin assignments by design. Connecting unlike devices (PC → switch) naturally lines up. Connecting like devices (switch → switch) creates a mismatch — both sides TX on the same pins, nobody is listening. The crossover cable was the fix before hardware solved it automatically.

## How It Works

### Pin assignments (10/100 Mbps Ethernet)

|Device Type|Transmits On|Receives On|
|---|---|---|
|End device (PC, router)|Pins 1 & 2|Pins 3 & 6|
|Switch|Pins 3 & 6|Pins 1 & 2|

### Straight-through cable

- Pin 1 → Pin 1, Pin 2 → Pin 2, Pin 3 → Pin 3, Pin 6 → Pin 6
- Works when TX pins on one side map to RX pins on the other
- Use case: **unlike devices** — PC to switch, router to switch

### Crossover cable

- Pin 1 → Pin 3, Pin 2 → Pin 6, Pin 3 → Pin 1, Pin 6 → Pin 2
- Physically swaps TX and RX pairs inside the cable
- Use case: **like devices** — switch to switch, PC to PC, router to router
- Wired as T568B on one end, T568A on the other

### Auto MDI-X

- Hardware feature on modern switches and NICs
- Detects what's connected on the far end and internally flips which pins it uses for TX/RX
- Result: cable type doesn't matter — straight-through works everywhere
- Standard on virtually all gear made in the last 15+ years

## Key Terms

|Term|Meaning|
|---|---|
|TX|Transmit — pins the device sends data on|
|RX|Receive — pins the device listens on|
|Straight-through|Same pin mapping both ends — for unlike devices|
|Crossover|TX/RX pairs swapped — for like devices|
|Auto MDI-X|Hardware auto-detection that makes cable type irrelevant|
|802.3|IEEE standard family governing Ethernet — ensures vendor interoperability|

## Real-World Connection

In a modern enterprise environment, Auto MDI-X means engineers use standard patch cables everywhere and don't think about crossover cables day-to-day. But older gear — budget switches, embedded devices, legacy industrial equipment — may not have Auto MDI-X. A link that won't come up between two old switches is a classic crossover cable problem. First troubleshooting check: link lights. No link light at Layer 1 means the physical connection is broken before anything else matters.

## Exam Traps

1. **Crossover = T568B on one end + T568A on the other.** T568A alone is not a crossover cable — it's just an alternate straight-through standard.
2. **Router-to-router needs a crossover**, just like PC-to-PC. Routers are end devices, not switches — their pin assignments match PCs, not switches.
3. **Auto MDI-X doesn't rewire the cable** — it changes which pins the device internally uses for TX/RX. The cable itself is unchanged.

## Commands (if applicable)

```
! Verify link status and duplex after connecting devices:
show interfaces GigabitEthernet0/1

! Look for:
! "line protocol is up" = Layer 1 and Layer 2 are good
! "line protocol is down" = likely a physical/cable issue
! Input errors, CRC errors = bad cable or duplex mismatch
```

## Recall Questions

1. A PC connects to a switch. Which cable type — and why does it work at the pin level?
2. Two routers connect directly. Which cable type — and why?
3. Auto MDI-X is enabled on a modern switch. Does it matter if you use straight-through or crossover? Why?
4. You're in a server room with old gear and a link won't come up between two switches. What's your first physical-layer suspicion and how do you confirm the fix worked?