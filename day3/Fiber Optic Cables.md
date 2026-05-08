
## Simple Explanation

Fiber optic cables transmit data as pulses of light through a thin glass or plastic core rather than electrical signals over copper. One cable carries data at speeds and distances that copper physically cannot match. The core diameter determines whether a cable is single mode or multimode — and that single difference drives everything else.

## Why It Exists

Copper Ethernet hits a hard wall at 100 meters and degrades with electromagnetic interference. As networks scaled to connect buildings, campuses, data centers, and continents, a medium was needed that could go farther, carry more data, and ignore electrical noise. Fiber solves all three problems simultaneously.

## How It Works

### The Physics — Total Internal Reflection

A fiber cable has two concentric layers:

- **Core** — the tiny center where light actually travels
- **Cladding** — surrounds the core; different optical density causes light to reflect inward rather than escape

When light hits the cladding at or beyond the **critical angle**, it reflects back into the core instead of passing through. This is called **total internal reflection** — the mechanism that lets light travel through kilometers of cable, including gentle bends, without escaping.

```
Cross-section:
    [ Jacket — physical protection      ]
    [ Strengthening fibers              ]
    [ Buffer coating                    ]
    [======= Cladding (125µm) ==========]
    [======== CORE (light path) =========]
    [======= Cladding (125µm) ==========]
```

> Never kink fiber. It is glass or plastic. It snaps. Unlike copper, you cannot re-terminate it with a cheap tool.

### Why Fiber Wins Over Copper

|Advantage|Detail|
|---|---|
|Speed|Light travels at ~70% the speed of light in a vacuum. Hundreds of Tbps is achievable.|
|Distance|Single mode reaches 100km without a repeater. Copper caps at ~100m.|
|No EMI|Light generates zero electromagnetic interference. Copper carries electricity and does.|

---

## Single Mode vs Multimode

### The Core Diameter Difference

|Type|Core Diameter|Light Behavior|
|---|---|---|
|Multimode|50–62.5µm|Light bounces off cladding walls — **modal dispersion**|
|Single mode|9µm|Core so narrow, light travels in one straight path — no dispersion|

```
Multimode (50µm core):          Single Mode (9µm core):

  |~~~~~~~~~~~~~~~~~~~~~|          |~~~|
  | \/\/\/\/\/\/\/\/\/\ |          | → |
  | /\/\/\/\/\/\/\/\/\/ |          | → |
  |~~~~~~~~~~~~~~~~~~~~~|          |~~~|

  Bouncing = modal dispersion       Straight = no dispersion
  Higher attenuation                Lower attenuation
  Shorter distance                  Much longer distance
```

**Modal dispersion** — different light rays travel slightly different path lengths, arriving at slightly different times. This smears the signal and is why multimode has a distance limit despite having a physically larger core.

### Standards

**Multimode (orange or aqua jacket):**

|Type|Core|10G Distance|Light Source|
|---|---|---|---|
|OM1|62.5µm|33m|LED|
|OM2|50µm|82m|LED|
|OM3|50µm|300m|VCSEL laser|
|OM4|50µm|400m|VCSEL laser|
|OM5|50µm|400m|VCSEL (wideband)|

**Single mode (yellow jacket):**

|Type|Core|Distance|
|---|---|---|
|OS1|9µm|up to 10km|
|OS2|9µm|up to 100km|

Single mode uses precision lasers at 1310nm and 1550nm wavelengths. Multimode uses cheaper LEDs or VCSELs at 850nm and 1300nm. That light source cost difference is a major reason single mode is more expensive overall.

### When to Use Each

|Scenario|Fiber Type|Why|
|---|---|---|
|Floor-to-floor inside a building|Multimode OM3/OM4|Short distance, no need to pay for single mode|
|Building-to-building on a campus|Single mode|Exceeds multimode range|
|ISP delivering internet to a business|Single mode|Long haul|
|Data center switch uplinks|Multimode OM4|High bandwidth, short runs in rack/row|
|Undersea / intercontinental|Single mode + repeaters|100km+ spans|

**The economic principle:** single mode is technically superior in every metric, but multimode is sufficient and cheaper for short distances. Don't pay for capability you don't need.

---

## Connectors

|Connector|Notes|
|---|---|
|LC|Small, latching — most common in modern enterprise gear|
|SC|Older, push-pull — still seen in legacy installs|
|ST|Bayonet twist-lock — legacy, rarely seen in new deployments|

Most fiber cables are **duplex** — two strands, one for TX (transmit) and one for RX (receive). Always verify connector type before ordering. Unlike copper, you cannot re-terminate fiber on site.

---

## SFP Modules

Switches often have SFP ports instead of fixed fiber ports. An SFP (Small Form-factor Pluggable) is a slot that becomes whatever module you insert:

```
Switch chassis:
┌──────────────────────────────────────┐
│  [RJ45] [RJ45] [RJ45] ... [SFP][SFP] │
└──────────────────────────────────────┘
                                ↑
          Insert copper SFP module  → RJ45 port
          Insert fiber SFP module   → LC fiber port
          Same physical slot, completely different capability
```

|Module|Speed|
|---|---|
|SFP|1G|
|SFP+|10G|
|QSFP / QSFP+|40G / 100G|

Always match: SFP module type → fiber type (single mode vs multimode) → connector type → distance rating.

---

## Fiber vs Copper — When to Use What

|Factor|Fiber|Copper|
|---|---|---|
|Max distance|100km (single mode)|~100m|
|Bandwidth|Effectively unlimited|Limited by category|
|EMI immunity|Full immunity|Vulnerable|
|PoE support|❌ Not possible|✅ Native|
|Cost|Higher|Lower|
|Termination|Requires specialized tools|Simple crimping|
|Endpoint support|Needs SFP or fiber NIC|Universal RJ45|

**The rule:** fiber for backbone and uplinks, copper for endpoints.

---

## Real-World Connection

**Enterprise campus layout:**

```
[Building A Core Switch] ══ Single Mode OS2 ══ [Building B Core Switch]
         |                  (campus backbone)              |
   [Floor switches] ── Multimode OM3/OM4 ──         [Floor switches]
         |              (within building)                  |
   [Access switches] ── Copper Cat6 ──             [Access switches]
         |               (to endpoints)                    |
   [PCs, phones, APs, cameras]                    [PCs, phones, APs]
```

**ISP delivery:** AT&T fiber arriving at a building is single mode — it had to travel kilometers from the central office to reach you. Inside the building, you may convert to copper or multimode for distribution.

---

## Exam Traps

1. **Color code matters.** Yellow = single mode. Orange or aqua = multimode. The exam will describe a cable color and expect you to identify the type instantly.
    
2. **Bigger core ≠ better performance.** Multimode has a larger core but shorter distance and lower bandwidth than single mode. The physics of modal dispersion explains why. Don't let "bigger pipe" intuition mislead you.
    
3. **SFP ≠ fiber.** SFP is a slot type. You can insert a copper SFP module or a fiber SFP module. The exam may describe an SFP port and ask what type of cable it uses — the answer depends entirely on the module inserted, not the slot itself.
    
4. **Fiber cannot deliver PoE.** Power over Ethernet requires copper. This is a hard limit. No fiber PoE standard exists for standard network deployments.
    

---

## Commands (if applicable)

```ios
! Verify SFP module type inserted in interface
show interfaces GigabitEthernet1/0/1 transceiver

! Check interface status including fiber uplinks
show interfaces status

! Check all transceiver modules installed
show inventory
```

---

## Recall Questions

1. What is the core diameter of single mode fiber? Multimode fiber?
2. What is modal dispersion and which fiber type suffers from it?
3. What color jacket indicates single mode fiber? Multimode?
4. A network engineer needs to connect two buildings 4km apart. Which fiber type is required and why?
5. What is an SFP port and why is it more flexible than a fixed fiber port?
6. Why can't fiber deliver PoE?
7. OM3 multimode — what is its maximum distance at 10G?
8. What light sources does multimode use? What does single mode use? Why does this affect cost?