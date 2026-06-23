
## Objective
Create VLAN 10 (ADMIN_DEVICES) and VLAN 20 (PATRON_DEVICES) on both switches.
Assign patron-facing access ports on CafeSwitch01 to VLAN 20. Confirm port
membership via `show vlan brief`.

---
![[ccna-summer-2026/day_34/lab/Network_Diagram.png]]
## Task 0 — Baseline: Inspect Default VLAN State (CafeSwitch01)

Before making any changes, captured the default VLAN footprint.

```
CafeSwitch01> en
CafeSwitch01# show vlan brief

VLAN Name                             Status    Ports
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                Et1/0, Et1/1, Et1/2, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup
```

**What this confirmed:**
- All 16 ports are in VLAN 1 by default — one broadcast domain, no separation
- VLANs 1002–1005 are legacy artifacts (FDDI, Token Ring) — always present, cannot
  be deleted, irrelevant to modern switching
- No custom VLANs exist yet

---

## Task 1 — Create VLANs on CafeSwitch01

```
CafeSwitch01# conf t
CafeSwitch01(config)# vlan 10
CafeSwitch01(config-vlan)# name ADMIN_DEVICES
CafeSwitch01(config-vlan)# exit
CafeSwitch01(config)# vlan 20
CafeSwitch01(config-vlan)# name PATRON_DEVICES
CafeSwitch01(config-vlan)# exit
CafeSwitch01(config)# end
```

**Verified with `show vlan brief`:**

```
VLAN Name                             Status    Ports
1    default                          active    Et0/0 ... Et3/3 (all 16)
10   ADMIN_DEVICES                    active
20   PATRON_DEVICES                   active
```

**What this confirmed:**
- VLAN 10 and VLAN 20 exist and are active
- Neither VLAN has any ports yet — empty shells at this point
- All ports still live in VLAN 1; no separation has occurred yet

**Note — SVI behavior observed:** Created `interface vlan 10` and `interface vlan 20`
and added descriptions (ADMIN-DEV, PATRON-DEV) while exploring IOS options. Both SVIs
showed `admin down / down` in `show interface description` — expected, because no IP
address was assigned and the VLANs had no active ports yet. SVIs only come up when
the VLAN has at least one active access port assigned to it.

```
Vl10    admin down    down    ADMIN-DEV
Vl20    admin down    down    PATRON-DEV
```

---

## Task 2 — Mirror VLANs on CafeSwitch02

```
CafeSwitch02> en
CafeSwitch02# show vlan brief   ← confirmed default state (all ports in VLAN 1)

CafeSwitch02# conf t
CafeSwitch02(config)# vlan 10
CafeSwitch02(config-vlan)# name ADMIN_DEVICES
CafeSwitch02(config-vlan)# exot   ← typo: "exot" not recognized
                            ^
% Invalid input detected at '^' marker.

CafeSwitch02(config-vlan)# exit   ← corrected
CafeSwitch02(config)# vlan 20
CafeSwitch02(config-vlan)# name PATRON_DEVICES
CafeSwitch02(config-vlan)# exit
CafeSwitch02(config)# end
```

**Verified with `show vlan brief`:**

```
VLAN Name                             Status    Ports
1    default                          active    Et0/0 ... Et3/3 (all 16)
10   ADMIN_DEVICES                    active
20   PATRON_DEVICES                   active
```

**What this confirmed:**
- VLAN definitions now match CafeSwitch01 — same IDs, same names
- No port assignments on SW02 yet; that happens after trunk configuration
- Typo `exot` instead of `exit` was caught immediately by IOS — no config damage,
  just stayed in VLAN config mode until corrected

---

## Task 3 — Assign Patron Ports to VLAN 20 on CafeSwitch01

Used `interface range` to configure Et2/2, Et2/3, and Et3/0–3 as a group.

```
CafeSwitch01# conf t
CafeSwitch01(config)# interface range Ethernet 2/2 - 3, Ethernet 3/0 - 3
CafeSwitch01(config-if-range)# switchport mode access
CafeSwitch01(config-if-range)# switchport access vlan 20
CafeSwitch01(config-if-range)# end
```

**Verified with `show vlan brief`:**

```
VLAN Name                             Status    Ports
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                Et1/0, Et1/1, Et1/2, Et1/3
                                                Et2/0, Et2/1
10   ADMIN_DEVICES                    active
20   PATRON_DEVICES                   active    Et2/2, Et2/3, Et3/0, Et3/1
                                                Et3/2, Et3/3
```

**What this confirmed:**
- Et2/2, Et2/3, Et3/0–3 moved out of VLAN 1 and into VLAN 20
- Any device plugged into those ports is now in the PATRON_DEVICES broadcast domain,
  isolated from VLAN 1 and VLAN 10
- `interface range` with a comma-separated group worked correctly across
  non-contiguous port ranges in a single command

---

## State After Lab

```
CafeSwitch01:
  VLAN 1  (default)      → Et0/0–Et0/3, Et1/0–Et1/3, Et2/0–Et2/1
  VLAN 10 (ADMIN)        → no ports assigned
  VLAN 20 (PATRON)       → Et2/2, Et2/3, Et3/0–Et3/3

CafeSwitch02:
  VLAN 1  (default)      → Et0/0–Et3/3 (all 16 ports)
  VLAN 10 (ADMIN)        → no ports assigned
  VLAN 20 (PATRON)       → no ports assigned
```

VLAN 10 has no ports on either switch — intentional. Trunk links between the switches
must be configured before assigning production ports, otherwise devices lose
connectivity the moment they're moved out of VLAN 1.

---

## Key Commands Used

```
! Verify baseline
show vlan brief

! Create and name VLANs
vlan 10
 name ADMIN_DEVICES
vlan 20
 name PATRON_DEVICES

! Assign access ports across non-contiguous range
interface range Ethernet 2/2 - 3, Ethernet 3/0 - 3
 switchport mode access
 switchport access vlan 20

! Verify port assignment
show vlan brief

! Check SVI status
show interface description
```
