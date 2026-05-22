
## What I Did
Configured a bare Cisco switch from scratch — hostname, banners, layered passwords, a management SVI, and saved the config. This is the standard hardening template every switch gets before it touches a production network.

**Device:** `Cafe-01-SW1` (real Cisco switch, IOS 17.16)  
**Management IP assigned:** `192.168.10.10/24` on VLAN 1 SVI

---

## What I Learned

### Hostname
First thing — rename the device so you know what you're looking at in the prompt and in any network management tool.

```
Switch> enable
Switch# configure terminal
Switch(config)# hostname Cafe-01-SW1
Cafe-01-SW1(config)#
```

The prompt changes immediately. That's your confirmation.

---

### MOTD Banner
The banner displays before the login prompt. It's not just cosmetic — a warning banner is legally required in most organizations. Without it, unauthorized access is harder to prosecute in court because there was no posted notice.

```
Cafe-01-SW1(config)# banner motd #
Unauthorized access ends badly. Authorized engineers only.
#
```

The `#` is the delimiter — any character works, as long as it doesn't appear in your banner text.

---

### Securing Privileged EXEC — `enable secret` vs `enable password`

This is where I learned a critical distinction:

|                   | `enable password`                           | `enable secret`             |
| ----------------- | ------------------------------------------- | --------------------------- |
| Storage           | Plaintext (or Type 7 if encryption enabled) | Type 9 Scrypt hash — always |
| Security          | Weak                                        | Strong                      |
| Wins if both set? | No                                          | **Yes — always**            |

```
Cafe-01-SW1(config)# enable secret C4stleRysen!
```

In `show running-config` this appears as:
```
enable secret 9 $9$VO7ZTasyeOKLi.$UOzuP3x...
```

The `9` means Scrypt hash. That's a real one-way hash — not reversible.

---

### Console Line Password
Protects physical walk-up access. Anyone who plugs a console cable in has to authenticate.

```
Cafe-01-SW1(config)# line console 0
Cafe-01-SW1(config-line)# password VaultAccess
Cafe-01-SW1(config-line)# login
```

`login` is required — without it, the password is set but never actually prompted.

---

### VTY Lines — Remote Access via SSH
VTY (Virtual Teletype) lines are the virtual ports that handle remote sessions. There are 5 by default (0–4), meaning 5 simultaneous remote sessions.

```
Cafe-01-SW1(config)# line vty 0 4
Cafe-01-SW1(config-line)# password ShelterAccess
Cafe-01-SW1(config-line)# login
Cafe-01-SW1(config-line)# transport input ssh
```

`transport input ssh` blocks Telnet — remote access is SSH only.

> **Note:** `transport input ssh` restricts the protocol, but SSH won't actually work yet until RSA keys are generated and a domain name is set. That's the next lab.

---

### Password Encryption Service
Without this, console and VTY passwords sit in the running config as plaintext. With it, they get Type 7 obfuscation.

```
Cafe-01-SW1(config)# service password-encryption
```

After enabling, `show running-config` shows:
```
password 7 122F04021E1F2D07292E373B
```

**Important:** Type 7 is NOT real encryption. It's a reversible XOR scramble — crackable in seconds with free online tools. It only stops casual shoulder-surfing of the config file. `enable secret` is already a proper hash and is unaffected by this service.

---

### Management SVI — Why Not Just Use an Ethernet Port?

Layer 2 switch ports don't do IP — they only switch frames. To give the switch an IP address for remote management, I had to use an SVI (Switch Virtual Interface): a logical Layer 3 interface tied to a VLAN.

VLAN 1 is the default VLAN — all ports are members by default — so assigning the IP here makes the whole switch reachable.

```
Cafe-01-SW1(config)# interface vlan 1
Cafe-01-SW1(config-if)# ip address 192.168.10.10 255.255.255.0
Cafe-01-SW1(config-if)# no shutdown
```

Verified with:
```
Cafe-01-SW1# show ip interface brief

Interface     IP-Address      OK? Method Status   Protocol
Ethernet0/0   unassigned      YES unset  up        up
Vlan1         192.168.10.10   YES manual up        up
```

Both Status and Protocol showing `up` — the SVI is live.

---

### Interface Description
Tagged the uplink port so any engineer reading the config knows where it connects without having to trace cables.

```
Cafe-01-SW1(config)# interface ethernet0/0
Cafe-01-SW1(config-if)# description "Uplink-to-Core-Distribution"
```

---

### Saving the Config
Running config lives in RAM — a reload wipes it. Saving writes it to NVRAM so it survives power cycles.

```
Cafe-01-SW1# copy running-config startup-config
! or legacy shorthand — same result:
Cafe-01-SW1# write memory
```

---

## Key Terms

| Term           | Meaning                                                        |
| -------------- | -------------------------------------------------------------- |
| Running Config | Active config in RAM — lost on reload                          |
| Startup Config | Saved config in NVRAM — survives power loss                    |
| SVI            | Switch Virtual Interface — logical Layer 3 interface on a VLAN |
| VTY            | Virtual terminal lines for remote (SSH/Telnet) sessions        |
| Type 7         | Cisco's reversible XOR obfuscation — not real security         |
| Type 9         | Scrypt hash used by `enable secret` — cryptographically secure |
| MOTD           | Message of the Day — displayed before login prompt             |
| NVRAM          | Non-Volatile RAM — persists through power loss                 |

---

## Exam Traps

1. **`enable secret` always beats `enable password`** — If both are configured, `enable password` is completely ignored. The exam shows configs with both present and asks which one you use.

2. **`service password-encryption` ≠ real security** — Type 7 is obfuscation, not encryption. It does nothing to `enable secret` (already hashed). Don't confuse the two.

3. **Can't IP a Layer 2 port directly** — The IP must go on the SVI (`interface vlan X`), not on a physical switch port. The exam tests whether you know *why*.

4. **`login` is required on line config** — Setting a password without `login` means the password is never prompted. Easy to miss, easy to test.

5. **`copy run start` vs `write memory`** — Same result. Unsaved changes after the last save are lost on reload.

---

## Recall Questions

1. You see both `enable secret` and `enable password` in a config. Which one grants access — and what does IOS do with the other?
2. Why can't you run `ip address 192.168.10.10 255.255.255.0` directly on `interface ethernet0/0` of a Layer 2 switch?
3. You enable `service password-encryption`. Does that make your `enable secret` more secure? Why or why not?
4. You configure `transport input ssh` on the VTY lines. A colleague tries to SSH in and gets refused. What's the likely missing piece?
5. An engineer makes changes, runs `write memory`, makes more changes, then the switch loses power. What config comes back up?
