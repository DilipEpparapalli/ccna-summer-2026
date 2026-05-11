
## What Packet Tracer Is
A network simulator by Cisco. Lets you build, configure, and test networks without physical hardware.

## Two Workspace Views
| View | Shows |
|------|-------|
| Logical | How devices connect — topology, relationships |
| Physical | Where devices live — rooms, racks, geography |

Use logical for learning concepts. Use physical when location and cable runs matter.

## Realtime vs Simulation Mode
| Mode | Behavior |
|------|----------|
| Realtime | Traffic flows at normal speed — just like a live network |
| Simulation | Traffic pauses between steps — inspect every frame |

Key insight: the network behaves identically in both modes.
Simulation just slows it down so you can watch.

## Why Simulation Mode Matters
- See ARP happen before a ping completes
- Watch a switch flood frames when destination is unknown
- Inspect frame headers at each hop
- Builds the mental model that Wireshark will later confirm

## Preferences Worth Knowing
- Show Port Labels → see which interface connects to what
- Notes > Labels for placement control on complex topologies
- Functional over flashy — readable diagrams beat pretty ones

## Exam Traps
- Simulation mode is a learning tool — no equivalent exists
  on real Cisco gear. Real-world equivalent is Wireshark.
- Physical view ≠ logical view. Don't confuse topology
  with location.

## Recall Questions
1. What does Simulation mode let you inspect that Realtime doesn't?
2. If a switch sees an unknown destination MAC, what does it do?
3. What's the difference between a logical and physical topology view?