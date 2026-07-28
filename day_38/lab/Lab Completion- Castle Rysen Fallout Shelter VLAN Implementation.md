
Second run of this lab for practice. Same objective as attempt 1 — VLAN database on the core, trunk it out to three access switches and the router, place access ports, stand up router-on-a-stick with DHCP. This run has two real mistakes worth understanding, one of which is **still unresolved in the final config** — flagged clearly below.

## Topology

```
                         Fallout-RT1
                              │ Et0/0 (physical, no IP)
                              │ ├─ Et0/0.10  10.0.16.1/25   VLAN 10 MGMT
                              │ ├─ Et0/0.20  10.0.16.129/25 VLAN 20 INTERNAL
                              │ ├─ Et0/0.30  10.0.17.1/25   VLAN 30 VIDEO
                              │ └─ Et0/0.40  10.0.17.129/25 VLAN 40 GUEST
                              │
                         Fallout-SW1  (VTP Server, domain: fallout)
                   ┌──────────┼──────────┐
              Et0/2│          │Et0/3     │Et1/0
                   │          │          │
              Fallout-SW3  Fallout-SW4  Fallout-SW5
               Et0/3          Et0/3     Et0/3   Et1/1
                 │              │          │       │
              VLAN 10        VLAN 20   VLAN 30  VLAN 40
```

---

## Task 0 — VLAN Database on Fallout-SW1

**Before state:**
```
Fallout-SW1#show vtp status
VTP Domain Name                 :
VTP Operating Mode              : Server
Number of existing VLANs        : 5
Configuration Revision          : 0
```

**Config applied:**
```
Fallout-SW1#conf t
Fallout-SW1(config)#vtp domain fallout
Changing VTP domain name from NULL to fallout
Fallout-SW1(config)#vlan 10
Fallout-SW1(config-vlan)#name FALLOUT-MGMT
Fallout-SW1(config-vlan)#vlan 20
Fallout-SW1(config-vlan)#name FALLOUT-INTERNALCOMM
Fallout-SW1(config-vlan)#vlan 30
Fallout-SW1(config-vlan)#name FALLOUT-VIDEOSUR
Fallout-SW1(config-vlan)#vlan 40
Fallout-SW1(config-vlan)#name FALLOUT-GUEST
Fallout-SW1(config-vlan)#end
```

**After state — confirmed:**
```
Fallout-SW1#show vtp status
VTP Domain Name                 : fallout
VTP Operating Mode              : Server
Number of existing VLANs        : 9
Configuration Revision          : 4

Fallout-SW1#show vlan brief
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------
10   FALLOUT-MGMT                     active
20   FALLOUT-INTERNALCOMM             active
30   FALLOUT-VIDEOSUR                 active
40   FALLOUT-GUEST                    active
```

Revision 4 confirms four `vlan`/`name` pairs were counted as four separate database changes — same signature as attempt 1.

---

## Task 1 — Trunking

### ⚠️ Mistake: wrong interface trunked first

The live topology map says **Fallout-SW1 Ethernet0/1** connects to Fallout-RT1. Instead, **Ethernet0/0** was configured as the trunk first:

```
Fallout-SW1(config)#interface ethernet0/0
Fallout-SW1(config-if)#switchport trunk encapsulation dot1q
Fallout-SW1(config-if)#switchport mode trunk
```

Verified — and confirmed the mistake — with:
```
Fallout-SW1#show interface trunk
Port           Mode             Encapsulation  Status        Native vlan
Et0/0          on               802.1q         trunking      1

Port           Vlans allowed on trunk
Et0/0          1-4094
```

Et0/0 came up trunking, but it isn't the link to Fallout-RT1 per the topology doc — it's an unused port in this trimmed lab. This is exactly the kind of error `show interface trunk` is for: the port *looks* successful (state = trunking) even though it's the wrong physical link, so a passing status alone doesn't confirm the topology is correct — you have to cross-check against the actual cabling map.

**Correction applied** — shut the wrong port down, then trunked the right one:
```
Fallout-SW1(config)#interface ethernet0/0
Fallout-SW1(config-if)#shutdown

Fallout-SW1(config)#interface ethernet0/1
Fallout-SW1(config-if)#switchport trunk encapsulation dot1q
Fallout-SW1(config-if)#switchport mode trunk
```

**Residual artifact:** Et0/0 was left administratively shut down with trunk commands still configured on it, rather than being reverted to its default access-port state. Not harmful (a shutdown port forwards nothing), but if this device were handed to someone else, the leftover trunk config on a shut interface could cause confusion later.

### Remaining trunks — SW3, SW4, SW5 links (configured together via interface range)

```
Fallout-SW1(config)#interface range ethernet 0/2-3, ethernet 1/0
Fallout-SW1(config-if-range)#switchport trunk encapsulation dot1q
Fallout-SW1(config-if-range)#switchport mode trunk
```

**Final trunk verification on Fallout-SW1:**
```
Fallout-SW1#show interface trunk
Port           Mode             Encapsulation  Status        Native vlan
Et0/1          on               802.1q         trunking      1
Et0/2          on               802.1q         trunking      1
Et0/3          on               802.1q         trunking      1
Et1/0          on               802.1q         trunking      1

Port           Vlans allowed on trunk
Et0/1          1-4094
Et0/2          1-4094
Et0/3          1-4094
Et1/0          1-4094

Port           Vlans allowed and active in management domain
Et0/1          1,10,20,30,40
Et0/2          1,10,20,30,40
Et0/3          1,10,20,30,40
Et1/0          1,10,20,30,40
```

**Deviation (same as attempt 1):** `switchport trunk allowed vlan 10,20,30,40` was never issued on any trunk. All four still show `1-4094` allowed rather than being pruned to the four active VLANs. VTP's "allowed and active in management domain" line is doing the practical filtering here, not an explicit trunk allow-list. See attempt 1's notes for the exam-relevant distinction between these two filters.

### Access switches confirm VTP sync

All three (SW3, SW4, SW5) independently verified:
```
show vtp status
VTP Domain Name                 : fallout
VTP Operating Mode              : Server
Number of existing VLANs        : 9
Configuration Revision          : 4
```
Matching MD5 digest across SW1, SW3, SW4, SW5 confirms all four switches hold an identical VLAN database — this is VTP working as intended, propagated purely over the trunk links with zero manual VLAN entry on the access switches.

---

## Task 2 — Access Port Assignment

### Fallout-SW3 — Et0/3 → VLAN 10 (MGMT console)
```
Fallout-SW3(config)#interface ethernet0/3
Fallout-SW3(config-if)#description MGMT-CONSOLE
Fallout-SW3(config-if)#switchport mode access
Fallout-SW3(config-if)#switchport access vlan 10
Fallout-SW3(config-if)#spanning-tree portfast
%Warning: portfast should only be enabled on ports connected to a single
 host... Use with CAUTION
```
(That warning is Cisco's standard caution message any time PortFast is enabled — not an error, and not specific to this port being misconfigured.)

### Fallout-SW4 — Et0/3 → VLAN 20 (Internal workstation)
```
Fallout-SW4(config)#interface ethernet0/3
Fallout-SW4(config-if)#description INTERNAL-WORKSTATION
Fallout-SW4(config-if)#switchport mode access
Fallout-SW4(config-if)#switchport access vlan 20
Fallout-SW4(config-if)#spanning-tree portfast
```

### Fallout-SW5 — Et0/3 → VLAN 30 (Video NVR)

**⚠️ Mistake: wrong VLAN assigned first, then corrected:**
```
Fallout-SW5(config-if)#description VIDEO-NVR
Fallout-SW5(config-if)#switchport mode access
Fallout-SW5(config-if)#switchport access vlan 40    ← wrong (should be 30)
Fallout-SW5(config-if)#switchport access vlan 30    ← corrected
```
Since `switchport access vlan` is not additive — each command simply overwrites the port's VLAN assignment — the second command silently fixed the first with no cleanup needed. Confirmed correct outcome:
```
Fallout-SW5(config)#do show vlan brief
30   FALLOUT-VIDEOSUR    active    Et0/3
```

### Fallout-SW5 — Et1/1 → VLAN 40 (Guest kiosk)
```
Fallout-SW5(config)#interface ethernet1/1
Fallout-SW5(config-if)#description GUEST-KIOSK
Fallout-SW5(config-if)#switchport mode access
Fallout-SW5(config-if)#switchport access vlan 40
Fallout-SW5(config-if)#spanning-tree portfast
```

**Final verification, Fallout-SW5:**
```
Fallout-SW5#show vlan brief
30   FALLOUT-VIDEOSUR    active    Et0/3
40   FALLOUT-GUEST       active    Et1/1
```
Both access ports landed correctly — the earlier VLAN-40 mistake on Et0/3 left no trace once corrected.

---

## Task 3 — Fallout-RT1 Gateway + DHCP

### Physical + subinterfaces
```
Fallout-RT1(config)#interface ethernet0/0
Fallout-RT1(config-if)#no shutdown

Fallout-RT1(config)#interface ethernet0/0.10
Fallout-RT1(config-subif)#encapsulation dot1Q 10
Fallout-RT1(config-subif)#ip address 10.0.16.1 255.255.255.128

Fallout-RT1(config)#interface ethernet0/0.20
Fallout-RT1(config-subif)#encapsulation dot1Q 20
Fallout-RT1(config-subif)#ip address 10.0.16.129 255.255.255.128

Fallout-RT1(config)#interface ethernet0/0.30
Fallout-RT1(config-subif)#encapsulation dot1Q 30
Fallout-RT1(config-subif)#ip address 10.0.17.1 255.255.255.128

Fallout-RT1(config)#interface ethernet0/0.40
Fallout-RT1(config-subif)#encapsulation dot1Q 40
Fallout-RT1(config-subif)#ip address 10.0.17.129 255.255.255.128
```

Verified:
```
Fallout-RT1#show ip interface brief
Ethernet0/0            unassigned      YES unset  up   up
Ethernet0/0.10         10.0.16.1       YES manual up   up
Ethernet0/0.20         10.0.16.129     YES manual up   up
Ethernet0/0.30         10.0.17.1       YES manual up   up
Ethernet0/0.40         10.0.17.129     YES manual up   up
```
All four subinterfaces up/up, physical parent interface with no IP — router-on-a-stick standing correctly.

### DHCP pools

**MGMT — correct:**
```
Fallout-RT1(config)#ip dhcp pool MGMT
Fallout-RT1(dhcp-config)#network 10.0.16.0 255.255.255.128
Fallout-RT1(dhcp-config)#default-router 10.0.16.1
Fallout-RT1(dhcp-config)#dns-server 1.1.1.1
Fallout-RT1(dhcp-config)#domain-name fallout.local
```

**INTERNAL — ⚠️ unresolved mistake:**
```
Fallout-RT1(config)#ip dhcp pool INTERNAL
Fallout-RT1(dhcp-config)#network 10.0.16.128 255.255.255.128
Fallout-RT1(dhcp-config)#dns-server 1.1.1.1
Fallout-RT1(dhcp-config)#default-router 10.0.16.1
Fallout-RT1(dhcp-config)#domain-name fallout.local
```
`default-router` was set to `10.0.16.1` — that's the **MGMT** pool's gateway (VLAN 10), not this pool's own. INTERNAL's subnet is `10.0.16.128/25`, served by subinterface `Et0/0.20` at `10.0.16.129`. Confirmed still wrong in the final saved config:
```
Fallout-RT1#show running-config | section ip dhcp pool
ip dhcp pool INTERNAL
 network 10.0.16.128 255.255.255.128
 dns-server 1.1.1.1
 default-router 10.0.16.1
 domain-name fallout.local
```
**Impact:** any client that pulled a lease from this pool would be handed `10.0.16.1` as its default gateway — an address that lives on a different subnet entirely (`10.0.16.0/25`, not the client's own `10.0.16.128/25`). The client would have an IP/mask/gateway combination where the gateway is unreachable on its local segment, breaking off-subnet connectivity even though the DHCP lease itself "succeeds."

**Fix:** `default-router 10.0.16.129` (matching `Et0/0.20`'s IP).

**VIDEO — mistake caught and corrected in the same breath:**
```
Fallout-RT1(config)#ip dhcp pool VIDEO
Fallout-RT1(dhcp-config)#network 10.0.17.0 255.255.255.128
Fallout-RT1(dhcp-config)#dns-server 1.1.1.1
Fallout-RT1(dhcp-config)#default-router 10.0.17.129    ← wrong (that's Guest's gateway)
Fallout-RT1(dhcp-config)#domain-name fallout.local
Fallout-RT1(dhcp-config)#default-router 10.0.17.1       ← corrected
```
Same pattern as the SW5 VLAN mistake — a single-value command (`default-router` only holds one address per pool) means the second entry simply overwrote the first. Final running-config confirms it landed correctly:
```
ip dhcp pool VIDEO
 network 10.0.17.0 255.255.255.128
 dns-server 1.1.1.1
 default-router 10.0.17.1
 domain-name fallout.local
```

**GUEST — correct:**
```
Fallout-RT1(config)#ip dhcp pool GUEST
Fallout-RT1(dhcp-config)#network 10.0.17.128 255.255.255.128
Fallout-RT1(dhcp-config)#default-router 10.0.17.129
Fallout-RT1(dhcp-config)#dns-server 1.1.1.1
Fallout-RT1(dhcp-config)#domain-name fallout.local
```

**Binding table** — empty, as expected (no workstation client tabs in this trimmed lab):
```
Fallout-RT1#show ip dhcp binding
Bindings from all pools not associated with VRF:
(empty)
```

---

## Mistakes Made

| # | Device | What happened | Caught? | Status |
|---|--------|----------------|---------|--------|
| 1 | Fallout-SW1 | Trunked Ethernet0/0 (wrong port) instead of Ethernet0/1 for the Fallout-RT1 link | Yes — verified against topology map, shut the port down | Fixed, but Et0/0 left shutdown with trunk config still attached rather than reverted to default |
| 2 | Fallout-SW5 | Assigned Et0/3 to VLAN 40 instead of VLAN 30 (Video NVR port) | Yes — immediately re-issued the command | Fully corrected, no trace in final `show vlan` |
| 3 | Fallout-RT1 | DHCP pool VIDEO: `default-router` set to `10.0.17.129` (Guest's gateway) instead of `10.0.17.1` | Yes — re-entered the command right after | Fully corrected, confirmed in running-config |
| 4 | **Fallout-RT1** | **DHCP pool INTERNAL: `default-router` set to `10.0.16.1` (MGMT's gateway) instead of `10.0.16.129`** | **No** | **Still wrong in final running-config — needs to be fixed** |

## Deviations (carried over from attempt 1)

- `switchport trunk allowed vlan 10,20,30,40` never applied on any Fallout-SW1 trunk — all show `1-4094` allowed, relying on VTP's active-VLAN filtering instead of an explicit trunk allow-list.
- PortFast applied on SW3/SW4/SW5 access ports this run (unlike attempt 1) — good, matches the lab guide.
- VLAN names again used shorthand (`FALLOUT-MGMT`, etc.) rather than the guide's exact strings — cosmetic only.

## Key Reinforcement

- A `show interface trunk` reporting "trunking" on a port confirms the port's own state — it says nothing about whether that's the *correct* physical port per your topology. Cross-checking the interface map before configuring, not just after, would have caught the Et0/0 mistake before it happened.
- Single-value DHCP options like `default-router` and `switchport access vlan` are **overwrite, not append** — re-issuing the command with a corrected value is a valid and clean fix, no `no` command needed first.
- The INTERNAL pool mistake is the practical lesson here: a DHCP misconfiguration can look completely healthy in the config (syntax valid, pool active) while still being operationally broken, because the error is in the *values*, not the *structure*. Always double check `default-router` against the actual subinterface IP for that VLAN, not just that the line exists.
