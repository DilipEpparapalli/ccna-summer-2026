
## Simple Explanation

An open switch port is an attack surface. Port security is a layered approach to making switch ports useless to unauthorized devices — by shutting unused ports down, isolating available ports into dead-end networks, and locking active ports to specific MAC addresses so only trusted hardware can communicate.

## Why It Exists

A device like a Shark Jack can plug into any active switch port, grab an IP address via DHCP, and immediately learn the subnet mask, default gateway, DNS server, and scan every host on the network. Port security removes the conditions that make that attack possible.

## The Threat Model

```
Open switch port → attacker plugs in
→ DHCP responds with IP address
→ Attacker learns: subnet mask, default gateway, DNS server
→ nmap scans all hosts and open ports
→ Network is fully mapped in seconds
```

For this attack to work, three conditions must be true:

1. The port must be active (not shut down)
2. A DHCP server must be reachable from that port
3. The attacker must be able to reach other hosts on the network

Remove any one condition and the attack fails.

---

## The Three Layers of Defense

### Layer 1 — Shut Down Unused Ports

Every port that has no legitimate device connected should be administratively disabled.

```ios
! Find all currently down interfaces
show ip interface brief | include down

! Shut down an unused port
interface FastEthernet0/1
 shutdown
```

**Port states — know the difference:**

|State|Meaning|Who caused it|
|---|---|---|
|Down|Nothing plugged in — but will come up if something connects|Nobody|
|Administratively down|Intentionally disabled by admin|You|
|Error-disabled|Automatically shut down by switch due to security violation|The switch|

An administratively down port will not come up even if something is plugged in. That is the goal.

---

### Layer 2 — Black Hole VLAN

For ports that cannot be fully shut down, isolate them into a VLAN that leads nowhere. No DHCP server, no default gateway, no path to production devices.

```ios
! Create the black hole VLAN
vlan 666

! Assign unused but active ports to it
interface FastEthernet0/10
 switchport access vlan 666

interface FastEthernet0/11
 switchport access vlan 666
```

An attacker who plugs into one of these ports gets an active link and absolutely nothing else. No IP address, no gateway, no targets to reach.

**Real world tip:** use both Layer 1 and Layer 2 together on unused ports. Networks drift — configs get partially applied, templates get changed, someone manually brings a port up. If one control fails, the other is still there.

---

### Layer 3 — Port Security

Handles the sneakiest attack: an attacker unplugs a legitimate device and substitutes their own on an active, production port.

**How it works:**

The switch tracks which MAC address is allowed on each port. If a different MAC address appears, the switch takes the configured violation action immediately.

```
Normal:
[Pi — MAC: AA:BB:CC:DD:EE:FF] → plugged in → switch learns MAC → traffic flows

Attack:
[Pi unplugged] → [Shark Jack — MAC: 11:22:33:44:55:66] plugged in
→ switch sees unauthorized MAC
→ VIOLATION
→ port enters error-disabled state
→ attacker gets nothing
```

**Configuration:**

```ios
interface GigabitEthernet0/1
 switchport mode access                      ! hardcode as access port
 switchport port-security                    ! enable port security
 switchport port-security maximum 1          ! allow only 1 MAC address
 switchport port-security mac-address sticky ! learn first MAC, lock it in
 switchport port-security violation shutdown ! error-disable on violation
```

**MAC address options:**

|Method|How it works|When to use|
|---|---|---|
|Manual|You type the exact MAC address|Small deployments, high control|
|Sticky|Switch learns the first MAC it sees and locks it|Most common — no manual lookup needed|
|Forbidden|Explicitly deny a specific MAC|Rare — block a known bad device|

**Maximum value:** use `maximum 2` when an IP phone with a PC plugged into its passthrough port shares one switch port — two MACs, one port.

---

### Violation Modes

|Mode|Port State After Violation|Traffic|SNMP Alert|
|---|---|---|---|
|**Shutdown** (default)|Error-disabled|Blocked|Yes|
|**Restrict**|Stays up|Dropped|Yes|
|**Protect**|Stays up|Dropped|No|

Shutdown is the default and most common in production. The port goes error-disabled — meaning the **switch** shut it down automatically, not the admin.

---

### Recovery After a Violation

```ios
! Port is error-disabled after a security violation
! Step 1: reconnect the legitimate device
! Step 2: bounce the interface

interface GigabitEthernet0/1
 shutdown
 no shutdown

! Verify recovery
show port-security interface GigabitEthernet0/1
```

Look for **Secure-up** — port is healthy. **Secure-shutdown** means it is still error-disabled.

---

## Key Terms

|Term|Meaning|
|---|---|
|Administratively down|Port disabled by admin via `shutdown` command|
|Error-disabled|Port automatically shut down by switch due to policy violation|
|Sticky MAC|Switch learns and permanently locks the first MAC address seen on a port|
|Black hole VLAN|A VLAN with no DHCP, no gateway, and no path to production — isolates unused ports|
|Violation|Event where an unauthorized MAC address appears on a port-security-enabled port|
|SNMP|Protocol used to send alerts to network management systems|

---

## Real-World Connection

**Enterprise floor switch — layered approach:**

```
UNUSED PORTS (no device connected):
  shutdown → administratively down
  + switchport access vlan 666 → black hole
  = two controls, if one drifts the other holds

ACTIVE PORTS (legitimate devices):
  port-security sticky → locks to first MAC seen
  violation shutdown → wrong MAC = port dies instantly
  maximum 1 → one device per port (or 2 for phone+PC)
```

**Scenario — office deployment:** 48-port switch. 30 ports active with known devices. 18 ports unused. Action: shut all 18 unused ports and assign them to VLAN 666. On all 30 active ports, configure port-security sticky with violation shutdown. A contractor plugs a personal laptop into a colleague's port while they step away — switch detects the MAC mismatch, error-disables the port, sends an SNMP alert. Attacker gets nothing.

---

## Beyond Port Security (CCNA Awareness)

Port security is the baseline. Enterprise environments extend this with:

|Technology|What it does|
|---|---|
|802.1X|Requires device or user authentication before port activates|
|Certificate-based auth|Device must present a valid cert — no login prompt|
|Cisco ISE|Intelligent device profiling and automated policy enforcement|

Do not skip port security basics because advanced options exist. Open unmanaged ports with no protection are the most common real-world failure.

---

## Exam Traps

1. **Error-disabled vs administratively down.** The exam will describe a port that was shut down and ask why. Error-disabled = switch did it due to a violation. Administratively down = you did it with `shutdown`. They look similar in output but mean completely different things.
    
2. **Sticky does not survive a reload by default.** Unless you save the running config (`copy running-config startup-config`), sticky MAC addresses are lost when the switch reloads. Always save after configuring sticky.
    
3. **Violation mode default is shutdown.** If the exam asks what happens when port security is enabled but no violation mode is configured — the answer is shutdown. That is the default behavior.
    
4. **Maximum default is 1.** If no maximum is configured, only one MAC address is permitted. A second device appearing on that port — even a legitimate one — triggers a violation.
    

---

## Commands

```ios
! Enable port security with sticky and shutdown violation
interface GigabitEthernet0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation shutdown

! Verify port security status
show port-security
show port-security interface GigabitEthernet0/1
show port-security address

! Check port states
show ip interface brief
show interfaces GigabitEthernet0/1 status

! Recover from error-disabled state
interface GigabitEthernet0/1
 shutdown
 no shutdown

! Save sticky MAC addresses across reloads
copy running-config startup-config
```

---

## Recall Questions

1. What three conditions must be true for a Shark Jack attack to succeed?
2. What is the difference between a port that is "down" and one that is "administratively down"?
3. What is a black hole VLAN and what makes it effective?
4. Why use both shutdown AND a black hole VLAN on unused ports?
5. What does sticky MAC do and why is it preferred over manual MAC entry?
6. A port security violation occurs. What state does the port enter and who caused it?
7. What are the three violation modes? What does each do to traffic and alerting?
8. How do you recover a port from error-disabled state?
9. Why might you set `maximum 2` on a port instead of `maximum 1`?
10. What happens to sticky MAC addresses if the switch reloads and you never saved the config?