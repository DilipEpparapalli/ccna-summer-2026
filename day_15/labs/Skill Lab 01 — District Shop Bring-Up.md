## Lab Scenario
Castle Rysen's District Shop Delta-7 needs its micro-campus network brought online.
Two access switches and a WAN edge router are staged. The job: establish identity,
harden access, configure management interfaces, and verify everything survives a reload.

---

## Topology

```
[DS-07-RTR1]
 Eth0/0 — 192.168.10.1/24
     |
     | (link-to-DS-07-SW1)
     |
[DS-07-SW1]                  [DS-07-SW2]
 Vlan1 — 192.168.10.11/24    Vlan1 — 192.168.10.12/24
 GW → 192.168.10.1           GW → 192.168.10.1
```

All three devices share the `192.168.10.0/24` management subnet.

---

## Concepts Covered

### 1. Device Identity
**Hostname** sets the CLI prompt and identifies the device in logs and SSH sessions.

```
Router(config)# hostname DS-07-RTR1
```

**Banner MOTD** displays a warning before login — legal notice and unauthorized access deterrent.

```
DS-07-RTR1(config)# banner motd ^C
Castle Rysen Ops: Authorized engineers only.
^C
```

> The `^C` is a delimiter — any character works, just must match start and end.

---

### 2. Password Hierarchy

| Protection Layer | Command | Protects |
|-----------------|---------|---------|
| Console access | `line con 0` → `password` + `login` | Physical console port |
| Remote access | `line vty 0 4` → `password` + `login` | SSH / Telnet sessions |
| Privileged EXEC | `enable secret` | The `#` prompt (global config access) |

**Enable secret vs enable password:**
- `enable secret` uses MD5/scrypt hashing — **always use this**
- `enable password` stores in weak reversible encryption — **never use alone**
- If both are set, `enable secret` wins

**Service password-encryption** applies type 7 obfuscation to plaintext passwords (console, VTY).
Type 7 is not true encryption — it can be reversed. It prevents shoulder-surfing, nothing more.

```
DS-07-SW1(config)# enable secret CrC0ffee!
DS-07-SW1(config)# service password-encryption
DS-07-SW1(config)# line con 0
DS-07-SW1(config-line)# password VaultAccess
DS-07-SW1(config-line)# login
DS-07-SW1(config-line)# logging synchronous
DS-07-SW1(config)# line vty 0 4
DS-07-SW1(config-line)# password ShelterAccess
DS-07-SW1(config-line)# login
DS-07-SW1(config-line)# transport input ssh
```

---

### 3. Switch Virtual Interface (SVI)

A Layer 2 switch has no IP address by default — it just forwards frames.
The **SVI (Switched Virtual Interface)** is a logical Layer 3 interface assigned to a VLAN.
It gives the switch itself an IP address so it can be reached for management (SSH, ping).

```
DS-07-SW1(config)# interface Vlan1
DS-07-SW1(config-if)# ip address 192.168.10.11 255.255.255.0
DS-07-SW1(config-if)# no shutdown
```

> `no shutdown` is required — SVIs are administratively down by default.

**Key distinction:** The SVI IP is for management traffic *to the switch itself*, not for routing
traffic between hosts. It only needs to be reachable on the local LAN.

---

### 4. Default Gateway on a Switch

A Layer 2 switch doesn't route — but it does generate its own management traffic
(SSH responses, ping replies, syslog). When that traffic needs to leave the local subnet,
the switch needs to know where to send it.

```
DS-07-SW1(config)# ip default-gateway 192.168.10.1
```

Without this, the switch can receive SSH connections from the local subnet but cannot
respond to anything outside `192.168.10.0/24`. Remote management from the bunker breaks.

---

### 5. Router Interface Bring-Up

Router interfaces are **shutdown by default** — unlike switch ports.
Must assign IP and explicitly bring online.

```
DS-07-RTR1(config)# interface Ethernet0/0
DS-07-RTR1(config-if)# description link-to-DS-07-SW1
DS-07-RTR1(config-if)# ip address 192.168.10.1 255.255.255.0
DS-07-RTR1(config-if)# no shutdown
```

---

### 6. Save and Verify

**Compare running vs startup config:**
```
# show running-config
# show startup-config
```
If they differ, changes haven't been saved — a reload will revert them.

**Save to NVRAM:**
```
# write memory
```
or
```
# copy running-config startup-config
```

**Verify after reload:**
- Console prompts for `VaultAccess` → password layer is intact
- `enable` prompts for `CrC0ffee!` → privileged access secured
- `show running-config` shows correct hostname, banner, IPs

---

## Key Terms

| Term | Meaning |
|------|---------|
| SVI | Switch Virtual Interface — logical L3 interface for switch management |
| NVRAM | Non-volatile RAM — where startup-config is stored, survives reloads |
| `enable secret` | Hashed privileged EXEC password (MD5/scrypt) |
| `service password-encryption` | Applies type 7 obfuscation to plaintext passwords |
| `ip default-gateway` | Tells a L2 switch where to forward its own off-subnet traffic |
| Banner MOTD | Message displayed before login — legal warning |
| `transport input ssh` | Restricts VTY lines to SSH only — blocks telnet |

---

## Exam Traps

1. **Router interfaces are shutdown by default — switch ports are not.**
   Forgetting `no shutdown` on a router interface is a common config mistake.

2. **`enable secret` and `enable password` are not the same.**
   If both are configured, `enable secret` always wins. Never rely on `enable password` alone.

3. **`service password-encryption` does not secure `enable secret`.**
   The enable secret is already hashed. `service password-encryption` only affects
   plaintext passwords (console, VTY). Type 7 is reversible — don't confuse it with real encryption.

4. **SVIs are administratively down by default.**
   `no shutdown` is required. A switch with an SVI IP but no `no shutdown` will show
   the interface as down and won't respond to pings.

5. **`ip default-gateway` is only for Layer 2 switches.**
   On a router, you use routing (static or dynamic). The `ip default-gateway` command
   is ignored if `ip routing` is enabled.
## Recall Questions

1. What is the purpose of the SVI on a Layer 2 switch, and why does it need `no shutdown`?
2. Why does a switch need `ip default-gateway` if it isn't routing traffic?
3. What is the difference between `enable secret` and `enable password`? Which wins if both are set?
4. What does `service password-encryption` actually protect — and what are its limits?
5. What command saves the running config so it survives a reload?
6. How do you verify whether running-config and startup-config match?
7. Why are router interfaces shutdown by default but switch ports are not?
