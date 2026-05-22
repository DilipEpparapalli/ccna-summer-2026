
## Simple Explanation
When a Cisco device powers on, it loads its operating system from flash memory into RAM, applies the saved configuration from NVRAM, and then presents you with a CLI prompt. If any stage fails, the device stops there — and the console output tells you exactly where.

## Why It Exists
Understanding the boot sequence turns a "dead switch" from a mystery into a diagnostic puzzle. Each stage has a specific failure signature. If you can read the console output during startup, you can pinpoint the problem before you even touch a config command.

## How It Works

### Boot Sequence (in order)

```
Power On
    |
    v
POST — Power-On Self Test
    Hardware check: CPU, memory, interfaces
    |
    v
Bootstrap / ROM
    Minimal startup code burned into ROM
    Looks for IOS image
    |
    v
Flash Initialization
    Loads compressed IOS image from flash into RAM
    ← Most common failure point
    |
    v
IOS Loads into RAM
    Decompresses and executes
    |
    v
Hardware Verification
    Checks ASICs, interfaces, line cards
    |
    v
NVRAM Read
    Loads startup-config into running-config
    If no startup-config found → runs Setup wizard
    |
    v
"Press RETURN to get started"
    Device is fully operational
    Drops into User EXEC mode
```

### Memory Map — What Lives Where

| Memory | Contents | Persistent? |
|--------|----------|-------------|
| ROM | Bootstrap code, ROMMON | Yes (read-only) |
| Flash | IOS image file | Yes |
| RAM | Running IOS + running-config | No — lost on reboot |
| NVRAM | startup-config | Yes |

**The critical rule:**
- Flash → holds IOS
- NVRAM → holds startup-config
- RAM → where everything runs (volatile — gone on reboot)

### Reading Boot Failure Clues

| Where it stops | Likely cause |
|----------------|--------------|
| No output at all | Bad hardware, no power to console |
| Stops at POST | CPU or memory hardware failure |
| Hangs at "Initializing flash" | Flash corruption or failure |
| Loads IOS then stops | IOS image corrupted |
| Boots fine, no config applied | Startup-config missing or NVRAM issue |
| "Press RETURN to get started" | Full success |

## Key Terms

| Term | Meaning |
|------|---------|
| POST | Power-On Self Test — hardware self-check at startup |
| ROM | Read-Only Memory — holds the bootstrap code |
| ROMMON | ROM Monitor — recovery mode if IOS can't load |
| Flash | Non-volatile storage where IOS image lives |
| RAM | Volatile working memory where IOS runs |
| NVRAM | Non-volatile RAM — stores startup-config across reboots |
| startup-config | The saved configuration loaded at boot (from NVRAM) |
| running-config | The live configuration in RAM right now |
| ASIC | Application-Specific Integrated Circuit — hardware chip that handles fast packet forwarding |

## Real-World Connection

On an enterprise campus with 50+ switches, flash failure during reboot is a real
operational risk. Devices that have been running for years without rebooting may
have aging flash. Best practice: before any planned reboot, verify the IOS image
integrity (`verify` command), confirm you have a backup config, and have a spare
unit staged if the device is critical. A reboot is not a trivial action on production
gear.

## Exam Traps

1. **RAM vs NVRAM confusion** — Running-config lives in RAM (lost on reboot).
   Startup-config lives in NVRAM (persists). Forgetting `write memory` after making
   changes means your config vanishes on the next power cycle. The exam tests this
   constantly.

2. **Flash stores IOS, not the config** — The exam may try to trick you into thinking
   the config is in flash. It's not. Config = NVRAM. IOS = Flash.

3. **If startup-config is missing, the device runs Setup mode** — it does NOT just
   boot with a blank config silently. The device will prompt you through the setup
   wizard. Knowing this matters for first-time device setup scenarios.

## Commands

```
! --- Verify flash contents (IOS image) ---
Switch# show flash:
Switch# dir flash:

! --- Verify NVRAM (startup config) ---
Switch# show startup-config

! --- Verify what's running now ---
Switch# show running-config
Switch# show version              ! shows IOS version, uptime, flash/RAM sizes

! --- Save running config to NVRAM (so it survives reboot) ---
Switch# write memory
Switch# copy running-config startup-config    ! same thing, explicit form

! --- Erase startup config (factory reset) ---
Switch# write erase
Switch# reload
```

## Recall Questions

1. A switch powers on. You see POST messages, then it says "Initializing flash..."
   and freezes. What is the most likely cause? What would you do next?

2. You spend 30 minutes configuring a switch. The building loses power.
   When the switch comes back, all your changes are gone. What happened,
   and what should you have done?

3. What is the difference between startup-config and running-config?
   Where does each one live?

4. A switch boots successfully but immediately drops into the Setup wizard
   instead of showing a normal prompt. What does that tell you?

5. True or false: Flash memory is where the running configuration is stored.
   Explain your answer.
