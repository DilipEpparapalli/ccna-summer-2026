## Simple Explanation

Router interfaces work a lot like switch interfaces in terms of CLI — same environment,
same commands, mostly the same logic. The critical difference is that router interfaces
are **administratively down by default**. You must explicitly run `no shutdown` before
any traffic flows, regardless of whether a cable is plugged in.

## Why It Exists

Routers often sit at network boundaries where misconfiguration can have wide impact.
Keeping interfaces administratively down by default is a safety mechanism — it forces
an engineer to make a deliberate decision before any interface becomes active on the
network. On a switch, a port lighting up when a cable is plugged in is expected and
low-risk. On a router, that same behavior could expose a network segment before it's
ready.

## How It Works

1. **Enter global config mode** — `configure terminal`
2. **Select the interface** — `interface GigabitEthernet0/0/0` (name varies by hardware)
3. **Assign an IP address** — `ip address <address> <subnet-mask>`
4. **Optionally add a description** — `description <text>`
5. **Bring the interface up** — `no shutdown`
6. **Verify** — `show ip interface brief`

The interface only becomes active after step 5. Steps 3 and 4 alone do nothing visible
to the network.

### Why configure before enabling?

If you apply changes while an interface is live, it can flap — bouncing up and down
repeatedly as settings are applied. On a switch, excessive flapping can trigger
**err-disabled** state, where the switch shuts the port down automatically to protect
the network. You then have to manually recover it. The clean habit: finish your full
config, then run `no shutdown` once.

## Key Terms

| Term | Meaning |
|------|---------|
| Administratively down | Interface disabled in software; requires `no shutdown` to activate |
| `no shutdown` | IOS command that brings an interface up from administratively down state |
| Flapping | Interface bouncing up and down rapidly, often caused by config changes on a live port |
| Err-disabled | Switch safety state triggered by repeated instability; port shuts down and must be manually recovered |
| `show ip interface brief` | Quick-view command showing all interfaces, their IPs, and up/down status |
| CDP | Cisco Discovery Protocol — discovers directly connected Cisco neighbors |
| LLDP | Link Layer Discovery Protocol — vendor-neutral neighbor discovery |

## Interface Naming

Router interface names reflect physical hardware layout. Fixed routers have simple
names like `GigabitEthernet0/0`. Modular routers add another layer:
`GigabitEthernet0/0/0` — that's module / submodule / port. More slashes = more
modular hardware. Always run `show ip interface brief` on an unfamiliar device to
confirm what interfaces exist before you start configuring.

## Real-World Connection

In an enterprise environment, a router being staged for deployment would be fully
configured — IP addresses, descriptions, security settings — before any `no shutdown`
commands are run. This is standard procedure in data center work: configure everything
in the rack with cables unplugged or interfaces shut, verify the config on paper, then
cut over. The alternative — configure live — risks traffic disruption and err-disabled
ports at exactly the wrong moment.

## Exam Traps

1. **Administratively down ≠ no cable** — A switch port shows `down/down` when nothing
   is plugged in, and comes up automatically when a cable is inserted. A router
   interface stays `administratively down` even with a live cable until you run
   `no shutdown`. These look similar in `show ip interface brief` output but have
   completely different causes and fixes.

2. **IP address ≠ interface is up** — Assigning an IP address to a router interface
   does not activate it. The interface remains administratively down until `no shutdown`
   is explicitly issued.

3. **Interface naming varies by platform** — `GigabitEthernet0/0` on one router,
   `GigabitEthernet0/0/0` on another. Exam questions may use either format. Read the
   interface name carefully and don't assume a fixed layout.

## Commands

```
! Base configuration (do this before touching interfaces)
hostname Cafe01-RT01
enable secret <password>
line vty 0 4
 password <password>
 login
!
! Interface configuration
interface GigabitEthernet0/0/0
 description Link to Cafe-SW1
 ip address 192.168.1.1 255.255.255.0
 no shutdown
!
! Verification
show ip interface brief
show cdp neighbors
show cdp neighbors detail
```

## Recall Questions

1. A router interface shows `administratively down / down` in `show ip interface brief`.
   A cable is plugged in and the other end is a live switch port. What is the fix, and
   why did the switch port come up but the router interface didn't?

2. You assign an IP address to a router interface. Is the interface now reachable? Why
   or why not?

3. You're logged into an unfamiliar router with no documentation. What two commands help
   you understand what interfaces exist and what is physically connected to each one?

4. Why is it better practice to run `no shutdown` after completing your interface config
   rather than before? What specific failure mode are you avoiding?
