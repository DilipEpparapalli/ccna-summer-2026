# Day 1 Labs — Cisco Packet Tracer

## What is Cisco Packet Tracer?

Packet Tracer is Cisco's free network simulation tool. It lets you build and test virtual networks — routers, switches, PCs, cables, the whole thing — without needing any physical hardware. You get a real Cisco IOS CLI, live packet simulation, and full control over the topology. Fantastic tool for learning how networks actually behave.

## Labs

### Lab 1 — Switch in Action
**File:** `ccna_switch_day1.pkz`

Opened the topology in simulation mode, filtered the event list to focus on relevant traffic, then opened the command prompt on a PC and pinged another device inside the same LAN. Watched the switch build its CAM table.

### Lab 2 — Router in Action
**File:** `ccna_router_day1.pkz`

Same approach — simulation mode, command prompt, but this time pinged a device on a different network. Watched the traffic leave the LAN, hit the router, get forwarded across to the other network. Saw the Layer 2 frame change at each hop while the Layer 3 IP stayed the same end-to-end. Router doing its job at Layer 3.
