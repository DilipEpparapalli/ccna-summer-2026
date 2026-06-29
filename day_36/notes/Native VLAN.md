## Simple Explanation
Every trunk port has a native VLAN — the designated home for untagged frames that
arrive on that trunk. When a frame crosses a trunk with no 802.1Q tag, the switch
has no label to read, so it delivers that frame into whichever VLAN is configured
as native. By default that's VLAN 1. Best practice is to change it to a dedicated
management VLAN and leave VLAN 1 unused.

## Why It Exists
On a trunk port, every frame is supposed to carry an 802.1Q tag identifying its
VLAN. But not every device that connects to a trunk can tag traffic — historically
this included hubs, and today it includes infrastructure devices like wireless access
points and hypervisor hosts that need a management path separate from their tagged
data traffic. The native VLAN gives untagged traffic a known, deliberate destination
instead of being dropped.

## How It Works

### Tagged vs Untagged on a Trunk

```
Frame arrives on trunk port:

  Has 802.1Q tag?
  ├── YES → read the VLAN ID, deliver to that VLAN
  └── NO  → deliver to the Native VLAN (default: VLAN 1)
```

Every other VLAN on the trunk is tagged. The native VLAN is the one exception —
its traffic crosses the trunk untagged. This is what makes it distinct from VLAN 10,
VLAN 20, or any other user VLAN you create.

### The Default — VLAN 1

Out of the box, the native VLAN on every Cisco trunk is VLAN 1. That's the same
VLAN all ports start in, the same VLAN management SVIs often live in, and the same
VLAN that carries a lot of switch control traffic by default.

Leaving native VLAN at VLAN 1 is a problem because:
- VLAN 1 already carries switch management and control traffic
- Untagged trunk traffic gets mixed in with everything else
- VLAN 1 is the most commonly targeted VLAN in attacks — it's predictable

### Best Practice — Dedicated Management VLAN

Create a separate VLAN purely for infrastructure management. Assign it as the native
VLAN on all trunks. Put user devices in explicitly tagged VLANs. Leave VLAN 1 empty
and unused.

```
VLAN 1   → unused (never assign devices here)
VLAN 10  → Staff devices        tagged on trunk
VLAN 20  → Guest WiFi clients   tagged on trunk
VLAN 99  → Management/Native    untagged on trunk
            (switches, APs, hypervisor hosts)
```

**Change the native VLAN on a trunk:**

```
interface fastEthernet 0/24
 switchport trunk native vlan 99
```

Both sides of the trunk must use the same native VLAN setting.

### Modern Use Case — Infrastructure Management

The most common real-world use of the native VLAN today is giving infrastructure
devices a management IP that's separate from user traffic.

Example — wireless access point on a trunk:

```
Trunk link to access point:
  Tagged   → VLAN 10  (staff WiFi traffic)
  Tagged   → VLAN 20  (guest WiFi traffic)
  Untagged → VLAN 99  (management access to the AP itself)
```

The AP uses the native VLAN to get its own IP address so admins can log in, update
firmware, and monitor it. The user VLANs ride the same trunk as tagged traffic
completely separate from the management path.

Same concept applies to:
- Hypervisor hosts running multiple VMs (VMs get tagged VLANs, host management
  gets native VLAN)
- Switches (management SVI lives in the native/management VLAN)
- Routers with trunk uplinks

### Native VLAN Mismatch — Why It Causes Chaos

Both sides of a trunk must agree on which VLAN is native. If they don't, you get a
**native VLAN mismatch** — one of the most confusing problems to troubleshoot because
traffic doesn't just stop, it goes to the wrong place.

```
SW1: native VLAN = 99        SW2: native VLAN = 1

Untagged frame leaves SW1 → SW1 treats it as VLAN 99 traffic
Frame arrives at SW2        → SW2 has no tag, delivers to VLAN 1

Result: VLAN 99 traffic leaks into VLAN 1
        Two VLANs accidentally bridged together
        Traffic crosses security boundaries it should never cross
```

IOS will warn you with a Spanning Tree PVID error when this happens:

```
%SPANTREE-2-RECV_PVID_ERR: Received 802.1Q BPDU on non trunk FastEthernet0/1 VLAN1.
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking FastEthernet0/1 on VLAN0001.
```

If you see these messages, check for a native VLAN mismatch immediately.

### Verify Native VLAN

```
show interfaces trunk

Port     Mode   Encapsulation  Status    Native vlan
Fa0/24   on     802.1q         trunking  1          ← default, should be changed
```

The Native vlan column shows the current native VLAN for each trunk. Both sides
of every trunk should show the same value here.

## Key Terms

| Term | Meaning |
|------|---------|
| Native VLAN | The VLAN that receives untagged frames arriving on a trunk port |
| Untagged frame | A frame with no 802.1Q tag; goes to the native VLAN on a trunk |
| Tagged frame | A frame with a 802.1Q VLAN ID stamped in the header; delivered to the correct VLAN |
| Native VLAN mismatch | Both sides of a trunk configured with different native VLANs; causes traffic to leak between VLANs |
| Management VLAN | A dedicated VLAN used only for administrative access to network infrastructure |
| PVID error | Spanning Tree warning that appears when a native VLAN mismatch is detected |
| `switchport trunk native vlan XX` | IOS command to change the native VLAN on a trunk port |

## Real-World Connection

In a well-designed enterprise network, VLAN 1 is left completely empty — no devices,
no management IPs, nothing. A dedicated management VLAN (commonly VLAN 99 or VLAN
100 by convention) is used as the native VLAN on all trunks. Every switch, AP, and
infrastructure device gets an IP in that management VLAN. Access to that VLAN is
restricted by firewall policy so only admins can reach it.

This design means user traffic in VLAN 10 or VLAN 20 never accidentally touches
infrastructure management, and a guest device on VLAN 20 has no path to log into
a switch or access point — even if they're connected through the same physical
hardware.

## Exam Traps

1. **Native VLAN is not a special separate concept — it's just whichever VLAN you
   designate for untagged traffic** — by default it's VLAN 1, but it can be any
   VLAN. The native VLAN itself still exists as a normal VLAN with ports and devices
   in it (or deliberately empty).

2. **Native VLAN traffic is untagged across the trunk** — frames in the native VLAN
   do NOT get an 802.1Q tag when crossing a trunk. Every other VLAN is tagged. This
   is the one exception.

3. **Both sides of a trunk must match** — native VLAN is configured per-port, not
   globally. If you change it on one side and forget the other, you have a mismatch.
   The network won't immediately crash but traffic will go to the wrong place silently.

4. **Native VLAN mismatch shows up as a Spanning Tree PVID error** — if you see
   `RECV_PVID_ERR` in the console, that's your signal to check native VLAN
   configuration on both sides of that trunk.

5. **Leaving native VLAN as VLAN 1 is a security risk** — VLAN 1 is always present
   on Cisco switches and carries switch management traffic by default. Mixing
   untagged trunk traffic into VLAN 1 expands its attack surface unnecessarily.

## Commands

```
! Change native VLAN on a trunk port
interface fastEthernet 0/24
 switchport trunk native vlan 99

! Verify native VLAN on all trunks
show interfaces trunk

! Check for native VLAN mismatch (look for PVID errors in logs)
show logging | include PVID

! Verify VLAN exists before assigning as native
show vlan brief
```

## Recall Questions

1. A frame arrives on a trunk port with no 802.1Q tag. Where does the switch send
   it, and what IOS command controls that behavior?
2. Why is leaving the native VLAN at VLAN 1 considered a security risk in a
   production network?
3. SW1 has native VLAN 99 on a trunk. SW2 has native VLAN 1 on the same trunk.
   What happens to untagged traffic crossing that link, and what error will you see?
4. A wireless access point is connected to a switch via a trunk carrying VLAN 10
   (staff) and VLAN 20 (guest). How does the native VLAN give the AP its own
   management IP address?
5. What is the difference between a tagged frame and an untagged frame on a trunk,
   and which one belongs to the native VLAN?
