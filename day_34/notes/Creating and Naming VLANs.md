## Simple Explanation
Creating a VLAN is a two-step process: first you create the VLAN and give it a name,
then you assign switch ports to it. Until ports are assigned, the VLAN exists as an
empty shell — the rooms are built but nobody has moved in. Port assignment is what
creates the actual traffic separation.

## Why It Exists
Understanding VLANs in theory is one thing. Configuring them is where the separation
actually happens. A business requirement like "separate guest devices from admin
systems with a security boundary" translates directly into VLAN creation and port
assignment on the switch — that's the tool that makes it real.

## How It Works

### VLAN 1 — Already There by Default

Every Cisco switch ships with all ports in VLAN 1. You don't create it — it's always
there. When you run `show vlan` on a fresh switch, every port shows up under VLAN 1.

This also explains the SVI (Switch Virtual Interface): the switch's management IP
lives on `interface vlan 1` by default, which is why any device plugged into a default
port can reach it — they're all in the same VLAN.

```
! On a default switch, show vlan reveals:
VLAN Name        Status    Ports
1    default      active    Fa0/1 ... Fa0/24, Gig0/1, Gig0/2
```

You'll also see VLANs 1002–1005 (FDDI, Token Ring legacy entries). These are
historical artifacts — ignore them.

### Step 1 — Create the VLAN and Name It

From global config mode, type the VLAN number to create it. The switch drops you into
VLAN config mode where you assign a name.

```
cafe01-SW01(config)#vlan 10
cafe01-SW01(config-vlan)#name ADMIN-DEVICES
cafe01-SW01(config-vlan)#exit

cafe01-SW01(config)#vlan 20
cafe01-SW01(config-vlan)#name PATRON-DEVICES
cafe01-SW01(config-vlan)#exit
```

Always name your VLANs. Numbers alone become confusing fast. All-caps names stand
out visually when scanning `show vlan` output — easy to distinguish what you
configured versus what the system generated.

**Verify the VLANs exist:**

```
cafe01-SW01#show vlan

VLAN Name                             Status    Ports
1    default                          active    Fa0/1 ... Fa0/24, Gig0/1, Gig0/2
10   ADMIN-DEVICES                    active
20   PATRON-DEVICES                   active
```

Notice: VLAN 10 and VLAN 20 are active but have no ports listed. The VLAN exists —
but no ports have been assigned to it yet. No separation has happened yet.

### Step 2 — Assign Ports to a VLAN

This is where the separation actually happens. Use `interface range` to configure
multiple ports at once — faster and more realistic than port-by-port config.

Two commands are required on every access port:

```
switchport mode access       ← locks the port to exactly one VLAN
switchport access vlan XX    ← specifies which VLAN that is
```

**From the lab — assigning Fa0/10 through Fa0/15 to VLAN 20 (PATRON-DEVICES):**

```
cafe01-SW01(config)#interface range fastEthernet 0/10 - 15
cafe01-SW01(config-if-range)#switchport mode access
cafe01-SW01(config-if-range)#switchport access vlan 20
cafe01-SW01(config-if-range)#end
```

**Verify the port assignment:**

```
cafe01-SW01#show vlan

VLAN Name                             Status    Ports
1    default                          active    Fa0/1 ... Fa0/9, Fa0/16 ... Fa0/24
10   ADMIN-DEVICES                    active
20   PATRON-DEVICES                   active    Fa0/10, Fa0/11, Fa0/12, Fa0/13
                                                Fa0/14, Fa0/15
```

Fa0/10–15 are now gone from VLAN 1 and live in VLAN 20. Any device plugged into
those ports is now in the PATRON-DEVICES broadcast domain — isolated from everything
in VLAN 1 and VLAN 10.

### What `switchport mode access` Actually Does

Cisco switch ports default to `dynamic auto` mode — meaning they can negotiate
whether to become an access port or a trunk port depending on what's plugged in.

That's a security problem. If someone plugs in another switch (or a device pretending
to be one), the port can negotiate into trunk mode — which carries all VLANs. That
person now has access to traffic they were never supposed to see. This is the
foundation of a **VLAN hopping attack**.

Explicitly setting `switchport mode access` removes the negotiation entirely. The
port is an access port, period. No negotiation. No accidents.

```
! Dynamic mode (default — avoid this on user ports)
switchport mode dynamic auto

! Explicit access mode (what you should always use)
switchport mode access
switchport access vlan 20
```

### The Full Picture After This Lesson

```
cafe01-SW01 after configuration:

VLAN 1  (default)     → Fa0/1–9, Fa0/16–24, Gig0/1–2  (unassigned ports)
VLAN 10 (ADMIN)       → no ports assigned yet
VLAN 20 (PATRON)      → Fa0/10–15
```

VLAN 10 still has no ports assigned — that's intentional. The trunk link between
switches needs to be configured before moving production ports, otherwise you create
isolation without connectivity. Ports without a trunk in place means devices that
used to communicate suddenly can't — a real outage, not a theoretical one.

## Key Terms

| Term | Meaning |
|------|---------|
| VLAN 1 | Default VLAN on every Cisco switch; all ports start here |
| SVI | Switch Virtual Interface — the logical interface tied to a VLAN that gives the switch an IP address |
| VLAN config mode | Entered via `vlan XX`; where you name a VLAN |
| `switchport mode access` | Locks a port to one VLAN; disables trunk negotiation |
| `switchport access vlan XX` | Assigns the port to a specific VLAN |
| `interface range` | Configures multiple ports simultaneously |
| Dynamic auto | Default port mode; negotiates access or trunk — avoid on user ports |
| VLAN hopping | Attack where a device exploits dynamic port mode to gain access to unintended VLANs |

## Real-World Connection

In a real deployment, VLAN creation and port assignment happen before devices are
even plugged in. You design the VLAN structure first (what VLANs, which ports, which
subnets), configure the switch, then connect devices into the right ports. Doing it
in reverse — plugging devices in first, then reconfiguring VLANs — causes brief
outages and confusion every time.

The pattern from this lab maps directly to enterprise practice: named VLANs,
explicitly configured access ports, and never leaving user-facing ports in dynamic
mode. Those three habits alone make your switch configs predictable and auditable.

## Exam Traps

1. **Creating a VLAN doesn't assign any ports** — after `vlan 10 / name X`, the VLAN
   exists but is empty. `show vlan` will show it active with no ports. Ports must be
   explicitly assigned.

2. **`switchport mode access` and `switchport access vlan XX` are two separate
   commands** — one sets the port type, one sets the VLAN. Doing only one doesn't
   complete the config.

3. **Ports removed from VLAN 1 don't disappear — they move** — when you assign a
   port to VLAN 20, it drops out of VLAN 1. It belongs to exactly one VLAN at a time
   (on an access port).

4. **VLAN 1 cannot be deleted** — it's the default VLAN and always exists on Cisco
   switches. VLANs 1002–1005 also cannot be deleted (legacy).

5. **`show vlans` is wrong — `show vlan` is correct** — as seen in the lab output,
   `show vlans` throws an invalid input error.

## Commands

```
! Create a VLAN and name it
vlan 10
 name ADMIN-DEVICES

vlan 20
 name PATRON-DEVICES

! Assign a range of ports to a VLAN as access ports
interface range fastEthernet 0/10 - 15
 switchport mode access
 switchport access vlan 20

! Verify VLANs and port assignments
show vlan

! Verify IP interfaces (confirm SVI and port status)
show ip interface brief
```

## Recall Questions

1. You create VLAN 30 and name it SERVERS. You run `show vlan` and see it listed
   with no ports. Is this correct behavior? What do you need to do next?
2. What is the risk of leaving switch ports in `dynamic auto` mode, and what attack
   does it enable?
3. What two commands are required to fully configure an access port, and what does
   each one do?
4. Why should you configure trunk links before moving ports out of VLAN 1 on
   production switches?
5. A port is currently in VLAN 1. You run `switchport access vlan 20`. Where does
   the port appear in `show vlan` output now?
