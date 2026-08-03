## Objective
Inspect default Rapid PVST access-port behavior, enable PortFast on true access ports, then arm BPDU Guard so a rogue switch plugged into the access edge gets automatically quarantined before it can cause a loop.

## Topology

```
                    trunk (Et0/0)
   Shelter-Core ------------------ Bunker-SW1
                                      |  \
                              Et0/3   |   \  Et1/0
                                      |    \
                                 Bunker-Host  Rogue-SW (Et0/0)
                                 (end device)  (simulated rogue switch)
```
![[ccna-summer-2026/day_41/lab/Network_Diagram.png]]

| Interface | Connects to | Role | Config applied |
|-----------|-------------|------|-----------------|
| Et0/0 | Shelter-Core | Trunk uplink | Untouched — no PortFast, no BPDU Guard |
| Et0/3 | Bunker-Host | Access (VLAN 10) | PortFast + BPDU Guard |
| Et1/0 | Rogue-SW Et0/0 | Access (VLAN 10) | PortFast + BPDU Guard |

## Task 0 — Baseline: Default Behavior Before PortFast

Checked interface status and STP state, then bounced Et0/3 to observe default Rapid PVST transition behavior.

```
Bunker-SW1#show ip interface brief
Bunker-SW1#show spanning-tree
Bunker-SW1#show vlan brief
Bunker-SW1#show interfaces trunk
```

`show interfaces trunk` confirmed Et0/0 as the only trunk, native VLAN 1, VLAN 10 allowed:

```
Port           Mode             Encapsulation  Status        Native vlan
Et0/0          on               802.1q         trunking      1
Port           Vlans allowed on trunk
Et0/0          10
```

Bounced the port to mimic a user reconnecting:

```
Bunker-SW1(config)#interface ethernet0/3
Bunker-SW1(config-if)#shutdown
Bunker-SW1(config-if)#no shutdown
```

Caught the transition mid-flight on the first detail check:

```
Port 4 (Ethernet0/3) of VLAN0010 is designated learning
  Timers: message age 0, forward delay 13, hold 0
  Number of transitions to forwarding state: 0
```

Second check showed it complete:

```
Port 4 (Ethernet0/3) of VLAN0010 is designated forwarding
  Timers: message age 0, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
```

Confirms default Rapid PVST behavior: the port passes through **learning** before it's allowed to forward — this is the delay PortFast exists to eliminate.

## Task 1 — Enable PortFast

Configured both access ports at once, explicitly setting access mode first:

```
Bunker-SW1(config)#interface range ethernet 0/3, ethernet 1/0
Bunker-SW1(config-if-range)#switchport mode access
Bunker-SW1(config-if-range)#spanning-tree portfast
```

IOS immediately returned its standard safety warning:

```
%Warning: portfast should only be enabled on ports connected to a single
 host. Connecting hubs, concentrators, switches, bridges, etc... to this
 interface  when portfast is enabled, can cause temporary bridging loops.
 Use with CAUTION

%Portfast will be configured in 2 interfaces due to the range command
 but will only have effect when the interfaces are in a non-trunking mode.
```

Verified via the STP table — both ports now show `Edge` in the Type column, RSTP's marker for a PortFast-enabled edge port:

```
Et0/3               Desg FWD 100       128.4    P2p Edge
Et1/0               Desg FWD 100       128.5    P2p Edge
```

Bounced Et0/3 again to confirm instant forwarding with no learning delay:

```
Port 4 (Ethernet0/3) of VLAN0010 is designated forwarding
  Timers: message age 0, forward delay 0, hold 0
  Number of transitions to forwarding state: 1
  The port is in the portfast mode
```

## Task 2 — Arm BPDU Guard and Trigger the Quarantine

BPDU Guard was enabled in the same command block as PortFast (see Mistakes table below — the lab's intended path was a separate global default command):

```
Bunker-SW1(config-if-range)#spanning-tree bpduguard enable
```

Confirmed on both interfaces:

```
Bunker-SW1#show spanning-tree interface ethernet 0/3 detail
 The port is in the portfast mode
 Bpdu guard is enabled
 BPDU: sent 6, received 0

Bunker-SW1#show spanning-tree interface ethernet 1/0 detail
 The port is in the portfast mode
 Bpdu guard is enabled
 BPDU: sent 183, received 5
```

Note Et1/0 already shows 5 BPDUs received — Rogue-SW had been sending its normal ~2-second BPDU heartbeat the whole time it was connected, before the guard was armed. Nothing happened yet because BPDU Guard wasn't active for those.

Triggered a fresh BPDU by bouncing Rogue-SW's port:

```
Rogue-SW(config)#interface ethernet0/0
Rogue-SW(config-if)#shutdown
Rogue-SW(config-if)#no shutdown
```

Bunker-SW1 caught it and immediately err-disabled the port — confirmed by watching `show ip interface brief` flip from `up/up` to `down/down` between checks, then the status and log detail:

```
Bunker-SW1#show interfaces status | include err-disabled|Et1/0|Port
Et1/0        Access to Rogue-SW err-disabled 10           full   auto 10/100/1000BaseTX

Bunker-SW1#show logging | include BPDU|BPDUGUARD|err
%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU from bridge aabb.cc00.0300 on port Et1/0 with BPDU Guard enabled. Disabling port.
%PM-4-ERR_DISABLE: bpduguard error detected on Et1/0, putting Et1/0 in err-disable state
```

`show spanning-tree interface Ethernet1/0 detail` returned `no spanning tree info available` — expected, since the port is fully down, not just STP-blocked.

**Recovery**, in the correct order — remove the rogue source first, then clear the error:

```
Rogue-SW(config)#interface ethernet0/0
Rogue-SW(config-if)#shutdown
```

```
Bunker-SW1(config)#interface ethernet1/0
Bunker-SW1(config-if)#shutdown
Bunker-SW1(config-if)#no shutdown
```

Confirmed restored:

```
Bunker-SW1#show interfaces status | include Et1/0|port
Et1/0        Access to Rogue-SW connected    10           full   auto 10/100/1000BaseTX
```

## Completion Check
- [x] Et0/3 showed default learning-state behavior before PortFast, `portfast mode` after
- [x] PortFast active on Et0/3 and Et1/0; trunk uplink Et0/0 left untouched
- [x] Rogue-SW triggered Et1/0 into err-disabled with a BPDU Guard log message
- [x] Et1/0 recovered via `shutdown` / `no shutdown` after Rogue-SW's port was shut down

## Mistakes Made
| What happened | Why it matters | Correct approach |
|---|---|---|
| Task 0 baseline used `show spanning-tree` (full) instead of `show spanning-tree summary`, and no pre-bounce `show spanning-tree interface Ethernet0/3 detail` was captured before bouncing the port | Missed the global summary view (where PortFast/BPDU Guard default status is confirmed at a glance); the pre-bounce detail baseline was skipped, so the "learning" state was only caught by luck on the post-bounce check | Run `show spanning-tree summary` and `show spanning-tree interface Ethernet0/3 detail` *before* touching the interface, so the baseline is deliberate, not incidental |
| PortFast and BPDU Guard were both applied per-interface in a single `interface range` block (`spanning-tree portfast` + `spanning-tree bpduguard enable` on Et0/3 and Et1/0 together) | Functionally protects these two ports, but doesn't scale — any new access port added later won't get BPDU Guard automatically | Use the global `spanning-tree portfast bpduguard default` in config mode once, which auto-applies BPDU Guard to every PortFast-enabled port present and future |
