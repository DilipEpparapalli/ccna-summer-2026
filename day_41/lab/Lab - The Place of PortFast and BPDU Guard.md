## Mission Briefing

Jeremy wheels a cart of contraband plastic switches into the Castle Rysen bunker to prove how quickly a cheap rogue switch can threaten an access edge. Your drill is to inspect the default Rapid PVST behavior, enable PortFast on real access ports, and then arm BPDU Guard so the rogue switch is quarantined before it can start a loop.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Inspect Bunker-SW1's access-port spanning-tree behavior before PortFast.
- Configure PortFast on the shelter's endpoint and rogue-switch access ports while leaving the uplink untouched.
- Enable BPDU Guard, trigger an errdisable event on the rogue-facing port, and safely restore service after removing the rogue BPDU source.

## Live Interface Map

- Bunker-SW1 `Ethernet0/0` trunks to Shelter-Core.
- Bunker-SW1 `Ethernet0/3` connects to Bunker-Host.
- Bunker-SW1 `Ethernet1/0` connects to Rogue-SW `Ethernet0/0`.

---

## Task 0 - Inspect the Default Access-Port Behavior

Document the default Rapid PVST behavior an endpoint experiences on Bunker-SW1 before PortFast is enabled.

**Steps:**

1. From Bunker-SW1 in privileged EXEC mode, inspect the spanning-tree summary and the detail output for `Ethernet0/3`.
2. Bounce `Ethernet0/3` to mimic a user reconnecting a device.
3. Immediately check the interface detail again. In the live Rapid PVST lab, you may catch the port in learning with a short forward-delay timer, or it may already be forwarding by the time the command runs.

---

## Task 1 - Launch PortFast

Accelerate the Castle Rysen access edge so endpoint ports do not wait on normal spanning-tree convergence.

**Steps:**

1. On Bunker-SW1, enable PortFast on `Ethernet0/3` for Bunker-Host and `Ethernet1/0` for the rogue-switch test port. Leave the trunk uplink `Ethernet0/0` untouched.
2. Bounce `Ethernet0/3` again to confirm it returns to forwarding quickly.
3. Capture the updated spanning-tree detail to prove the interface reports PortFast mode.

---

## Task 2 - Arm BPDU Guard

Protect the shelter network from rogue switches and rehearse the recovery steps after a violation.

**Steps:**

1. From Bunker-SW1 global configuration, activate BPDU Guard on all PortFast-enabled interfaces.
2. On Rogue-SW, bounce `Ethernet0/0` so it sends a fresh BPDU toward Bunker-SW1.
3. On Bunker-SW1, verify `Ethernet1/0` enters `err-disabled` and confirm the BPDU Guard log message.
4. Recover safely by shutting down Rogue-SW `Ethernet0/0`, then shutting and re-enabling Bunker-SW1 `Ethernet1/0`.

```
! Bunker-SW1
enable
configure terminal
spanning-tree portfast bpduguard default
end

! Rogue-SW - force a fresh BPDU toward Bunker-SW1
enable
configure terminal
interface Ethernet0/0
 shutdown
 no shutdown
end

! Bunker-SW1 - verify the protection hit
show interfaces status | include err-disabled|Et1/0|Port
show spanning-tree interface Ethernet1/0 detail
show logging | include BPDU|BPDUGUARD|err

! Remove the rogue BPDU source
! Rogue-SW
configure terminal
interface Ethernet0/0
 shutdown
end

! Bunker-SW1 - recover the protected port
configure terminal
interface Ethernet1/0
 shutdown
 no shutdown
exit
end
show interfaces status | include Et1/0|Port
```

Sample protection output:

```
Et1/0        Access to Rogue-SW err-disabled 10           full   auto 10/100/100BaseTX

%SPANTREE-2-BLOCK_BPDUGUARD: Received BPDU from bridge aabb.cc00.ab00 on port Ethernet1/0 with BPDU Guard enabled. Disabling port.
%PM-4-ERR_DISABLE: bpduguard error detected on Et1/0, putting Et1/0 in err-disable state
```

---

## Completion Check

- `Ethernet0/3` shows default Rapid PVST behavior before PortFast and reports `The port is in the portfast mode` afterward.
- PortFast is active on Bunker-SW1 `Ethernet0/3` and `Ethernet1/0`, while trunk uplink `Ethernet0/0` remains a normal trunk.
- Triggering Rogue-SW causes Bunker-SW1 `Ethernet1/0` to enter `err-disabled` with a BPDU Guard log message.
- After Rogue-SW `Ethernet0/0` is shut down, Bunker-SW1 `Ethernet1/0` recovers with `shutdown` / `no shutdown`.

 copy on select