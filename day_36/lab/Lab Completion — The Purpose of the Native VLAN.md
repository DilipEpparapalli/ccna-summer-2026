## Lab Environment
- **Platform:** Cisco Modeling Labs (CML) — IOSv Layer 2 switches
- **Devices:** Cafe-SW1, Cafe-SW2
- **Topology:**

```
Cafe-SW1 ── Et0/1 ── Cafe-SW2
  Et0/2 → VLAN 10 access      Et0/2 → VLAN 10 access
  Et0/3 → VLAN 20 access      Et0/3 → VLAN 20 access
```
![[ccna-summer-2026/day_36/lab/Network_Diagram_1.png]]
## Objective
Observe the default native VLAN behavior on a trunk link. Create VLAN 99 as a
dedicated management VLAN. Set VLAN 99 as the native VLAN on Cafe-SW1 first,
observe the mismatch errors that fire when only one side is changed, then align
Cafe-SW2 to resolve the mismatch and confirm STP unblocks the port.

---

## Task 0 — Baseline: Both Switches Before Any Changes

**Cafe-SW1 baseline `show vlan`:**

```
VLAN Name        Status    Ports
1    default      active    Et0/0, Et0/1
10   VLAN0010     active    Et0/2
20   VLAN0020     active    Et0/3
```

**Cafe-SW2 baseline `show vlan`:**

```
VLAN Name        Status    Ports
1    default      active    Et0/0, Et0/1
10   VLAN0010     active    Et0/2
20   VLAN0020     active    Et0/3
```

**`show interfaces trunk` on both switches returned blank** — no trunks configured.
Et0/1 was physically connected but operating as a static access port in VLAN 1.

**Confirmed Et0/1 default state on Cafe-SW1:**

```
Cafe-SW1# show interfaces Ethernet0/1 switchport | include Administrative Mode|Operational Mode|Access Mode

Administrative Mode: dynamic auto
Operational Mode:    static access
Access Mode VLAN:    1 (default)
Trunking VLANs Enabled: ALL
```

**What this confirmed:**
- Both switches had VLANs 10 and 20 pre-configured with access ports
- Et0/1 defaulted to `dynamic auto` — waiting for the other side to initiate
- Since both sides were `auto`, neither initiated, so no trunk formed (auto + auto
  = static access, no trunk)
- Native VLAN 1 was the default — all untagged traffic on any trunk would land here

Also checked `show dtp interface` at baseline — Et0/0 showed `TAS: AUTO` (still
negotiating), while Et0/1, Et0/2, Et0/3 showed `TAS: OFF / FSM state: S1:OFF`
because they were explicitly set to access mode earlier in the session.

---

## Task 1 — Baseline Port Mode Investigation

Before configuring the trunk, explored the DTP state more carefully. Temporarily
set Et0/1 to explicit access mode to observe the DTP output, then set it back to
`dynamic auto` to restore the true baseline:

```
Cafe-SW1(config)# interface ethernet0/1
Cafe-SW1(config-if)# switchport mode access   ← sets explicitly to access
Cafe-SW1(config-if)# exit

! Check DTP output → FSM state: S1:OFF, Enabled: no (DTP disabled on access port)

Cafe-SW1(config)# interface ethernet0/1
Cafe-SW1(config-if)# switchport mode dynamic auto   ← restore baseline
```

Verified baseline with `show interfaces Ethernet0/1 switchport`:

```
Administrative Mode: dynamic auto
Operational Mode:    static access
Access Mode VLAN:    1 (default)
```

Dynamic auto confirmed. No trunk. Ready to proceed.

---

## Task 2 — Create VLAN 99 and Configure Native VLAN Trunk on Cafe-SW1

```
Cafe-SW1(config)# vlan 99
Cafe-SW1(config-vlan)# name MGMT-NATIVE
Cafe-SW1(config-vlan)# exit

Cafe-SW1(config)# interface ethernet0/1
Cafe-SW1(config-if)# switchport trunk encapsulation dot1q
Cafe-SW1(config-if)# switchport mode trunk
```

Interface flapped — expected:

```
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface Ethernet0/1, changed state to up
```

```
Cafe-SW1(config-if)# switchport trunk native vlan 99
Cafe-SW1(config-if)# end
```

**Immediately after setting native VLAN 99 — STP blocked the port:**

```
%SPANTREE-2-BLOCK_PVID_PEER:  Blocking Ethernet0/1 on VLAN0001. Inconsistent peer vlan.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking Ethernet0/1 on VLAN0099. Inconsistent local vlan.
```

**What this means:**
- SW1 now expects untagged traffic → VLAN 99
- SW2 still expects untagged traffic → VLAN 1
- STP detected the disagreement and blocked Et0/1 for both VLAN 1 and VLAN 99 to
  prevent traffic leaking between VLANs that were never supposed to be bridged

**Verified trunk state on Cafe-SW1:**

```
Cafe-SW1# show interfaces trunk

Port   Mode  Encapsulation  Status    Native vlan
Et0/1  on    802.1q         trunking  99          ← native changed

Port   Vlans allowed and active in management domain
Et0/1  1,10,20,99

Port   Vlans in spanning tree forwarding state and not pruned
Et0/1  10,20                                      ← VLAN 1 and 99 blocked by STP
```

VLAN 10 and 20 were still forwarding because they're tagged — the mismatch only
affects untagged (native) traffic. VLAN 1 and VLAN 99 were blocked by STP.

**CDP mismatch messages started firing every ~49 seconds:**

```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on Ethernet0/1 (99),
with Cafe-SW2 Ethernet0/1 (1).
```

CDP is how the switch *knows* there's a mismatch — it reads the native VLAN from
the neighbor's CDP advertisement and compares it to its own setting.

---

## Task 3 — Fix the Mismatch on Cafe-SW2

On Cafe-SW2, the mismatch messages were already visible in the console (CDP had
detected the problem from SW1's advertisements):

```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on Ethernet0/1 (1),
with Cafe-SW1 Ethernet0/1 (99).
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking Ethernet0/1 on VLAN0001. Inconsistent local vlan.
```

**Error — Missing space in VLAN command:**

```
Cafe-SW2(config)# vlan99
% Invalid input detected at '^' marker.
```

`vlan99` is not recognized — must be `vlan 99` with a space. Minor typo, caught
immediately.

**Applied matching config on Cafe-SW2:**

```
Cafe-SW2(config)# vlan 99
Cafe-SW2(config-vlan)# name MGMT-NATIVE
Cafe-SW2(config-vlan)# exit

Cafe-SW2(config)# interface ethernet0/1
Cafe-SW2(config-if)# switchport trunk encapsulation dot1q
Cafe-SW2(config-if)# switchport mode trunk
Cafe-SW2(config-if)# switchport trunk native vlan 99
Cafe-SW2(config-if)# exit
Cafe-SW2(config)# end
```

**Immediately after — STP unblocked the port on both switches:**

```
! On Cafe-SW2:
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0099. Port consistency restored.
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0001. Port consistency restored.

! On Cafe-SW1 (at same timestamp):
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0001. Port consistency restored.
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0099. Port consistency restored.
```

Both switches fired `UNBLOCK_CONSIST_PORT` at the same moment — 20:29:06 — the
instant the native VLANs aligned. STP confirmed consistency and restored forwarding
on both VLANs simultaneously.

**Final trunk state on Cafe-SW2:**

```
Cafe-SW2# show interfaces trunk

Port   Mode  Encapsulation  Status    Native vlan
Et0/1  on    802.1q         trunking  99          ← matches SW1

Port   Vlans allowed and active in management domain
Et0/1  1,10,20,99

Port   Vlans in spanning tree forwarding state and not pruned
Et0/1  1,10,20,99                                 ← all VLANs forwarding
```

**Final `show vlan` on Cafe-SW2:**

```
VLAN Name          Status    Ports
1    default        active    Et0/0
10   VLAN0010       active    Et0/2
20   VLAN0020       active    Et0/3
99   MGMT-NATIVE    active              ← created, no access ports assigned
```

No new CDP mismatch messages appeared after alignment. Old messages remained in
`show logging` (log buffer doesn't auto-clear) but no new ones fired.

---

## Timeline — What Happened and When

```
20:26:22  SW1 trunk mode applied → Et0/1 flaps down/up
20:26:35  SW1 native VLAN set to 99 →
          STP immediately blocks Et0/1 VLAN 1 and VLAN 99 (mismatch)
20:27:21  First CDP mismatch message fires on SW1
20:28:14  Second CDP mismatch fires (SW2 still mismatched)
20:28:25  SW2 creates VLAN 99, STP blocks VLAN 99 on SW2 side too
20:29:06  SW2 native VLAN set to 99 →
          STP unblocks BOTH switches simultaneously — consistency restored
20:29:06  No more CDP mismatch messages
```

---

## Mistakes Made

| Mistake | What Happened | Correction |
|---------|---------------|------------|
| `vlan99` (no space) on Cafe-SW2 | Invalid input detected | `vlan 99` with a space |
| `switchport mode tre` typo on Cafe-SW2 | Invalid input, re-typed | `switchport mode trunk` |

---

## State After Lab

```
Cafe-SW1:
  Et0/1  → trunk, 802.1q, native VLAN 99
  Et0/2  → access, VLAN 10
  Et0/3  → access, VLAN 20
  VLAN 99 (MGMT-NATIVE) → active, no access ports

Cafe-SW2:
  Et0/1  → trunk, 802.1q, native VLAN 99
  Et0/2  → access, VLAN 10
  Et0/3  → access, VLAN 20
  VLAN 99 (MGMT-NATIVE) → active, no access ports

Both switches: native VLAN aligned → no mismatch, all VLANs forwarding
```

---

## Key Commands Used

```
! Create management VLAN
vlan 99
 name MGMT-NATIVE

! Configure trunk with custom native VLAN (IOSv — encapsulation required first)
interface ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 99

! Verify native VLAN and trunk state
show interfaces trunk
  → check "Native vlan" column — must match on both sides

! Verify VLAN exists on switch
show vlan

! Check DTP negotiation state on a port
show dtp interface ethernet0/1
  → TAS: AUTO = still negotiating
  → TAS: OFF  = DTP disabled (access mode or nonegotiate applied)
  → TAS: NONEGOTIATE = nonegotiate explicitly set

! What you see in the log during a mismatch
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on Ethernet0/1 (99),
with Cafe-SW2 Ethernet0/1 (1).

%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking Ethernet0/1 on VLAN0099. Inconsistent local vlan.
%SPANTREE-2-BLOCK_PVID_PEER:  Blocking Ethernet0/1 on VLAN0001. Inconsistent peer vlan.

! What you see when the mismatch is fixed
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0099. Port consistency restored.
%SPANTREE-2-UNBLOCK_CONSIST_PORT: Unblocking Ethernet0/1 on VLAN0001. Port consistency restored.
```
