## Simple Explanation
DTP (Dynamic Trunking Protocol) lets switch ports negotiate whether to become a
trunk automatically. VTP (VLAN Trunking Protocol) replicates VLAN databases across
switches automatically. Both were built for convenience. Both introduce risk that
outweighs that convenience in most modern environments. The professional move is to
understand what they do — then disable them.

## Why It Exists
DTP was built so switches could auto-detect and form trunk links without manual
configuration. VTP was built so admins could create a VLAN once and have it
propagate automatically to every switch in the environment. In theory: less manual
work, faster deployment. In practice: unpredictable behavior and serious security
and operational risks.

## How It Works

### DTP — Dynamic Trunking Protocol

DTP is Cisco-proprietary. It runs on switch ports and negotiates whether a link
should become a trunk. There are three relevant port modes:

| Mode | Behavior |
|------|----------|
| `trunk` | Always a trunk. Sends DTP frames to tell the other side. |
| `dynamic desirable` | Actively tries to negotiate a trunk with the other side. |
| `dynamic auto` | Waits for the other side to initiate. Becomes a trunk if asked. |
| `access` | Always an access port. Still sends DTP frames unless nonegotiate is set. |

**Negotiation outcome table — what forms based on both sides:**

```
                   Other side:
                   trunk    desirable    auto    access
This side:
trunk              TRUNK    TRUNK        TRUNK   access*
desirable          TRUNK    TRUNK        TRUNK   access
auto               TRUNK    TRUNK        access  access
access             access*  access       access  access
```

*Mismatched modes can cause connectivity issues — avoid.

The important one to memorize: **auto + auto = access (no trunk forms)**. Both
sides wait for the other to initiate — nobody does, nothing happens.

**The security risk:** If DTP is left running on an access port, a device plugged
in can negotiate the port into trunk mode. A trunk carries all VLANs. An attacker
who successfully triggers this gains access to traffic they should never see. This
is the foundation of a **VLAN hopping attack**.

**The fix — `switchport nonegotiate`:**

```
interface range fastEthernet 0/1 - 2
 switchport nonegotiate
```

This disables DTP entirely on the interface. The port is whatever you configured it
to be — trunk or access — and it will not negotiate with anything. No DTP frames
sent, no DTP frames honored.

**Best practice — always configure port mode explicitly:**

```
! Access port — explicit and locked
interface fastEthernet 0/5
 switchport mode access
 switchport access vlan 10
 switchport nonegotiate

! Trunk port — explicit and locked
interface fastEthernet 0/24
 switchport mode trunk
 switchport nonegotiate
```

Manual config + nonegotiate = predictable, auditable, secure.

---

### VTP — VLAN Trunking Protocol

Despite the name, VTP has nothing to do with how trunking works. The actual trunking
standard is **802.1Q**. VTP does one thing: **replicates VLAN database information
between switches**.

Create VLAN 30 on one switch → VTP advertises it → every other switch in the same
VTP domain automatically creates VLAN 30. Sounds convenient. The problem is it also
replicates deletions.

**VTP Modes:**

| Mode | Behavior |
|------|----------|
| Server | Creates, modifies, and deletes VLANs. Advertises changes to the domain. **Default on many Cisco switches.** |
| Client | Accepts VLAN updates from servers. Cannot create or delete VLANs locally. |
| Transparent | Manages its own VLAN database locally. Does not send or accept VTP updates. Passes VTP advertisements through (doesn't block them). |

The mode to use in modern networks: **Transparent**.

```
vtp mode transparent
```

In transparent mode the switch says: "I manage my own VLANs. I'm not learning from
you, and I'm not pushing to you." Predictable. Safe. No surprises.

**VTP Domain:**

VTP updates are scoped by domain name. Switches only exchange VTP information if
they share the same domain name. Domain names are **case-sensitive** — `COOKIE`
and `cookie` are treated as different domains.

```
vtp domain cookie
```

**From the lab — setting transparent mode and domain name:**

```
cafe01-SW01(config)# vtp mode transparent
Setting device to VTP TRANSPARENT mode.

cafe01-SW01(config)# vtp domain cookie
Changing VTP domain name from NULL to cookie
```

**Verify VTP status:**

```
cafe01-SW01# show vtp status

VTP Operating Mode    : Transparent
VTP Domain Name       : cookie
Configuration Revision: 0          ← resets to 0 in transparent mode
Number of existing VLANs: 7
```

Configuration Revision is the version number VTP uses to decide whose database is
newer. In server/client mode, a switch with a higher revision number wins and can
overwrite other switches' VLAN databases. In transparent mode, revision resets to 0
and this risk disappears entirely.

---

### The Two VTP Nightmares

**Nightmare 1 — Accidental VLAN deletion:**

An admin decommissions a switch and clears its config. That cleanup removes VLANs.
If the switch is a VTP server, those deletions propagate. Devices across the network
lose their VLAN assignments. Everything in those VLANs stops working.

**Nightmare 2 — Reused lab switch:**

An old lab switch with a high configuration revision number gets plugged into
production. VTP sees a higher revision and treats that switch's VLAN database as
authoritative. It overwrites the production VLAN database. Entire VLANs disappear.

Both scenarios are why most enterprise environments either use VTP transparent mode
or disable VTP entirely.

---

### VTP vs 802.1Q — Know the Difference

This is an exam trap. The names sound related. They are not.

| Protocol | What It Does | Layer |
|----------|-------------|-------|
| 802.1Q | Tags frames on trunk links so switches know which VLAN a frame belongs to | Layer 2 (Data Link) |
| VTP | Replicates VLAN database information between switches | Management plane |

VTP is about VLAN *database* synchronization. 802.1Q is about VLAN *frame* tagging.
You need 802.1Q for trunking to work. You do not need VTP for anything — it's
optional and usually better off disabled.

## Key Terms

| Term | Meaning |
|------|---------|
| DTP | Dynamic Trunking Protocol — Cisco-proprietary; negotiates trunk formation between ports |
| `dynamic auto` | Port waits to be asked before forming a trunk |
| `dynamic desirable` | Port actively tries to form a trunk |
| `switchport nonegotiate` | Disables DTP on the interface; port mode is locked to what you configured |
| VTP | VLAN Trunking Protocol — Cisco-proprietary; replicates VLAN database across switches |
| VTP Server | Default mode; creates/deletes VLANs and advertises changes |
| VTP Client | Accepts VLAN updates; cannot make local changes |
| VTP Transparent | Manages VLANs locally; ignores VTP updates; safest mode |
| VTP Domain | Name that scopes which switches exchange VTP updates; case-sensitive |
| Configuration Revision | Counter VTP uses to determine whose database is newest; higher wins |
| VLAN hopping | Attack where a device exploits DTP negotiation to gain trunk access and reach unintended VLANs |
| 802.1Q | The actual trunking standard — tags frames on trunk links; unrelated to VTP |

## Real-World Connection

In modern enterprise environments, the standard practice is:

- Set all switch ports to explicit `access` or `trunk` mode
- Apply `switchport nonegotiate` to disable DTP
- Set all switches to `vtp mode transparent`
- Manage VLANs manually per-switch or via a proper network management platform

This eliminates the unpredictability of auto-negotiation and the operational risk of
VTP propagation. The tradeoff — slightly more manual work when adding VLANs — is
worth it. A boring, predictable network is a healthy network.

## Exam Traps

1. **VTP is not a trunking protocol** — despite the name, VTP has nothing to do with
   how frames are tagged on trunk links. That's 802.1Q. VTP only replicates VLAN
   database entries between switches. Getting these confused is a common wrong answer.

2. **`auto + auto = no trunk`** — both ports wait for the other to initiate. Neither
   does. The link stays as access. If you expect a trunk and both sides are `dynamic
   auto`, nothing will happen.

3. **VTP transparent mode still passes VTP frames** — it doesn't participate in VTP
   updates for its own database, but it will forward VTP advertisements received on
   trunk ports to other switches. It's transparent to the VTP traffic, not blocking it.

4. **Configuration Revision is the danger number** — a switch with a higher revision
   number can overwrite the VLAN database of every switch in the domain. An old lab
   switch with revision 47 showing up on a production network with revision 12 will
   win. Transparent mode resets revision to 0 and eliminates this risk.

5. **`switchport nonegotiate` requires the port to already be in trunk or access mode**
   — you can't apply nonegotiate to a port still in dynamic mode. Set the mode first,
   then disable negotiation.

## Commands

```
! Verify current DTP/trunk negotiation mode
show interfaces fastEthernet 0/1 trunk
show interfaces fastEthernet 0/1 switchport

! Disable DTP negotiation (apply after setting explicit mode)
interface fastEthernet 0/1
 switchport mode access          ! or trunk
 switchport nonegotiate

! Check VTP status
show vtp status

! Set VTP to transparent mode (safest)
vtp mode transparent

! Set VTP domain name
vtp domain <name>

! Verify VTP counters (check if any VTP traffic is flowing)
show vtp counters
```

## Recall Questions

1. What is the outcome when one port is `dynamic desirable` and the other is
   `dynamic auto`? What about `auto` and `auto`?
2. What attack does leaving DTP enabled on an access port make possible? How does
   `switchport nonegotiate` prevent it?
3. What is the difference between VTP and 802.1Q? Why does confusing them matter
   on the exam?
4. A switch with VTP configuration revision 47 gets plugged into a production network
   running revision 12. What happens, and how does `vtp mode transparent` prevent it?
5. In VTP transparent mode, does the switch block VTP advertisements from passing
   through? Explain what transparent actually means.
