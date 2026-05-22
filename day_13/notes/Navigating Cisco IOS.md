
## Simple Explanation
Cisco IOS is the command-line operating system on Cisco switches and routers. You navigate through a hierarchy of modes — each mode unlocks different commands and restricts others. The prompt itself tells you where you are at all times.

## Why It Exists
Without mode separation, any logged-in user could accidentally (or maliciously) change the device's configuration. Mode hierarchy enforces least-privilege access — look around at user level, escalate only when you need to make changes.

## How It Works

### Mode Hierarchy

```
User EXEC        Switch>          Read-only, basic verification
     |
     | enable
     v
Privileged EXEC  Switch#          Full show commands, file ops, copy
     |
     | configure terminal
     v
Global Config    Switch(config)#  Device-wide changes (hostname, passwords)
     |
     | interface <type> <id>
     v
Interface Config Switch(config-if)# Per-port changes (description, shutdown)
```

### Navigation Commands

| Command | Effect |
|---------|--------|
| `enable` | User EXEC → Privileged EXEC |
| `configure terminal` | Privileged EXEC → Global Config |
| `interface <type> <id>` | Global Config → Interface Config |
| `exit` | One level back |
| `end` or `Ctrl+Z` | Any mode → back to Privileged EXEC immediately |
| `disable` | Privileged EXEC → User EXEC |

### Productivity Helpers

| Helper | What it does |
|--------|--------------|
| `?` | Lists all valid commands in current mode |
| `show ?` | Lists all options for show |
| `sh` + `Tab` | Auto-completes to `show` |
| `↑ arrow` | Cycles through command history |
| `show history` | Prints recent command history |

### Key Verification Commands

```
show ip interface brief        → concise table of all interfaces, IPs, status
show running-config            → the live config currently in RAM
show version                   → IOS version, uptime, hardware info
show clock                     → current device clock
```

## Key Terms

| Term | Meaning |
|------|---------|
| IOS | Internetwork Operating System — Cisco's CLI OS |
| User EXEC | Entry-level mode, prompt ends in `>` |
| Privileged EXEC | Elevated mode, prompt ends in `#` |
| Global Config | Device-wide configuration context |
| Interface Config | Per-port configuration context |
| Running config | Live configuration held in RAM |
| Startup config | Saved configuration stored in NVRAM |

## Real-World Connection

In an enterprise network, a junior tech might have credentials that only reach User EXEC — enough to check port status or ping a host, but not to touch the config. The `enable` command (protected by `enable secret`) is the escalation gate.
This is the network equivalent of `sudo` on Linux.

## Security Note — `enable secret` vs `enable password`

```
enable password cisco    ← stored in CLEARTEXT in running-config. Never use.
enable secret cisco      ← hashed with scrypt ($9$) by default. Always use this.
```

When you run `show running-config`, the secret appears as:
```
enable secret 9 $9$t8nO5v6LQab6y.$HB/L0TLo0N63/Fl4N6jWUf5T9UaP4axCD6EPdCGWriY
```
The `$9$` prefix means scrypt — Cisco's strongest hash type. It cannot be reversed.
`$5$` = MD5 (weak, avoid). `$8$` = PBKDF2 (strong). `$9$` = scrypt (strongest).

## Exam Traps

1. **`enable password` vs `enable secret`** — if both are configured, `enable secret`
   always wins. The exam loves testing this. Always set `enable secret`, never rely
   on `enable password`.

2. **`exit` vs `end`** — `exit` goes back one level. `end` / `Ctrl+Z` jumps straight
   to Privileged EXEC from anywhere. Know which one you need in the moment.

3. **Running config vs startup config** — changes take effect immediately in RAM
   (running-config) but are LOST on reboot unless you save with `write memory`
   or `copy running-config startup-config`. Classic trap: you configure something,
   reboot the device, and it's gone.

## Commands

```
! --- Mode navigation ---
Switch> enable                          ! enter privileged EXEC
Switch# configure terminal              ! enter global config
Switch(config)# interface ethernet0/0  ! enter interface config
Switch(config-if)# exit                 ! back to global config
Switch(config)# end                     ! back to privileged EXEC (or Ctrl+Z)
Switch# disable                         ! back to user EXEC

! --- Useful verification ---
Switch# show ip interface brief         ! interface status table
Switch# show running-config             ! live config in RAM
Switch# show version                    ! IOS version and hardware info

! --- Basic config examples ---
Switch(config)# hostname CoreSW-01               ! rename the device
Switch(config)# enable secret StrongPass123!     ! protect privileged EXEC
Switch(config)# interface ethernet0/0
Switch(config-if)# description Link-to-Router   ! label the port
Switch(config-if)# shutdown                      ! disable the port
Switch(config-if)# no shutdown                   ! re-enable the port

! --- Save config ---
Switch# write memory
! or
Switch# copy running-config startup-config
```

## Recall Questions

1. You type a command and get `% Invalid input detected`. You're in Global Config.
   What are two possible reasons the command failed?

2. What is the difference between `exit` and `end` when you're in Interface Config mode?

3. A colleague sets `enable password` and `enable secret` on the same switch.
   Which one is used when someone types `enable`? Why?

4. You make several config changes, then the device is rebooted by the data center team
   without warning. Your changes are gone. What did you forget to do?

5. What does the `$9$` prefix in an enable secret hash tell you about how it's stored?
