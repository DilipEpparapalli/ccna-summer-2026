## Simple Explanation
PortFast makes a switch port skip Spanning Tree's 30-second wait (listening + learning) and jump straight to forwarding traffic, so end devices come online instantly instead of sitting on a delay. BPDU Guard is the safety net for that speed — it watches a PortFast port for any sign that a switch (not an end device) got plugged in, and instantly shuts the port down if it does.

## Why It Exists
Spanning Tree's listening/learning delay exists to prevent loops — but a port with a single laptop, phone, or printer on it will never cause a loop, so making it wait 30 seconds before passing traffic is pure friction with zero benefit. PortFast removes that friction. But removing the wait also removes STP's built-in "look before you leap" protection, so BPDU Guard exists to restore that protection without bringing back the delay — it detects the one signal (a BPDU) that only a switch would ever send, and reacts instantly.

## How It Works

**PortFast:**
- Applied to access ports (single end device) — skips listening/learning entirely, port goes straight to forwarding.
- Applied to a trunk port via the separate `spanning-tree portfast trunk` command — trunk ports are excluded from plain PortFast by default because a trunk is assumed to be a switch-to-switch uplink (the exact place loops form). Using `portfast trunk` is an explicit override: the administrator is vouching that only one non-switch device (e.g., a router doing router-on-a-stick, or a single hypervisor host) sits on the other end — not another switch.

**BPDU Guard:**
- Every switch port continuously sends BPDUs (Bridge Protocol Data Units) roughly every 2 seconds — STP's way of discovering other switches and electing the root bridge.
- End devices never send BPDUs back. If a BPDU is ever *received* on a PortFast-enabled port, that's proof a switch (or a loop) just appeared where one shouldn't be.
- BPDU Guard reacts by immediately error-disabling the port — dropping it, not just blocking STP traffic — which kills the loop before it can form.
- Recovery: `shutdown` then `no shutdown` on the interface clears the err-disabled state.

## Key Terms
| Term | Meaning |
|------|---------|
| BPDU | Bridge Protocol Data Unit — STP's discovery/heartbeat message, sent every ~2 seconds by every switch port |
| PortFast | Feature that skips the listening and learning STP states on a port, going straight to forwarding |
| `spanning-tree portfast trunk` | Explicit override enabling PortFast behavior on a trunk port, promising only one non-switch device is downstream |
| BPDU Guard | Feature that error-disables a PortFast port the instant it receives a BPDU |
| err-disabled | A port state where the switch has forcibly shut the port down due to a detected violation (BPDU Guard, security violation, etc.) |
| Listening / Learning | The two ~15-second STP states a normal port passes through before forwarding — PortFast skips both |

## Real-World Connection
A common enterprise incident: a well-meaning employee plugs a cheap unmanaged switch into a wall jack to get an extra port under their desk, then later a stray cable gets plugged back into that same switch's other port — an instant loop, since unmanaged switches don't run STP at all. On an access port with PortFast + BPDU Guard enabled, that loop never gets the chance to form — the moment the unmanaged switch (or the loop itself) causes a BPDU to appear where it shouldn't, the port drops.

The scalable deployment pattern: enable `spanning-tree portfast bpdu-guard default` globally (applies BPDU Guard to every PortFast port automatically), then explicitly disable it only on the handful of ports that legitimately connect to other switches with `spanning-tree bpdu-guard disable`.

For router-on-a-stick or hypervisor trunk uplinks specifically: `spanning-tree portfast trunk` + `spanning-tree bpdu-guard enable` on that single port gives fast convergence without leaving the door open if someone later swaps the router for a real switch.

## Exam Traps
- Plain `spanning-tree portfast` does **not** reliably apply to a trunk port — the separate `spanning-tree portfast trunk` command is required, and it's easy to blank on this distinction under exam pressure.
- Trunk ports are excluded from PortFast by default not because of VLAN scoping, but because a trunk is assumed to be a switch-to-switch uplink — the highest-risk place for a loop to form.
- BPDU Guard error-disables the port entirely (hard down), not just blocks STP — don't confuse this with BPDU Filter, which just suppresses BPDUs rather than shutting the port down.
- Recovery from err-disabled requires manual `shutdown` / `no shutdown` unless errdisable auto-recovery is configured — the port does not come back on its own by default.

## Commands
```
! Per-interface PortFast (access port)
interface GigabitEthernet0/1
 spanning-tree portfast

! Per-interface PortFast override for a trunk (e.g., router-on-a-stick uplink)
interface GigabitEthernet0/2
 switchport mode trunk
 spanning-tree portfast trunk

! Global PortFast for all access ports
spanning-tree portfast default

! Global BPDU Guard applied to all PortFast ports
spanning-tree portfast bpdu-guard default

! Disable BPDU Guard on a specific port (e.g., a legitimate switch uplink)
interface GigabitEthernet0/3
 spanning-tree bpdu-guard disable

! Per-interface BPDU Guard (explicit, without relying on the global default)
interface GigabitEthernet0/2
 spanning-tree bpdu-guard enable

! Recovering an err-disabled port
interface GigabitEthernet0/1
 shutdown
 no shutdown

! Verification
show interfaces status
show spanning-tree interface GigabitEthernet0/1 detail
```

## Recall Questions
1. Why is a trunk port excluded from plain PortFast by default, and what specific promise is an administrator making when they override that with `spanning-tree portfast trunk`?
2. A port with PortFast and BPDU Guard enabled receives a BPDU. What exactly happens to the port, and what two commands bring it back?
3. What's the difference in risk profile between an access port with PortFast enabled and a trunk port with `portfast trunk` enabled — and why does BPDU Guard matter more on the latter?
