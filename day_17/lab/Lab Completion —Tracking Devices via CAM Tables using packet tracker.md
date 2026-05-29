
**Environment:** Cisco Packet Tracer, check the packet tracer file 
**Topology:** 2 core switches (SW1, SW2) → 4 access switches (SW3–SW6) → 8 PCs + 2 servers

---

## Topology

```
          SW1 ──────────────── SW2
         /    \              /    \
       SW3    SW4          SW5    SW6
      /   \  /   \        /   \  /   \  \
    PC1  PC2 PC3 PC4    PC5  PC6 PC7 PC8 SRV1/SRV2
```

All devices on VLAN 1 (192.168.1.0/24). Uplinks are FastEthernet trunks between switches.  
CDP confirms switch-to-switch connections at each hop.

---

## The Workflow (burn this in)

```
1. ping <IP>              ← force ARP exchange, refresh both tables
2. show arp               ← IP → MAC translation
3. show mac address-table ← MAC → Port mapping
4. Multiple MACs on port? → another switch is behind it
5. show cdp neighbors     ← identify which switch is on that port
6. Log into next switch → repeat from step 3
7. Single MAC on port → you found the endpoint
```

---

## Scenario 1 — Slow Device: 192.168.1.18

### Starting switch (SW1, IP 192.168.1.1)

**Step 1 — Ping to refresh ARP**
```
Switch#ping 192.168.1.18
Success rate is 80 percent (4/5), min/avg/max = 0/2/9 ms
```

**Step 2 — ARP lookup: IP → MAC**
```
Switch#show arp
Internet  192.168.1.18    0   0005.5E5D.B43C   ARPA   Vlan1
```
✅ Target MAC: `0005.5e5d.b43c`

**Step 3 — CAM lookup: MAC → Port**
```
Switch#show mac address-table
   1    0005.5e5d.b43c    DYNAMIC     Fa0/2
   1    00e0.a30d.a801    DYNAMIC     Fa0/2   ← multiple MACs on Fa0/2
```
⚠️ Multiple MACs on Fa0/2 → another switch is behind this port

**Step 4 — CDP: who is on Fa0/2?**
```
Switch#show cdp neighbors
Switch    Fas 0/2    167    S    2960    Fas 0/1
```
→ Move to that switch (SW connected on Fa0/2)

---

### Hop 2 — Next switch (SW, IP 192.168.1.4)

**Step 5 — Ping again to keep tables fresh**
```
Switch#ping 192.168.1.18
Success rate is 80 percent (4/5)
```

**Step 6 — CAM lookup**
```
Switch#show mac address-table
   1    0005.5e5d.b43c    DYNAMIC     Fa0/4
```

**Step 7 — Check if Fa0/4 has multiple MACs**
Only `0005.5e5d.b43c` appears on Fa0/4 → single device on this port

**Step 8 — Verify with interface**
```
Switch#show interfaces fastEthernet 0/4
FastEthernet0/4 is up, line protocol is up (connected)
Full-duplex, 100Mb/s
```

### ✅ Scenario 1 Result
> **192.168.1.18** → MAC `0005.5e5d.b43c` → **Fa0/4** on the access switch (SW4 area)  
> Device is directly connected. Single endpoint confirmed.

---

## Scenario 2 — Security Breach: 192.168.1.11

### Starting switch (SW1)

**Step 1 — Ping**
```
Switch>ping 192.168.1.11
Success rate is 80 percent (4/5), min/avg/max = 0/2/9 ms
```

**Step 2 — ARP**
```
Switch>show arp
Internet  192.168.1.11    0   0005.5E3C.0DC1   ARPA   Vlan1
```
✅ Target MAC: `0005.5e3c.0dc1`

**Step 3 — CAM**
```
Switch>show mac address-table
   1    0005.5e3c.0dc1    DYNAMIC     Fa0/3
   1    0060.3e1c.4902    DYNAMIC     Fa0/3   ← multiple MACs on Fa0/3
```
⚠️ Multiple MACs on Fa0/3 → another switch behind it

**Step 4 — CDP**
```
Switch>show cdp neighbors
Switch    Fas 0/3    167    S    2960    Fas 0/2
```
→ Move to that switch

---

### Hop 2 — Next switch (SW, IP 192.168.1.3)

**Step 5 — Ping**
```
Switch#ping 192.168.1.11
Success rate is 80 percent (4/5)
```

**Step 6 — ARP**
```
Switch#show arp
Internet  192.168.1.11    0   0005.5E3C.0DC1   ARPA   Vlan1
```

**Step 7 — CAM**
```
Switch#show mac address-table
   1    0005.5e3c.0dc1    DYNAMIC     Fa0/1
```

**Step 8 — Check Fa0/1**
```
Switch#show cdp neighbors
Switch    Fas 0/1    143    S    2960    Fas 0/1
```
Still a switch on Fa0/1 → one more hop

---

### Hop 3 — Final switch (access layer)

**Step 9 — CAM**
```
Switch#show mac address-table
   1    0005.5e3c.0dc1    DYNAMIC     Fa0/1
```
Only one MAC on Fa0/1 — no CDP neighbor that matters

### ✅ Scenario 2 Result
> **192.168.1.11** → MAC `0005.5e3c.0dc1` → **Fa0/1** on the access switch (SW5/SW6 area)  
> Device located. Ready for malware scan.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ping <IP>` | Refresh ARP and CAM tables — always first |
| `show arp` | Translate IP → MAC address |
| `show mac address-table` | Find which port a MAC was learned on |
| `show cdp neighbors` | Identify which switch is behind a port |
| `show interfaces fa0/X` | Verify port is up and connected |
| `show ip interface brief` | Check switch's own IP and active ports |

---

## Key Observations from This Lab

**The 80% ping result is normal** — the first packet is always dropped while ARP resolves. Don't mistake it for a connectivity problem.

**Multiple MACs on one port = switch behind it.** Every time you saw more than one MAC on a port, a downstream switch was there. A single MAC on a port means you've hit the endpoint.

**CDP is the map.** Without CDP you'd be guessing which physical switch to console into next. With it, you follow the trail by name and port.

**ARP ages out fast** (default 4 minutes on these switches). Always ping before checking `show arp` — a stale table will send you hunting a ghost.

---

## Recall Questions

1. You ping a device and `show arp` still shows no entry. What happened and what do you do?
2. You find the target MAC on Fa0/3, but there are 4 other MACs on that same port. What does that tell you and what's your next command?
3. What is the default ARP timeout on Cisco IOS, and why does it matter for device tracking?
4. You reach a switch where the target MAC appears on Fa0/2 and CDP shows no neighbor on Fa0/2. What does that mean?
5. Why does the first ping packet always fail (80% success rate) when hunting a device that hasn't been recently active?
