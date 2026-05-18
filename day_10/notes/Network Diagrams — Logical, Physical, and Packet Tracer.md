
## Simple Explanation
A logical diagram shows what your network does — IP addresses, subnets, and traffic flow. A physical diagram shows what it looks like in reality — exact devices, port numbers, and cable runs. A Packet Tracer lab is the digital mirror of both: a safe environment to build, test, and break your design without touching production gear.

## Why It Exists
Networks grow over time. Design decisions made during setup get forgotten. Diagrams preserve your intent permanently — so future engineers (including future you) aren't guessing at 2am when something breaks or needs to be rebuilt from scratch. Packet Tracer extends this by giving you a testbed before any real hardware is touched.

## How It Works

### Step 1 — Build the logical diagram first
- Identify every major device (routers, switches, APs, servers, firewalls)
- Draw connections showing traffic flow
- Label IP addresses and subnets on each segment
- Record design decisions — why a device is placed where it is

### Step 2 — Build the physical diagram from the logical
- Map every logical device to its real hardware counterpart
- Label exact interfaces on both ends of every connection
  - Example: Router Gi0/0 → Switch Gi0/24 (not just "router connects to switch")
- Note cable types where relevant (Cat6, fiber, DAC, etc.)

### Step 3 — Mirror the design in Packet Tracer
- Drag in devices that match the roles in your diagram (router, switch, AP, server, cloud)
- Connect manually — choose the cable type and exact interface yourself
  - Do NOT use the auto-connect lightning bolt tool
  - Reason: manually selecting ports builds mental anchors — your brain logs exactly which ports are connected, which is what makes troubleshooting fast later
- Adapt to lab hardware without changing the design
  - Example: Packet Tracer's 2960 uses FastEthernet (100 Mbps) instead of GigabitEthernet (1000 Mbps) — the speed changes, the topology does not

### Step 4 — Build to the diagram, not the other way around
- The physical install and the Packet Tracer lab must both match the diagram exactly
- If the rack or the sim doesn't match the diagram, one of them is wrong — fix it

## Key Terms
| Term | Meaning |
|------|---------|
| Logical diagram | Shows IP addresses, subnets, traffic flow — the network's behaviour |
| Physical diagram | Shows exact devices, port numbers, cable runs — the network's reality |
| Interface label | The specific port used on a device (e.g. GigabitEthernet0/0) |
| Topology | The layout/structure of how devices are connected — who connects to who |
| Uplink | The port connecting a switch up toward the router or core |
| Redundancy | Backup paths or devices to survive a single point of failure |
| Lightweight AP | An access point with no local intelligence — requires a WLC to be configured |
| WLC | Wireless LAN Controller — centrally configures and manages all APs from one place |
| FastEthernet | 100 Mbps Ethernet — common on older/simulated gear |
| GigabitEthernet | 1000 Mbps Ethernet — standard on modern enterprise hardware |

## Logical vs Physical — Side by Side

```
LOGICAL                          PHYSICAL
-------                          --------
Router                           Cisco ISR4321
  |                                Gi0/0 ──────────────── Switch Gi0/24
  | 10.0.0.0/24                    Gi0/1 ──────────────── ISP handoff
Switch
  |── AP1  (SSID: Staff)         Catalyst 2960 (24-port)
  |── AP2  (SSID: Guest)           Gi0/1  ── AP1 (ceiling mount, east)
  |── Server 10.0.0.10             Gi0/2  ── AP2 (ceiling mount, west)
                                   Gi0/3  ── Server
```

## Packet Tracer Topology (Coffee House)

```
         [Cloud/ISP]
              |
         [Router]
          Gi0/0 (to ISP)
          Gi0/1 (to Switch1 Fa0/24)
              |
     [Switch1]─────────[Switch2]
      Fa0/1─AP1    Fa0/1─AP2
      Fa0/2─Server
```

## Real-World Connection
In an enterprise branch deployment, the logical diagram is built during the design phase and handed to the installation team alongside the physical diagram. Engineers use Packet Tracer or GNS3 to validate the design before any hardware ships. When a fault occurs in production, the on-call engineer pulls up both diagrams and immediately knows which cable, which port, and which device to check.

For wireless at scale: a retail chain with 50 locations and 3 APs each has 150 access points. Without a WLC, changing the guest WiFi password means logging into 150 devices individually. With a WLC, it is one change pushed everywhere instantly. That is why lightweight APs exist.

## Exam Traps
1. Logical and physical diagrams are both drawn documents — not one drawn and one real. The difference is what they represent, not where they exist.
2. A different port label (FastEthernet vs GigabitEthernet) does not mean a different network design. Topology is about who connects to who — not the speed of the link.
3. Redundant switch links look good on a physical diagram but create Layer 2 loops without STP. Physical design must account for protocol requirements, not just hardware placement.
4. Interface naming like Gi0/0/1 (three numbers) indicates a modular chassis — slot/module/port. Gi0/1 (two numbers) is module/port. These are not interchangeable on an exam.
5. Lightweight APs cannot operate without a WLC — they have no local configuration capability. Do not confuse with autonomous APs, which are self-contained.

## Commands (if applicable)
```
! View all interfaces and their status
show ip interface brief

! View detailed info on a specific interface
show interface GigabitEthernet0/0

! See what is physically connected to each port (CDP must be enabled)
show cdp neighbors detail

! Confirm interface naming on your specific hardware
show version
```

## Recall Questions
1. What is the difference between a logical and a physical network diagram?
2. Why should you manually select ports in Packet Tracer instead of using auto-connect?
3. The Packet Tracer 2960 shows FastEthernet instead of GigabitEthernet. Does this break your design? Why or why not?
4. What problem does a WLC solve, and why does it matter at scale?
5. You inherit a network with no documentation. What do you build first, and in what order?
6. The rack is fully cabled but doesn't match the diagram. Which one do you trust, and what do you do next?