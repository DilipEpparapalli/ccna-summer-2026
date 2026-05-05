# What is a Network?

## Simple Explanation
A network is two or more devices communicating and sharing data. The goal has never changed since the 1960s — get data from point A to point B. Everything else is just solving the problems that appear when you scale that idea up.

## Why It Exists
Isolated devices are useless at scale. Networking was invented so computers could stop being islands and start working together — across a room, a building, or the planet.

## How It Works
- Two devices can connect directly
- More than two devices need a **switch** to coordinate local communication
- Multiple networks need a **router** to move traffic between them
- The internet is just millions of networks connected by routers

## Core Devices
| Device | Job |
|--------|-----|
| Switch | Connects devices *inside* a local network |
| Router | Connects *different* networks together |
| Firewall | Allows good traffic, blocks bad traffic |
| WAP (Wireless Access Point) | Extends the network over WiFi |

> **Home vs Enterprise:** At home, one box does all four jobs. In a business, each device is separate and purpose-built.

## Exam Traps
1. A WAP is **not** a router — it extends the network wirelessly but still connects back into a switch
2. The internet is not magic — it's routers making billions of forwarding decisions per second
3. "Network" doesn't mean internet — a switch connecting two PCs with no internet access is still a network

## Recall Questions
1. What's the difference between a switch and a router in one sentence each?
2. Why can't two devices on different IP networks communicate through a switch alone?
3. What does a WAP actually connect to behind the scenes?
