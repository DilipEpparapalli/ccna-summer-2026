
## Simple Explanation

PoE lets a single Ethernet cable carry both data and electrical power to a device simultaneously. The switch delivers power; the end device consumes it. One cable run replaces what used to require a network drop and a separate electrical outlet.

## Why It Exists

Deploying IP phones, wireless access points, and security cameras traditionally required two separate installations — a network engineer for data cabling and an electrician for power. PoE eliminates that dependency. The network team owns the entire deployment, devices can be repositioned without an electrician, and total installation cost drops significantly.

## How It Works

### The Two Roles

Every PoE deployment has exactly two roles:

|Role|Full Name|Example|
|---|---|---|
|PSE|Power Sourcing Equipment|Your switch|
|PD|Powered Device|IP phone, camera, AP, Pi 5|

The PSE sources power onto the Ethernet cable. The PD receives it.

### How Power Travels the Cable

Cat5e/Cat6 cable has 4 twisted pairs (8 wires total). Early PoE used the idle pairs (pins 4, 5, 7, 8) that standard data transmission didn't need. Modern 802.3bt uses all 4 pairs simultaneously — that's how higher wattage is achieved. Some power is lost as heat across the copper, so a port rated at 30W delivers approximately 27W to the device.

### The Standards

|IEEE Standard|Common Name|Type|Per-Port Max|Typical Use|
|---|---|---|---|---|
|802.3af|PoE|Type 1|15.4W|Legacy IP phones, basic cameras|
|802.3at|PoE+|Type 2|30W|Modern APs, Raspberry Pi 5, HD cameras|
|802.3bt|PoE++|Type 3|60W|High-power APs, powering downstream switches|
|802.3bt|PoE++|Type 4|90W|Laptops, LED lighting systems, HVAC sensors|

All standards are backward compatible. A Type 4 switch can power a Type 1 device — the negotiation handles it.

### Active vs Passive PoE

**Active PoE** — negotiates before delivering power:

```
Switch: "Who are you? What do you need?"
Device: "I'm a camera. I need 12.5W."
Switch: "Confirmed. Here's 12.5W."
```

Negotiation uses **CDP** (Cisco Discovery Protocol) for Cisco devices or **LLDP** (Link Layer Discovery Protocol) for everything else. Safe to use with any device.

**Passive PoE** — always on, no negotiation:

```
Switch: *sends power regardless of what's plugged in*
Wrong device: fried
```

Used in some Ubiquiti gear (24V passive). Only deploy when you know exactly what's plugged in. Connecting an unsupported device can permanently damage it.

### The Budget Problem

Per-port wattage is only half the picture. Every switch has a total PoE power budget shared across all ports. A switch with 48 PoE+ ports at 30W each would theoretically need 1,440W — most switch PSUs provide far less. Ports power up first-come, first-served. When the budget runs out, subsequent devices don't power on.

```
show power inline
```

This command shows:

- Total watts available
- Total watts currently used
- Watts remaining
- Per-port draw and device class

If a device tries to draw more than its port is configured to allow, the switch error-disables that port and logs a syslog message.

## Key Terms

|Term|Meaning|
|---|---|
|PSE|Power Sourcing Equipment — the device delivering power (switch)|
|PD|Powered Device — the device receiving power (phone, camera, AP)|
|CDP|Cisco Discovery Protocol — Cisco's PoE negotiation protocol|
|LLDP|Link Layer Discovery Protocol — industry-standard negotiation protocol|
|PoE Budget|Total watts the switch can distribute across all PoE ports simultaneously|
|Error-disabled|Port state when a device exceeds its allocated power draw|
|Cable loss|Power dissipated as heat across copper; typically 2–3W per run|

## Real-World Connection

**Enterprise scenario — office deployment:** A 48-port switch with a 740W PoE budget is used to power desk phones across a floor. Each phone draws ~10W. The switch can comfortably support all 48 phones (48 × 10W = 480W, well within budget). Adding 10 high-power Wi-Fi 6 APs drawing 25W each changes the math: 480W + 250W = 730W — cutting it close. Adding one more AP tips the budget. The last AP refuses to power on. `show power inline` reveals the problem immediately.

**Pi 5 home lab:** The Raspberry Pi 5 draws ~27W under load. PoE+ (802.3at, Type 2, 30W) is the minimum standard required. After cable loss (~3W), delivered power is approximately 27W — right at the Pi's requirement. Use a quality cable and verify the delivered wattage with `show power inline`.

## Exam Traps

1. **Per-port wattage ≠ available budget.** A switch advertised as "30W per port, 48 ports" does not guarantee simultaneous full-power delivery to all 48 ports. Always check the total PoE budget spec.
    
2. **Passive PoE is dangerous.** The exam may present a scenario where a device gets damaged or fails to operate after being plugged into a switch. If the switch uses passive PoE, no negotiation occurs — the wrong device receives full voltage and fries. Active PoE prevents this.
    
3. **Standard numbers matter.** The exam uses 802.3af, 802.3at, and 802.3bt — not "Type 1/2/3/4" alone. Know both the IEEE standard number and the type designation, and map each to its wattage.
    

## Commands

```ios
! Check PoE status on all ports
show power inline

! Check PoE status on a specific interface
show power inline GigabitEthernet1/0/1

! Configure maximum power on an interface
interface GigabitEthernet1/0/1
 power inline consumption 15400     ! milliwatts

! Enable PoE policing (error-disables port if device exceeds limit)
interface GigabitEthernet1/0/1
 power inline police
```

## Recall Questions

1. What is the difference between a PSE and a PD? Give an example of each.
2. A switch port is rated at 30W. A connected device draws 28W. How much power does the device actually receive, and why?
3. Your switch supports PoE on all 48 ports at 30W each, but you have a 740W total budget. How many ports can run at full draw simultaneously before the budget is exhausted?
4. An engineer plugs a laptop into a switch port configured for passive PoE. What happens and why?
5. What two protocols does a Cisco switch use to negotiate power with connected devices? When does it use each?
6. What command do you run to see how much PoE budget remains on a switch?
7. A Wi-Fi 6E access point requires 25W. Which PoE standard is the minimum required, and what is its IEEE designation?