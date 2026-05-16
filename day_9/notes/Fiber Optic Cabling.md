
## Simple Explanation

Fiber optic cable transmits data as pulses of light through a glass core instead of electrical signals through copper wire. Because light doesn't suffer from electrical resistance or electromagnetic interference, fiber can carry data over vastly greater distances and at much higher speeds than copper.

## Why It Exists

Copper ethernet tops out at 100 meters per run and is vulnerable to EMI. Once a network needs to span buildings, campuses, or cities — or needs to move massive amounts of data at high speed — copper physically cannot do the job. Fiber solves both the distance problem and the bandwidth problem simultaneously.

## How It Works

1. Data is converted into pulses of light by a transceiver (inside an SFP module)
2. Light travels down a thin glass core inside the cable
3. The glass cladding around the core reflects light inward, keeping it on path
4. At the far end, the transceiver converts light pulses back into electrical signals the device understands
5. Because glass has no electrical resistance and is immune to EMI, the signal stays clean over long distances

## Single-Mode vs Multimode

|Property|Single-Mode (SMF)|Multimode (MMF)|
|---|---|---|
|Core diameter|~9 microns|~50–62.5 microns|
|Light path|Single straight path|Multiple bouncing paths|
|Distance|Miles (long-haul)|Hundreds of meters|
|Bandwidth|Higher|Lower|
|Cost|Higher (transceiver cost)|Lower|
|Typical use|Building-to-building, WAN, data center backbone|Intra-building, campus short runs|

**Why single-mode goes farther:** The narrow core forces light into one straight path. Multimode's wider core lets light travel at multiple angles simultaneously — those paths arrive at slightly different times, causing the signal to spread and degrade over distance. This is called **modal dispersion**.

**Note:** Both types use glass cores. The difference is core size, not material.

## Connecting Fiber to a Switch

Standard switches have RJ45 copper ports — fiber doesn't plug directly in. The solution:

```
[Fiber Cable] → [LC Connector] → [SFP Module] → [Switch SFP Slot]
```

- **SFP** (Small Form-factor Pluggable) — a hot-swappable module that slots into the switch and gives it a fiber interface
- **LC connector** — the most common fiber connector type used with SFPs; small, latching, duplex
- The entire chain must match: SFP type must match the fiber type (SMF SFP for single-mode fiber, MMF SFP for multimode)

## Fiber vs Copper — When to Use Which

|Scenario|Use|
|---|---|
|Desktop, VoIP phone, WAP, IP camera|Copper (PoE capability needed)|
|Switch-to-switch uplinks, long runs|Fiber|
|Cross-building or campus backbone|Fiber|
|Devices needing Power over Ethernet|Copper only — fiber carries no electrical power|
|40/100 Gbps high-speed links|Fiber|

## Key Terms

|Term|Meaning|
|---|---|
|SMF|Single-Mode Fiber — narrow core, long distance|
|MMF|Multimode Fiber — wider core, shorter distance|
|Modal dispersion|Signal degradation in MMF caused by light taking multiple paths at different angles|
|SFP|Small Form-factor Pluggable — module that gives a switch a fiber (or copper) interface|
|LC connector|Most common fiber connector used with SFPs|
|PoE|Power over Ethernet — electrical power delivered over copper cable alongside data; impossible over fiber|
|Transceiver|Device that converts electrical signals to light and back|

## Real-World Connection

In an enterprise environment, the typical design is a hybrid: fiber runs the backbone — switch-to-switch uplinks, inter-floor and inter-building connections, server room links. Copper runs to every endpoint — workstations, phones, access points, cameras — because those devices need PoE or are close enough that fiber's distance advantage doesn't matter. Neither technology replaces the other; they each own their lane.

## Exam Traps

1. **Both SMF and MMF use glass cores.** The difference is core diameter, not material. Plastic fiber exists but is a niche product — don't confuse it with standard MMF.
2. **Fiber cannot deliver PoE.** There is no electrical conductor in a fiber cable. Any device needing power over the cable must use copper — no exceptions.
3. **SFP must match the fiber type.** A single-mode SFP used with multimode fiber (or vice versa) will not work correctly. Match the whole chain: switch port → SFP module → fiber type → connector.

## Commands (if applicable)

```
! Check SFP module details and fiber link status:
show interfaces GigabitEthernet0/1 transceiver

! Verify link is up on a fiber uplink port:
show interfaces GigabitEthernet0/1
! "line protocol is up" = fiber link established
! No link = check SFP seated, fiber connected, correct SFP for fiber type
```

## Recall Questions

1. Why does fiber support longer distances than copper? Give the physics reason, not just "it uses light."
2. What is modal dispersion and which fiber type suffers from it?
3. You need to connect two buildings 800 meters apart. Which fiber type do you choose and why?
4. A wireless access point needs to be installed in a ceiling 60 meters from the switch. Can you run fiber to it? Why or why not?
5. What is an SFP and why is it needed when connecting fiber to a switch?