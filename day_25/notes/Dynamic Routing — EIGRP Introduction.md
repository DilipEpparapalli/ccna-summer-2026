## Simple Explanation

Dynamic routing lets routers automatically share route information with each other instead of requiring manual static entries. EIGRP (Enhanced Interior Gateway Routing Protocol) is one protocol that handles this sharing — routers form neighbor relationships, exchange what networks they know about, and populate each other's routing tables automatically.

## Why It Exists

Static routes work fine for small topologies. Two routers, one link — no problem. But as a network grows (more sites, more VLANs, redundant WAN links), manually entering every route becomes error-prone and unmanageable. One missing route causes a silent drop. Dynamic routing eliminates that burden: routers discover and adapt to the network automatically.

## How It Works

### Step 1 — Enable EIGRP and set the AS number

```
Router(config)# router eigrp 1
```

The `1` is the **autonomous system (AS) number** — a group ID that all participating routers must share. Routers with mismatched AS numbers will see each other's hellos and silently ignore them. No adjacency forms. No error message appears. This is a common troubleshooting trap.

### Step 2 — Advertise networks with `network` statements

```
Router(config-router)# network 10.0.0.0
Router(config-router)# network 192.168.1.0
```

Each `network` statement does **two things simultaneously**:

1. Activates EIGRP on any interface whose IP falls within that network — causing the router to send **multicast hellos** (to `224.0.0.10`) on that interface
2. Advertises that network to EIGRP neighbors so they can learn it

> ⚠️ This is not broadcast — EIGRP uses **multicast** (224.0.0.10), not broadcast. Common exam distractor.

### Step 3 — Neighbor adjacency forms

When two routers with matching AS numbers receive each other's hellos on a shared link, they form an **adjacency** (neighbor relationship). Once adjacent, they exchange routing information.

### Step 4 — Routes appear in the routing table

Learned routes are marked with **`D`** in the routing table:

```
D    192.168.2.0/24 [90/2172416] via 10.0.0.2, 00:01:03, GigabitEthernet0/1
```

The `D` stands for EIGRP — derived from **DUAL** (Diffusing Update ALgorithm), the algorithm EIGRP uses internally to select loop-free paths. It does *not* stand for "dynamic."

### Topology: Castle Rysen EIGRP Lab

```
[Cafe-PC]
    |
[Cafe-SW1]
    |
[Cafe-RT1]─────────── WAN /30 ────────────[Fallout-RT1]
  .1 (G0/0)                                    .2 (G0/0)
  192.168.1.0/24 (LAN)                         192.168.2.0/24 (LAN)
                                                    |
                                              [Fallout-SW1]
                                                    |
                                              [Fallout-Server]
```

- EIGRP enabled on WAN link and internal LAN interfaces
- EIGRP **not** enabled on Cafe-RT1's internet-facing interface
- After adjacency forms: Cafe-RT1 learns `192.168.2.0/24` via `D` route; Fallout-RT1 learns `192.168.1.0/24` via `D` route

## Key Terms

| Term | Meaning |
|------|---------|
| Dynamic Routing | Routers automatically share and learn routes without manual configuration |
| EIGRP | Enhanced Interior Gateway Routing Protocol — Cisco's dynamic routing protocol used here |
| Autonomous System (AS) | A group of routers sharing the same routing domain; identified by a number (e.g., `1`) |
| Adjacency / Neighbor | Two routers that have recognized each other via hellos and agreed to exchange routes |
| DUAL | Diffusing Update ALgorithm — EIGRP's internal path-selection algorithm; source of the `D` label |
| Multicast 224.0.0.10 | The address EIGRP uses to send hello messages (not broadcast) |
| `D` in routing table | Route learned via EIGRP |
| Network Statement | Command that activates EIGRP on an interface AND advertises that network to neighbors |

## Real-World Connection

In an enterprise with dozens of branch offices, dynamic routing protocols like EIGRP or OSPF handle route distribution automatically. When a new branch comes online, its router forms adjacencies with neighbors and the entire network learns its subnets within seconds — no engineer needs to touch every other router. At scale, this is the difference between a manageable network and a configuration nightmare.

## Exam Traps

1. **AS number mismatch = silent failure.** Routers with different EIGRP AS numbers ignore each other's hellos. No error, no log message — just no adjacency. Always verify AS numbers match when troubleshooting EIGRP.

2. **`D` ≠ "dynamic."** The `D` label in the routing table specifically identifies EIGRP routes. It comes from DUAL, EIGRP's algorithm — not from the word "dynamic." Other protocols have their own letters (`O` = OSPF, `R` = RIP, `S` = static).

3. **`network` statement does two things.** It both activates EIGRP on matching interfaces AND advertises that network. Forgetting to add a network statement means that interface won't send hellos and that network won't be shared — even if EIGRP is globally enabled.

4. **Don't enable EIGRP on internet-facing interfaces.** Advertising internal routes toward an ISP exposes your topology unnecessarily and can cause route leakage if the ISP runs a matching EIGRP AS. Always scope dynamic routing to internal links only.

## Commands

```
! Enable EIGRP with AS number 1
Router(config)# router eigrp 1

! Advertise networks (activates EIGRP on matching interfaces + shares network with neighbors)
Router(config-router)# network 10.0.0.0
Router(config-router)# network 192.168.1.0

! Verify neighbors have formed
Router# show ip eigrp neighbors

! Verify routing table — look for 'D' entries
Router# show ip route

! Verify which networks EIGRP is advertising
Router# show ip eigrp topology
```

## Recall Questions

1. What are the two things a `network` statement does when configured under `router eigrp`?
2. What AS number must neighbor routers share, and what happens if they don't match?
3. What does the `D` in the routing table actually stand for, and why isn't it `E`?
4. Why should you avoid enabling EIGRP on an internet-facing interface?
5. What multicast address does EIGRP use to send hello messages?
