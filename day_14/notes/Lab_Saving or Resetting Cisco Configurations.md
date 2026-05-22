
## What I Did
Explored how Cisco devices store configuration across two memory types, practiced saving and verifying configs, performed a factory reset using `write erase`, and discovered real behavioral differences between emulated and physical hardware.

**Device:** `Cafe-01-SW1` → renamed `Cafe-01-Temp` mid-lab to prove running/startup divergence  
**IOS Version:** 17.16.1a (IOS-XE on emulated Linux platform)

---

## The Core Concept: Two Configs, Two Memory Locations

```
+--------------------------+        boot / reload        +--------------------------+
|         NVRAM            |  --------------------------> |          RAM             |
|   startup-config         |                             |   running-config         |
|   (survives power loss)  |                             |   (lost on reload)       |
+--------------------------+                             +--------------------------+
```

- **RAM** — where the switch lives right now. All active config is here. Lost on reload.
- **NVRAM** — where the switch remembers. Startup config lives here. Survives power loss.
- On modern IOS-XE, NVRAM is emulated by flash storage — same concept, different physical implementation.

**Boot sequence:**
1. IOS loads from flash into RAM
2. Switch looks for startup-config in NVRAM
3. If found → plays it out line by line into running-config
4. If not found → boots to factory defaults (blank slate)

---

## What I Did Step by Step

### Task 1 — Updated Banner, Then Compared RAM vs NVRAM

Changed the MOTD banner in running config, then checked both stores before saving:

```
Cafe-01-SW1# show running-config   ← shows new banner (RAM)
Cafe-01-SW1# show startup-config   ← startup-config is not present
```

This proved the two can be out of sync. Changes in RAM don't automatically persist.

---

### Task 2 — Saved the Baseline

```
Cafe-01-SW1# copy running-config startup-config
Destination filename [startup-config]?
Building configuration...
[OK]
```

After saving, `show startup-config` matched `show running-config` exactly.

---

### Task 3 — Proved It Survived a Reload

```
Cafe-01-SW1# reload
Proceed with reload? [confirm]
```

When asked to save before reload — answered **no** (config was already saved).

After boot: hostname `Cafe-01-SW1` confirmed in prompt and `show running-config | include hostname`.

Key lesson: the startup-config loaded from NVRAM into RAM automatically on boot.

---

### Task 4 — Proved Running vs Startup Divergence

Changed hostname in running config only:

```
Cafe-01-SW1(config)# hostname Cafe-01-Temp
Cafe-01-Temp#
```

Prompt immediately showed `Cafe-01-Temp`. But `show startup-config` still showed `hostname Cafe-01-SW1`.

Two different values. Same device. That's the divergence — RAM changed, NVRAM didn't.

---

### Task 5 — Factory Reset with `write erase`

```
Cafe-01-Temp# write erase
Erasing the nvram filesystem will remove all configuration files! Continue? [confirm]
[OK]
Erase of nvram: complete

Cafe-01-Temp# show startup-config
startup-config is not present
```

NVRAM cleared. But the running config in RAM was still fully active — `Cafe-01-Temp` prompt, all passwords still working. `write erase` only touches NVRAM.

---

### Task 6 — Reloaded Without Saving

```
Cafe-01-Temp# reload
System configuration has been modified. Save? [yes/no]: no   ← critical — said NO
Proceed with reload? [confirm]
```

**Said no** — if you say yes here, you copy the running config back into startup config and undo the entire wipe. The reload save prompt is a trap.

After boot: device came up to a near-default state.

---

### Task 7 — Alternative Wipe via NVRAM Direct Delete

On this emulated platform, there was no `flash:` filesystem visible — only `nvram:`. The lab referenced `delete flash:config.text` which applies to older physical IOS. On IOS-XE the config lives in `nvram:startup-config`.

```
Cafe-01-SW1# dir nvram:
126     -rw-     0     startup-config    ← 0 bytes after write erase

Cafe-01-SW1# delete nvram:startup-config
Delete nvram:startup-config? [confirm]

Cafe-01-SW1# show startup-config
Using 5 out of 131072 bytes
end
```

Discovered: on this emulated platform, deleting the startup-config file causes the platform to **recreate it automatically** (the virtual filesystem expects the file to exist). `show startup-config` returned `end` instead of "not present." This is emulator behavior — on real physical hardware, the file is genuinely gone.

---

### Task 8 — Rebuilt the Baseline

Reconfigured all essentials from scratch after the wipe:

```
hostname Cafe-01-SW1
banner motd #
Castle Rysen Ops: Authorized engineers only.
#
enable secret C4stleRysen!
line console 0
 password VaultAccess
 login
line vty 0 4
 password ShelterAccess
 login
 transport input ssh
```

Saved with `copy running-config startup-config` and verified both files matched.

---

## Key Terms

| Term | Meaning |
|------|---------|
| RAM | Volatile memory — holds running-config, lost on reload |
| NVRAM | Non-volatile memory — holds startup-config, survives power loss |
| Running Config | Active live configuration in RAM |
| Startup Config | Saved configuration in NVRAM, loaded at boot |
| `write erase` | Clears NVRAM — does NOT touch RAM |
| `reload` | Reboots device — clears RAM, loads startup-config from NVRAM |
| `copy run start` | Copies RAM → NVRAM (saves current config) |

---

## The Full Reset Workflow

```
1. show running-config      ← confirm what's active
2. show startup-config      ← confirm what's saved
3. write erase              ← clear NVRAM
4. show startup-config      ← verify "not present"
5. reload                   ← reboot
6. Save? → NO               ← critical — don't re-save the old config
7. Confirm reload
8. Verify clean boot
```

---

## Real-World Discovery: Emulator vs Physical Hardware

| Behaviour | Physical Cisco Switch | This Emulated Switch (IOS-XE on Linux) |
|---|---|---|
| `write erase` result | startup-config genuinely gone | File recreated by platform layer |
| `delete nvram:startup-config` | File deleted permanently | Platform auto-recreates the file |
| Post-wipe reload | True clean slate | May retain some config artifacts |
| `dir flash:` | Shows flash filesystem | Not visible — only `nvram:` present |

**Why:** The emulated switch runs IOS-XE on a Linux host. Its "NVRAM" is a hosted file managed by the hypervisor. The platform recreates the startup-config file automatically when it's deleted because the virtual filesystem expects it to exist. Real hardware has no such behavior.

**Ground truth:** Always verify reset behavior on real hardware before trusting emulator results in production.

---

## Exam Traps

1. **The reload save prompt is a trap** — if you just ran `write erase` and then answer `yes` to "Save before reload?", you copy the running config straight back into startup config. The wipe is undone. Always answer **no** after an intentional wipe.

2. **`write erase` does not touch RAM** — the running config keeps running after `write erase`. The device doesn't forget its current config until the reload clears RAM.

3. **Unsaved changes are lost on reload** — if you made changes and didn't `copy run start`, a reload wipes them. The switch comes back with whatever was last saved to NVRAM.

4. **`show startup-config` vs `show running-config`** — these can show different things. A changed hostname in RAM won't appear in startup until saved. The exam shows mismatched configs and asks which one survives a reload.

---

## Recall Questions

1. You run `write erase` and immediately check `show running-config`. Does it still show the old hostname? Why?
2. After `write erase`, the device asks "Save before reload?" — what do you type and why?
3. What is the boot sequence from power-on to active config?
4. You changed the hostname in global config but never saved. The switch loses power. What hostname comes back when it boots?
5. On a physical switch, what is the practical difference between `write erase` and `delete nvram:startup-config`?
