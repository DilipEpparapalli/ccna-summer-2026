## Lab Overview
**Scenario:** Castle Rysen cafe floor migrated from dynamic pool NAT to NAT Overload (PAT). The existing pool mapping was removed and replaced with a single overload rule pointing at the WAN interface, collapsing all inside private addresses behind one public IP using port-based session tracking.

**Environment:** Cisco Modeling Labs — IOS-XE 17.16  
**Device:** Cafe-Rtr (IOSv router)  
**Inside interface:** Ethernet0/0 — 192.168.1.1/24  
**Outside interface:** Ethernet0/1 — 216.0.5.2/30 (WAN toward ISP-Rtr)

---

## Topology

```
PC1 (192.168.1.50) ─┐
                     ├─ Eth0/0 ─ Cafe-Rtr ─ Eth0/1 ─ 216.0.5.1 (ISP) ─ 1.1.1.1
PC2 (192.168.1.51) ─┘
                          PAT: both hosts share 216.0.5.2, differentiated by port
```
![[ccna-summer-2026/day_29/lab/Network_Diagram.png]]
---

## Starting State

Dynamic NAT pool rule was still active from the previous lab:

```
ip nat pool Cafe-Public 216.0.5.20 216.0.5.30 netmask 255.255.255.240
ip nat inside source list 1 pool Cafe-Public
```

NAT table showed active translations:
```
Pro  Inside global       Inside local       Outside local   Outside global
icmp 216.0.5.20:672     192.168.1.50:672   1.1.1.1:672     1.1.1.1:672
---  216.0.5.20         192.168.1.50       ---             ---
icmp 216.0.5.21:671     192.168.1.51:671   1.1.1.1:671     1.1.1.1:671
---  216.0.5.21         192.168.1.51       ---             ---
```

---

## Task 0 — Disable Dynamic NAT Pool Mapping

**First attempt — tried to remove the pool definition directly:**
```
Cafe-Rtr(config)# no ip nat pool Cafe-Public 216.0.5.20 216.0.5.30 netmask 255.255.255.240
%Pool Cafe-Public in use, cannot destroy
```

**Second attempt — cleared translations, tried again:**
```
Cafe-Rtr# clear ip nat translation *
Cafe-Rtr(config)# no ip nat pool Cafe-Public ...
%Pool Cafe-Public in use, cannot destroy
```

Still blocked — traffic was rebuilding translations faster than they were being cleared because the interfaces were still up.

**Third attempt — shut both interfaces, cleared translations, still blocked:**
```
%Pool mapping is using pool, cannot destroy
```

Root cause identified: the pool cannot be removed while the `ip nat inside source` rule still references it. The rule must be removed first, then the pool becomes orphaned and deletable.

**What finally worked — correct sequence:**
```
# 1. Shut both interfaces to stop traffic rebuilding translations
interface ethernet0/0
 shutdown
interface ethernet0/1
 shutdown

# 2. Clear translation table
clear ip nat translation *

# 3. Remove the RULE that references the pool (not the pool itself)
no ip nat inside source list 1 pool Cafe-Public

# 4. Now the pool can be removed if needed (pool definition is orphaned)
# 5. Bring interfaces back up
interface ethernet0/0
 no shutdown
interface ethernet0/1
 no shutdown
```

**Verified:** `show ip nat translations` returned empty.

---

## Task 1 — Configure NAT Overload

ACL 1 was already in place from the previous lab — reused as-is:
```
ip access-list standard 1
 10 permit 192.168.1.0 0.0.0.255
```

Applied overload rule pointing at the WAN interface:
```
Cafe-Rtr(config)# ip nat inside source list 1 interface ethernet0/1 overload
```

Interface NAT roles confirmed unchanged:
- `Ethernet0/0` — `ip nat inside`
- `Ethernet0/1` — `ip nat outside`

---

## Task 2 — Restore and Verify

Brought interfaces back up:
```
interface ethernet0/0
 no shutdown
interface ethernet0/1
 no shutdown
```

Both returned to `up/up` confirmed via `show ip interface brief`.

**NAT translation table after PC1 and PC2 pinged 1.1.1.1:**
```
Pro  Inside global        Inside local        Outside local    Outside global
icmp 216.0.5.2:1025      192.168.1.50:694    1.1.1.1:694      1.1.1.1:1025
icmp 216.0.5.2:1024      192.168.1.51:691    1.1.1.1:691      1.1.1.1:1024
```

Both inside hosts (`192.168.1.50` and `192.168.1.51`) share the same inside global (`216.0.5.2`) — differentiated only by port number. PAT confirmed working.

**NAT statistics:**
```
Total active translations: 2 (0 static, 2 dynamic; 2 extended)
Dynamic overload mapping configured: 1
[Id: 2] access-list 1 interface Ethernet0/1 refcount 2
Hits: 1635  Misses: 0
```

`Dynamic overload mapping configured: 1` — this line explicitly confirms PAT is active, not standard dynamic NAT.

---

## Mistakes Made

**Attempted to remove the pool before removing the rule that references it — three times.**

The error `%Pool Cafe-Public in use, cannot destroy` means the pool is still being referenced by an `ip nat inside source` mapping. The correct order is:

1. Stop traffic (shut interfaces)
2. Clear translations (`clear ip nat translation *`)
3. Remove the **binding rule** (`no ip nat inside source list 1 pool Cafe-Public`)
4. Then remove the pool definition if needed

Trying to delete the pool while the rule still pointed at it would never work regardless of how many times translations were cleared. The rule is the reference — the pool can't be destroyed while anything points to it.

---

## Key Observations

- **PAT signature in the NAT table:** same inside global IP (`216.0.5.2`) appearing on multiple rows with different port numbers — that's the definitive visual proof of overload vs dynamic NAT
- **`Dynamic overload mapping configured: 1`** in `show ip nat statistics` explicitly identifies the overload rule — useful for quick verification
- **ACL was reused** — no need to recreate it when migrating from dynamic NAT to PAT; only the binding rule changes
- **Interface method adapts automatically** — if the ISP changes the public IP on Ethernet0/1, the overload rule follows it without any config change needed
- **Pool definition (`ip nat pool Cafe-Public`) remained in running-config** as an orphaned line after the binding rule was removed — it was harmless but could be cleaned up with `no ip nat pool Cafe-Public`

---

## Commands Used

```bash
# Verify starting state
show ip nat translations
show running-config | include ip nat inside source

# Stop traffic and clear state
interface ethernet0/0
 shutdown
interface ethernet0/1
 shutdown
clear ip nat translation *

# Remove dynamic NAT binding (must happen before pool can be removed)
no ip nat inside source list 1 pool Cafe-Public

# Configure PAT
ip nat inside source list 1 interface ethernet0/1 overload

# Restore interfaces
interface ethernet0/0
 no shutdown
interface ethernet0/1
 no shutdown

# Verify PAT is active
show ip nat translations        # both hosts share same global IP, different ports
show ip nat statistics          # look for "Dynamic overload mapping configured: 1"
show ip interface brief         # confirm up/up
```

---

## NAT Evolution — All Three Types

| Type | Config keyword | Mapping | Port tracking |
|------|---------------|---------|---------------|
| Static NAT | `static` | 1-to-1, permanent | No |
| Dynamic NAT | `list X pool Y` | many-to-many (pool) | No |
| NAT Overload / PAT | `list X interface Y overload` | many-to-one | Yes |
