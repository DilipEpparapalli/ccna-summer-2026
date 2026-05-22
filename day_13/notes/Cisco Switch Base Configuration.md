
## Simple Explanation
A base configuration is a standard set of commands applied to every Cisco device
before it goes into production. It gives the device an identity, protects access at
every entry point, enables remote management, and documents its ports. Once built,
it becomes a deployment template — paste, tweak three lines, done.

## Why It Exists
A factory-fresh Cisco switch has no hostname, no passwords, no management IP,
and no port labels. It works at Layer 2 out of the box, but it is unmanageable,
insecure, and completely anonymous. Base config turns a box into a known,
secured, addressable network device.

## The Base Config Checklist (in order)

```
1. Hostname
2. Banner MOTD
3. Enable secret       ← protects privileged EXEC
4. Console password    ← protects physical access
5. VTY password        ← protects remote (SSH/Telnet) access
6. service password-encryption
7. Management IP (interface vlan 1)
8. Port descriptions
9. Save the config
```

## How It Works — Command by Command

### 1. Hostname
```
Switch(config)# hostname cafe01-sw01
```
Sets the device's identity. Use a naming convention that encodes location and role.
Good: `cafe01-sw01`, `dc01-core-sw01`
Bad: `Harvey`, `MySwitch`, `Switch1`

### 2. Banner MOTD
```
Switch(config)# banner motd #
Authorized access only. Disconnect immediately if not authorized.
#
```
The `#` is the delimiter — it marks where the message starts and ends.
Any character can be a delimiter as long as it doesn't appear inside the message.
This banner displays before the login prompt. Required in enterprise environments
for legal standing if unauthorized access occurs.

### 3. Enable Secret
```
Switch(config)# enable secret StrongPassword123!
```
Protects privileged EXEC mode (`Switch#`). Always use `enable secret`, never
`enable password`. If both are set, `enable secret` always wins.

### 4. Console Line Security
```
Switch(config)# line console 0
Switch(config-line)# password cisco123
Switch(config-line)# login
```
`password` sets the credential. `login` tells IOS to actually enforce it.
Without `login`, the password is configured but never asked for. Classic trap.

### 5. VTY Line Security (remote access)
```
Switch(config)# line vty 0 4
Switch(config-line)# password cisco123
Switch(config-line)# login
```
VTY = Virtual Terminal Lines. These are the SSH/Telnet sessions used to
manage the device remotely. `0 4` means lines 0 through 4 (5 simultaneous sessions).
Some switches have `vty 0 15` (16 sessions total).

### 6. Service Password Encryption
```
Switch(config)# service password-encryption
```
Applies Type 7 (Vigenère cipher) obfuscation to all plaintext passwords in the
config — console, VTY, etc. Protects against casual shoulder surfing only.
Does NOT affect `enable secret`, which is already hashed.

### 7. Management IP on VLAN 1
```
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.168.1.10 255.255.255.0
Switch(config-if)# no shutdown
```
This gives the switch its own IP address so it can be reached remotely.
This is NOT routing. This is the switch's management identity on the network.
VLAN 1 is the default management VLAN. By default, the interface is shutdown —
`no shutdown` is required.

Don't forget the default gateway if you need to manage from another subnet:
```
Switch(config)# ip default-gateway 192.168.1.1
```

### 8. Port Descriptions
```
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# description Connection to cafe01-sw02 fa0/1
```
Labels the port. Six months from now, this is the difference between knowing
what's plugged in and tracing cables by hand at midnight.

### 9. Save the Config
```
Switch# write memory
! or the explicit form:
Switch# copy running-config startup-config
```
Changes live in RAM (running-config). RAM is wiped on reboot.
NVRAM holds startup-config. If you don't save, changes vanish on power loss.

---

## Password Type Reference

| Type | Algorithm | How to spot it | Safe? |
|------|-----------|----------------|-------|
| `0` | Plaintext | `password 0 cisco` | ❌ Never use |
| `7` | Vigenère cipher (reversible) | `password 7 0822455D0A16` | ❌ Shoulder surfing only |
| `5` | MD5 hash | `secret 5 $1$mERr$...` | ⚠️ Weak, legacy IOS |
| `8` | PBKDF2-SHA256 | `secret 8 $8$...` | ✅ Strong |
| `9` | scrypt | `secret 9 $9$...` | ✅ Strongest |

`service password-encryption` → produces Type 7 on line passwords.
`enable secret` → produces Type 9 on modern IOS, Type 5 on older IOS (Packet Tracer).

---

## Key Terms

| Term | Meaning |
|------|---------|
| MOTD | Message of the Day — displayed before login prompt |
| Console line | Physical port for direct cable access (`line con 0`) |
| VTY line | Virtual terminal — SSH/Telnet remote access sessions |
| `login` | Command that enforces password checking on a line |
| `no` prefix | Removes/undoes a command. `no shutdown` = enable port. `no login` = remove login requirement |
| Running config | Live config in RAM — lost on reboot if not saved |
| Startup config | Saved config in NVRAM — survives reboots |
| Interface VLAN 1 | Virtual interface giving the switch a management IP |
| `ip default-gateway` | Tells the switch where to send traffic destined outside its subnet |

---

## The `no` Command — Understand This Deeply

`no` in IOS means **undo this configuration line**. It does not mean "deny" or
"block" in everyday English terms.

```
no shutdown    → removes the shutdown state → port comes UP
no login       → removes the login requirement → no password asked
no ip address  → removes the IP address from the interface
```

This trips up beginners constantly. `no login` does not block logins — it removes
the login enforcement. The prompt does NOT change when you're in line config mode;
look at the command output for confirmation.

---

## Real-World Connection

A network engineer deploying 20 access switches for a new office floor builds
the base config once, saves it as a text file, and uses it as a template. Per-device
changes: hostname, management IP, port descriptions. Everything else is pasted
identically. This is how real deployment works — repeatable, auditable, consistent.

---

## Exam Traps

1. **`login` is not optional.** Setting a password on a line without typing `login`
   means the password is never checked. The exam will absolutely test this.

2. **`enable password` vs `enable secret`** — if both are configured, `enable secret`
   wins every time. Type `7` from `service password-encryption` does NOT affect
   `enable secret`. They are independent.

3. **Management IP needs `no shutdown`.** Interface VLAN 1 is administratively
   down by default. Forgetting `no shutdown` means the switch is unreachable
   remotely — the IP is set but the interface is off.

4. **Default gateway is required for cross-subnet management.** If your management
   workstation is on a different subnet, the switch needs `ip default-gateway` to
   reply to you. Without it, pings time out from the far side.

5. **`write memory` vs `copy run start`** — both save the config. `wr` is the
   old-school shortcut, still works on modern IOS. Know both.

---

## Full Base Config Template

```
! ============================================================
! Cisco Switch Base Configuration Template
! ============================================================

! --- Identity ---
hostname <DEVICE-NAME>

! --- Legal warning ---
banner motd #
Authorized access only. Disconnect immediately if not authorized.
#

! --- Privileged EXEC protection ---
enable secret <STRONG-PASSWORD>

! --- Console access ---
line console 0
 password <CONSOLE-PASSWORD>
 login
 logging synchronous

! --- Remote access (SSH/Telnet) ---
line vty 0 4
 password <VTY-PASSWORD>
 login

! --- Encrypt plaintext passwords in config ---
service password-encryption

! --- Management IP ---
interface vlan 1
 ip address <IP-ADDRESS> <SUBNET-MASK>
 no shutdown

! --- Default gateway (if managed from another subnet) ---
ip default-gateway <GATEWAY-IP>

! --- Port descriptions (repeat per port) ---
interface fastEthernet 0/1
 description <WHAT IS PLUGGED IN HERE>

! --- Save ---
! (run from privileged EXEC, not config mode)
! write memory
! ============================================================
```

---

## Recall Questions

1. You configure a password on `line console 0` but when you reconnect, no
   password is asked. What did you forget?

2. A colleague claims `service password-encryption` makes their switch secure.
   What's wrong with that statement?

3. You assign an IP address to `interface vlan 1` and can't ping the switch
   from across the network. Name two things to check.

4. What is the difference between `enable password` and `enable secret`?
   If both are configured, which one takes effect?

5. You make 45 minutes of configuration changes on a production switch.
   Someone trips on the power cable. What happens, and how do you prevent it?

6. What does `no shutdown` actually do? Why is the word "no" confusing here?
