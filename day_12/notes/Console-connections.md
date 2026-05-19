
## Simple Explanation
A console connection is a direct physical link between your laptop and a network device's console port. It gives you command-line access to the device without needing any network configuration to exist first.

## Why It Exists
New Cisco devices have no IP address, no SSH, no remote access of any kind. The console connection bypasses the network entirely and talks directly to the device's CPU — it's the only way in before the network is configured. It's also your emergency lifeline when remote access breaks in production.

## How It Works
1. Connect a console cable from your laptop's USB port to the device's RJ45 console port
2. Open a terminal program (PuTTY on Windows)
3. Select: Connection Type = **Serial**, Speed = **9600 baud**
4. Open the session — press **Enter** if the screen is blank
5. You land at the CLI prompt (no password on a fresh device)

## Key Terms
| Term | Meaning |
|------|---------|
| Console port | Dedicated management port on Cisco devices — not for data traffic |
| Baud rate | Serial connection speed — 9600 is the Cisco default |
| Out-of-band management | Managing a device via a path separate from the production network |
| COM port | Windows name for a serial port (e.g. COM3) |
| PuTTY | Free terminal emulator used to open serial/SSH sessions on Windows |

## Real-World Connection
When a misconfigured device loses its IP or SSH breaks, remote access is dead. The console cable is your emergency access — the equivalent of plugging a keyboard and monitor directly into the device. In enterprise environments, some racks have permanent **console servers** so engineers can always get in remotely via a separate out-of-band management network.

## Exam Traps
1. **Fresh device = no password prompt** — you land straight at the CLI. Don't expect a login on initial setup.
2. **Console isn't router-only** — switches, APs, and all Cisco devices use console for initial config and emergency access.
3. **Serial ≠ Ethernet** — the console port may look like RJ45 but it is NOT a network port. Don't plug it into your switch.

## Commands (if applicable)
```
! Nothing to configure for console access itself
! But once you're in, you'll see one of these prompts:

Router>       ! User EXEC mode — limited, read-only commands
Router#       ! Privileged EXEC mode — after typing 'enable'
Router(config)#  ! Global config mode — after typing 'conf t'
```

## Recall Questions
1. Why can't you SSH into a brand new router straight out of the box?
2. What are the correct PuTTY settings for a Cisco console connection?
3. What do you do if you connect via console and the screen is blank?
4. What's the difference between console access and out-of-band management?
5. Does a fresh Cisco device prompt for a password on first console login?
