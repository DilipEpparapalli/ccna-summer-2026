## Simple Explanation
A trunk port carries traffic for multiple VLANs simultaneously over a single link.
Where an access port belongs to one VLAN and connects to an end device, a trunk port
connects switch infrastructure together and uses 802.1Q tags to keep VLAN traffic
sorted across the link. Without trunks, a VLAN is trapped on one switch — an island.

## Why It Exists
Once you create VLANs on a switch, those VLANs need to extend across the network.
You can't dedicate a separate cable per VLAN between every pair of switches — that
doesn't scale. Trunking solves this by carrying all VLANs over one link, with each
frame tagged to identify which VLAN it belongs to.

A second driver: each VLAN maps to its own IP subnet. Adding VLANs means adding
subnets. If a /26 served one flat network, splitting into two VLANs requires
splitting that /26 into two /27s — one subnet per VLAN. The subnetting and the VLAN
design move together.

## How It Works

### The Problem Without Trunks

```
SW01                          SW02
┌─────────────┐               ┌─────────────┐
│ VLAN 10 ✓  │               │ VLAN 10 ✓  │
│ VLAN 20 ✓  │               │ VLAN 20 ✓  │
│             │               │             │
│  Fa0/1 ────┼───────────────┼─ Fa0/1     │
└─────────────┘               └─────────────┘
         access port (VLAN 1)
```

Both switches have VLAN 10 and VLAN 20 configured, but if the inter-switch link is
an access port, it only carries one VLAN. VLAN 10 on SW01 has no way to reach VLAN
10 on SW02. The VLANs are isolated per switch.

### The Solution — Trunk Port

```
SW01                          SW02
┌─────────────┐               ┌─────────────┐
│ VLAN 10 ───┼─── tagged ────┼─── VLAN 10 │
│ VLAN 20 ───┼─── tagged ────┼─── VLAN 20 │
│             │  trunk link   │             │
│  Fa0/1 ────┼───────────────┼─ Fa0/1     │
└─────────────┘               └─────────────┘
```

The trunk carries both VLANs simultaneously. Each frame is tagged with its VLAN ID
so the receiving switch knows exactly where to deliver it.

### Configuring a Trunk Port

One command sets a port to trunk mode unconditionally:

```
interface range fastEthernet 0/1 - 2
 switchport mode trunk
```

When you apply this, the interfaces flap — they briefly go down then come back up.
This is normal: the port is renegotiating its mode. In production, this causes a
brief outage, so trunk configuration should happen before connecting production
devices.

```
! What you see when applying trunk config:
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to down
%LINEPROTO-5-UPDOWN: Line protocol on Interface FastEthernet0/1, changed state to up
```

### Verifying the Trunk

`show interfaces trunk` is the key verification command. It tells you everything
you need to know about trunk status:

```
cafe01-SW01# show interfaces trunk

Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1
Fa0/2       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005
Fa0/2       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,20
Fa0/2       1,10,20

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,20
Fa0/2       none
```

Reading this output section by section:

| Section | What It Tells You |
|---------|-------------------|
| Mode / Encapsulation / Status | Port is explicitly set to trunk, using 802.1Q, and actively trunking |
| Native vlan | VLAN 1 — untagged traffic on this trunk belongs here |
| Vlans allowed on trunk | 1–1005 by default — no pruning has been applied |
| Vlans allowed and active | Only VLANs that actually exist on this switch appear here — VLAN 10 and 20 show up because they were created |
| Vlans in STP forwarding state | VLANs actively passing traffic; "none" on Fa0/2 means STP is still converging or no active devices |

### What Happens to Ports in `show vlan` After Trunking

Trunk ports disappear from `show vlan` output. This is expected and confuses a lot
of people.

```
! Before trunk config — Fa0/1 and Fa0/2 show under VLAN 1:
1    default    active    Fa0/1, Fa0/2, Fa0/3 ...

! After trunk config — Fa0/1 and Fa0/2 are gone from show vlan:
1    default    active    Fa0/3, Fa0/4, Fa0/5 ...
```

Trunk ports are not members of any single VLAN — they carry all of them. `show vlan`
only lists access port assignments. Use `show interfaces trunk` to see trunk ports.

### Proving the Trunk Works — Same VLAN, Different Switches

After configuring trunks, a PC on SW01 Fa0/4 (VLAN 10) and a PC on SW02 Fa0/7
(VLAN 10) should be able to ping each other — same VLAN, same subnet, trunk
carrying the traffic between switches.

```
SW01                    trunk                   SW02
[PC-A]──Fa0/4──VLAN 10──Fa0/1────Fa0/1──VLAN 10──Fa0/7──[PC-B]
192.168.10.x                                    192.168.10.y

Ping PC-A → PC-B: SUCCESS ✓   (same VLAN, trunk working)
```

Then move PC-B to VLAN 20 — pings die immediately. The separation is real:

```
Ping PC-A (VLAN 10) → PC-B (VLAN 20): FAIL ✗
```

Different VLANs = different subnets = need a router to communicate. No router
configured yet, so the separation holds.

### The Dangerous Trunk Command

By default, trunks carry all VLANs (1–1005). You can restrict this with the
`switchport trunk allowed vlan` command — but it has a trap:

```
! SAFE — adds VLAN 10 to the allowed list (keeps everything else)
switchport trunk allowed vlan add 10

! DANGEROUS — replaces the entire allowed list with ONLY VLAN 10
switchport trunk allowed vlan 10
```

The second command looks like it's adding VLAN 10. It isn't. It's replacing the
entire allowed list. Every other VLAN — including management VLAN 1 — is now blocked
on that trunk. In production, this causes an immediate outage and can lock you out
of the switch remotely.

Always use `add` and `remove` when modifying the allowed VLAN list on a live trunk.
Verify the result immediately with `show interfaces trunk`.

### 802.1Q vs ISL

| Standard | Status | Use |
|----------|--------|-----|
| 802.1Q | Industry standard | Always use this |
| ISL | Cisco proprietary, deprecated | Legacy only — may appear on old exam objectives |

802.1Q inserts a 4-byte tag into the frame header. Every vendor that supports 802.1Q
can trunk with every other vendor that supports it. ISL is dead — if you see it,
it's either a very old network or an exam distractor.

## Key Terms

| Term | Meaning |
|------|---------|
| Trunk port | Switch port that carries multiple VLANs simultaneously using 802.1Q tags |
| Access port | Switch port that belongs to exactly one VLAN; connects to end devices |
| 802.1Q | Industry standard for VLAN tagging on trunk links |
| Native VLAN | The VLAN whose traffic crosses a trunk *untagged*; default is VLAN 1 |
| ISL | Old Cisco-proprietary trunking standard; deprecated, replaced by 802.1Q |
| VLAN pruning | Restricting which VLANs are allowed across a specific trunk link |
| Interface flap | Brief down/up cycle when a port changes mode; normal during trunk config |
| `show interfaces trunk` | Primary command to verify trunk status, encapsulation, and active VLANs |

## Real-World Connection

In any multi-switch environment — a floor with two access switches, a building with
a distribution switch, a campus — trunk links are the backbone that lets VLANs span
the physical infrastructure. A network engineer configuring a new switch always
starts with: create the VLANs, configure the trunk uplinks, then assign access ports.
That sequence matters — trunk first, then access ports, to avoid connectivity gaps.

VLAN pruning (restricting allowed VLANs on trunks) becomes important at scale.
In a large campus, you don't want student VLAN traffic flooding across every trunk
to every switch in the building. Pruning keeps traffic local to where it's needed
and reduces broadcast noise across the whole network.

## Exam Traps

1. **Trunk ports don't appear in `show vlan`** — they're not members of any single
   VLAN. If you don't see a port in `show vlan`, check `show interfaces trunk` before
   assuming something is wrong.

2. **`switchport trunk allowed vlan X` replaces the list, it doesn't add to it** —
   always use `add` keyword when adding VLANs to an existing trunk. This is one of
   the most common real-world outage causes and a favorite exam scenario.

3. **Interface flap during trunk config is normal** — ports go down then up when
   switching modes. This is expected. In production, plan for it.

4. **Both sides of a trunk must be configured** — if SW01 Fa0/1 is a trunk but SW02
   Fa0/1 is still an access port, the link won't trunk properly. Mismatched port
   modes are a common misconfiguration.

5. **Native VLAN mismatch triggers STP errors** — if both sides of a trunk don't
   agree on the native VLAN, you'll see Spanning Tree PVID errors in the console.
   Both sides must match.

## Commands

```
! Configure trunk port (single or range)
interface range fastEthernet 0/1 - 2
 switchport mode trunk

! Verify trunk status — use this, not show vlan
show interfaces trunk

! Assign an access port to a VLAN (for testing)
interface fastEthernet 0/4
 switchport mode access
 switchport access vlan 10

! Safely add/remove VLANs from trunk allowed list
switchport trunk allowed vlan add 30
switchport trunk allowed vlan remove 30

! DANGEROUS — replaces entire allowed list
! switchport trunk allowed vlan 30   ← don't do this unless you mean it

! Verify VLAN membership (access ports only — trunks won't appear here)
show vlan brief
```

## Recall Questions

1. A trunk port disappears from `show vlan` output after you configure it. Is this
   a problem? Where do you look instead to confirm the trunk is working?
2. You run `switchport trunk allowed vlan 20` on a live trunk. What just happened,
   and why is it dangerous?
3. What does the "Vlans allowed and active in management domain" section of
   `show interfaces trunk` tell you — and how is it different from "Vlans allowed
   on trunk"?
4. You configure `switchport mode trunk` on SW01 Fa0/1 but forget to configure SW02
   Fa0/1. What will `show interfaces trunk` show, and what symptom will you see?
5. Ports Fa0/1 and Fa0/2 were in VLAN 1 before being converted to trunks. After
   trunk config, where does VLAN 1 untagged traffic go on those ports?
