## Simple Explanation

`show interfaces` is the deep-dive command for a single interface. It looks
overwhelming at first — dozens of lines of output. The skill isn't memorizing all of
it. It's knowing which handful of counters to read when something is wrong, and what
each one is telling you about the health of the link.

## Why It Exists

Interfaces fail in ways that don't show up in a simple up/down status check. A link
can be "up" and still be quietly corrupting frames, dropping packets, or operating at
degraded speed. The counters in `show interfaces` are the router's running record of
everything that has passed through that interface — and everything that went wrong.
Without them, troubleshooting is guesswork.

## How It Works

### Start here — the quick scan

```
show ip interface brief       ← is it up? does it have an IP?
show interfaces GigabitEthernet0/1   ← deep dive on that specific interface
```

Always start with `show ip interface brief`. Only go deeper if something looks wrong
or feels slow.

### Reading the output — what actually matters

```
GigabitEthernet0/1 is up, line protocol is up
```
Two states, both must be up. First is the physical layer (cable, signal).
Second is the data link layer (protocol negotiation). If either is down, nothing works.

---

```
Hardware is CN Gigabit Ethernet, address is 00d0.58a5.2902
```
The MAC address of this interface. Useful for verifying which port a device is learned
on in the switch's MAC address table.

---

```
Full-duplex, 100Mb/s
```
**Full-duplex** means the interface can send and receive simultaneously — like a phone
call. Both directions at the same time.  
**Half-duplex** means only one side transmits at a time — like a walkie-talkie. One
person talks, the other waits.

In modern switched networks, full-duplex is expected. If you see half-duplex on a
switched link, or if one side is full and the other is half — that's a **duplex
mismatch**, and it will wreck performance and generate collisions.

---

```
txload 1/255, rxload 1/255
```
Traffic load on a scale of 1–255. 255 = fully saturated. 1 = nearly idle.
Cisco doesn't show a clean percentage — divide by 255 to get the rough percentage.
If these are climbing toward 255, the interface is getting hammered.

---

```
5 minute input rate 0 bits/sec, 0 packets/sec
5 minute output rate 0 bits/sec, 0 packets/sec
```
Current traffic rate averaged over the last 5 minutes. Useful for spotting live
congestion or confirming an interface is truly idle.

---

### The error counters — the real diagnostic value

#### Input side

```
0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored, 0 abort
```

| Counter | What it means |
|---------|---------------|
| Input errors | Total of all input-side errors — a summary counter |
| CRC | Frame arrived but failed the integrity check (see below) |
| Frame | Received a frame with an illegal size or bad framing |
| Overrun | Frames arriving faster than the router's buffer can handle |
| Ignored | Interface ignored frames because it ran out of buffer space |

**CRC errors** are the most important counter to watch. Every Ethernet frame includes
an FCS (Frame Check Sequence) — a value calculated from the frame's data. The
receiving device recalculates that value and compares it. If they don't match, the
frame is corrupt and the CRC counter increments.

Rising CRC errors → think **physical problem first**: bad cable, poor termination,
cable too long, interference, or a cheap uncertified patch cable. The link can be
"up" and still be silently corrupting data.

#### Output side

```
0 output errors, 0 collisions, 1 interface resets, 0 late collision
```

| Counter | What it means |
|---------|---------------|
| Output errors | Frames the router tried to send but couldn't |
| Collisions | Two devices tried to transmit at the same time (abnormal on switched links) |
| Late collision | Collision detected late in the frame — almost always a duplex mismatch |
| Interface resets | Router internally reset the interface to recover a software/hardware sync loss |

**Collisions** on a switched link are a red flag. In the old hub days, collisions were
normal — every device shared the same wire and had to take turns. With switches, each
port has its own dedicated segment, so collisions should not happen. Seeing them climb
on a switched interface usually means one thing: **duplex mismatch**.

**Late collisions** are worse than normal collisions. A collision happening *late* in
the frame is physically abnormal — it signals something is wrong with the link itself.
Duplex mismatch is the classic cause.

**Interface resets**: the router lost sync between its software and hardware for that
interface and reset it to recover. A single reset on an otherwise clean interface
(zero errors, zero collisions) is benign — it usually happened when the interface
first came up. A reset counter that keeps climbing on a live interface is worth
investigating.

## Key Terms

| Term | Meaning |
|------|---------|
| CRC | Cyclic Redundancy Check — integrity verification on each received frame |
| FCS | Frame Check Sequence — the checksum field inside an Ethernet frame |
| Full-duplex | Send and receive simultaneously |
| Half-duplex | One direction at a time — like a walkie-talkie |
| Duplex mismatch | One side is full, the other is half — causes collisions and performance collapse |
| Late collision | Collision detected abnormally late in a frame — strong indicator of duplex mismatch |
| Interface reset | Internal router recovery event when software/hardware sync is lost |
| txload / rxload | Transmit/receive load on a scale of 1–255 |

## Real-World Connection

A user calls in: "The network feels slow, but it's not down." You log into the router
and run `show interfaces`. You see CRC errors incrementing every few seconds. That one
counter just told you the problem is probably a bad cable or a poorly terminated jack —
not the routing config, not the firewall, not the ISP. You've just saved yourself hours
of chasing the wrong thing.

Alternatively: a switch port shows collisions climbing on what should be a full-duplex
link. Someone hardcoded one side to half-duplex and left the other on auto-negotiate.
Late collisions confirm it. Fix the duplex, collisions drop to zero.

## Exam Traps

1. **Collisions on a switched link are not normal.** On a hub, they were expected. On
   a switch, rising collisions mean duplex mismatch. Don't confuse the two environments.

2. **CRC errors point to physical problems, not config problems.** If you see them
   climbing, don't start editing the routing table. Check the cable first.

3. **Interface reset ≠ interface failure.** One reset with zero errors on an otherwise
   clean interface is benign. It's the trend over time that matters, not a single
   occurrence.

4. **Full-duplex ≠ two separate cables.** Full-duplex means simultaneous bidirectional
   communication. The physical wiring is separate at the PHY layer, but that's not what
   the duplex setting describes.

5. **txload/rxload uses a 255 scale, not 100.** If you see `128/255` that's roughly
   50% utilization — not 128%.

## Commands

```
show ip interface brief              ← first stop: status and IP overview
show interfaces GigabitEthernet0/1  ← full counter output for one interface
show interfaces description          ← status + description for all interfaces
```

## Recall Questions

1. An interface is `up/up` but users on that segment report random slowness and
   occasional dropped connections. What counters do you check first, and what physical
   problem would explain rising CRC errors?

2. You see collisions incrementing on a port connected to a modern managed switch.
   What is the most likely cause? What counter would confirm your suspicion?

3. What is the difference between full-duplex and half-duplex? What happens to network
   performance when one side is full and the other is half?

4. An interface shows `3 interface resets` with zero errors and zero collisions. Is
   this a problem? What would make you take it seriously?

5. txload shows `191/255` on an interface. Roughly what percentage utilization is that,
   and what does it suggest about the link?
