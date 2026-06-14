
## Lab Overview
**Scenario:** Castle Rysen cafe floor migrated from static one-to-one NAT mappings to dynamic pool-based NAT. Static entries for two cafe workstations were removed and replaced with an ACL-defined inside network mapped to a public address pool.

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
                          NAT pool: 216.0.5.50 – 216.0.5.100
```
![[ccna-summer-2026/day_29/lab/Network_Diagram.png]]
---

## Task 0 — Clear Static NAT Translations

Static entries were present from the previous lab:

```
ip nat inside source static 192.168.1.50 216.0.5.20
ip nat inside source static 192.168.1.51 216.0.5.21
```

Attempted to remove while translations were active — router blocked it:

```
Cafe-Rtr(config)#no ip nat inside source static 192.168.1.50 216.0.5.20
Static entry in use, do you want to delete child entries? [no]: no
%: Error: static entry in use, cannot remove
```

**Fix:** cleared the translation table first, then removed the static entries:

```
Cafe-Rtr# clear ip nat translation *
Cafe-Rtr(config)# no ip nat inside source static 192.168.1.50 216.0.5.20
Cafe-Rtr(config)# no ip nat inside source static 192.168.1.51 216.0.5.21
```

**Verified:** `show ip nat translations` returned empty — clean slate confirmed.

---

## Task 1 — Define Inside Address List

Created a standard ACL to identify eligible inside addresses:

```
Cafe-Rtr(config)# access-list 1 permit 192.168.1.0 0.0.0.255
```

**Verified:**
```
Cafe-Rtr# show ip access-lists
Standard IP access list 1
    10 permit 192.168.1.0, wildcard bits 0.0.0.255
```

---

## Task 2 — Create Public Address Pool

```
Cafe-Rtr(config)# ip nat pool Cafe-Public 216.0.5.50 216.0.5.100 netmask 255.255.255.0
```

**Verified in running-config:**
```
ip nat pool Cafe-Public 216.0.5.50 216.0.5.100 netmask 255.255.255.0
```

Pool size: 51 public IP addresses (216.0.5.50 through 216.0.5.100 inclusive).

---

## Task 3 — Bind ACL to Pool

```
Cafe-Rtr(config)# ip nat inside source list 1 pool Cafe-Public
```

Interface roles were already correct from previous lab:
- `Ethernet0/0` — `ip nat inside`
- `Ethernet0/1` — `ip nat outside`

---

## Task 4 — Verify Dynamic Translations

**PC1 and PC2 pinged 1.1.1.1 (`ping -c 4 1.1.1.1`) — both successful.**

NAT table after traffic:
```
Pro  Inside global       Inside local       Outside local   Outside global
icmp 216.0.5.50:698     192.168.1.50:698   1.1.1.1:698     1.1.1.1:698
---  216.0.5.50         192.168.1.50       ---             ---
icmp 216.0.5.51:700     192.168.1.51:700   1.1.1.1:700     1.1.1.1:700
---  216.0.5.51         192.168.1.51       ---             ---
```

PC1 borrowed `216.0.5.50`, PC2 borrowed `216.0.5.51` — each device got a unique pool address.

**NAT statistics:**
```
Total active translations: 4 (0 static, 4 dynamic; 2 extended)
pool Cafe-Public: total addresses 51, allocated 2 (3%), misses 0
Hits: 48  Misses: 0
```

---

## Mistakes Made

**1. Tried to remove a static NAT rule while translations were active**  
The router rejected the `no ip nat inside source static` command because the translation was in use. Fix: run `clear ip nat translation *` first to flush the table, then remove the rule. This is the correct production sequence — not shutting down the interface first.

**2. Pool start address typo**  
Initially entered:
```
ip nat pool Cafe-Public 216.0.5.100 216.0.5.100 netmask 255.255.255.0
```
Both start and end were set to `.100`, creating a single-address pool instead of the intended 51-address range. Caught during `show running-config` verification. Fixed by re-entering the correct command — IOS-XE accepted the overwrite without requiring a `no` first:
```
ip nat pool Cafe-Public 216.0.5.50 216.0.5.100 netmask 255.255.255.0
```

**Key lesson:** always verify pool range with `show running-config | include ip nat pool` immediately after creation.

---

## Deliberate Test — Static NAT Removal Confirmed

Between removing static NAT and configuring dynamic NAT, a ping was run from PC1 to 1.1.1.1 — it failed with 100% packet loss as expected. This confirmed the static entries were truly gone before the dynamic rule was in place. Same test on PC2. Both failures were intentional verification steps, not errors.

---

## Key Observations

- Dynamic NAT assigned pool addresses sequentially — PC1 got `.50`, PC2 got `.51`
- The `---` rows in the NAT table are the persistent translation entries (no protocol/port); the `icmp` rows are the active session entries
- `show ip nat statistics` is more useful than the translation table for confirming pool health — shows total addresses, allocated count, and miss rate at a glance
- Pool exhaustion would silently fail translation for device 52 onward — no error to the client

---

## Commands Used

```bash
# View and clear NAT state
show ip nat translations
show ip nat statistics
clear ip nat translation *

# Remove static NAT entries
no ip nat inside source static 192.168.1.50 216.0.5.20
no ip nat inside source static 192.168.1.51 216.0.5.21

# Define inside address selector
access-list 1 permit 192.168.1.0 0.0.0.255

# Create public pool
ip nat pool Cafe-Public 216.0.5.50 216.0.5.100 netmask 255.255.255.0

# Bind ACL to pool
ip nat inside source list 1 pool Cafe-Public

# Verify
show ip access-lists
show running-config | include ip nat pool
show ip nat translations
show ip nat statistics
```
