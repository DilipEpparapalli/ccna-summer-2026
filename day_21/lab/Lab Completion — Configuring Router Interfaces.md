## What Was Done

### Task 0 — Baseline Security

Secured the router before touching any interfaces.

```
username cisco password cisco          ← tested type-0 warning (see note below)
service password-encryption            ← applied Type 7 obfuscation
username cisco secret cisco            ← correct production approach: MD5/scrypt hash
enable secret CrC0ffee!
line console 0 → login local
line vty 0 4   → login local
write memory
```

**Note on password vs secret:** `password` stores plaintext; `service password-encryption`
applies a reversible Type 7 cipher — trivially decoded with freely available tools.
`secret` stores a real one-way hash (MD5/Type 5 on older IOS, scrypt/Type 9 on modern
IOS-XE). The two are not equivalent. In production, always use `secret`.

---

### Task 1 — Interface Inventory

Ran `show ip interface brief` before touching anything. All eight interfaces confirmed
`administratively down / down` with no IPs assigned. Method column showed `TFTP`,
indicating addresses were previously attempted via TFTP boot — not manually assigned.

```
Ethernet0/0 through Ethernet0/3   ← module 0, ports 0–3
Ethernet1/0 through Ethernet1/3   ← module 1, ports 0–3
All: administratively down / down
```

---

### Task 2 — Interface Activation

Configured `Eth0/0` for LAN, left `Eth0/1` shut with a description for future WAN
hand-off.

```
interface ethernet0/0
 description Coffee House LAN
 ip address 192.168.42.1 255.255.255.0
 no shutdown

interface ethernet0/1
 description WAN
 (no shutdown — left administratively down)
```

Post-config `show ip interface brief` confirmed:

```
Ethernet0/0    192.168.42.1    YES manual    up    up
Ethernet0/1    unassigned      YES TFTP      administratively down    down
```

---

### Task 3 — Verification

**CDP neighbor discovery:**

```
show cdp neighbors detail
→ Device ID: Cafe-SW1
→ IP address: 192.168.42.2
→ Interface: Ethernet0/0 ↔ Ethernet0/0
→ Platform: Linux Unix (IOS-XE 17.16.1a)
```

**Interface description verification:**

```
show interfaces description
Et0/0    up / up        Coffee House LAN
Et0/1    admin down     WAN
```

Note: `show interfaces status` is not a valid command on this IOS-XE platform —
confirmed via `?`. Use `show ip interface brief` or `show interfaces description`
instead.

**Ping test to Cafe-SW1 (192.168.42.2):**

```
First ping:  .!!!!  — 80% success (1 drop)
Second ping: !!!!!  — 100% success
```

First-probe drop caused by ARP resolution — the router had no MAC entry for
192.168.42.2 and had to send an ARP request before the first ICMP echo could be
forwarded. Subsequent pings hit the ARP cache and succeeded immediately.

**Telnet verification:**

```
telnet 192.168.42.2
→ Connected, prompted for username/password
→ Logged into Cafe-SW1 successfully, then exited
```

Confirmed VTY access on the switch is reachable from the router across the management
subnet.

**Saved configuration:**

```
write memory → [OK]
```

---

## What This Proved

- Router interfaces require explicit `no shutdown` — confirmed administratively down
  state on all eight interfaces before any configuration was applied
- IP assignment alone does not activate an interface
- CDP discovered Cafe-SW1 and returned its IP, platform, and IOS version without
  needing a network diagram
- First-ping ARP drop is expected behavior, not a connectivity fault
- `service password-encryption` provides Type 7 obfuscation only — not a substitute
  for `secret`
