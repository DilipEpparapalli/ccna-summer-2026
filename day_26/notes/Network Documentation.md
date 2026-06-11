## Simple Explanation
Document your network as you build it — not at the end. A simple spreadsheet tracking devices, interfaces, IPs, and serial numbers is infinitely more useful than a perfect system that never gets built. When something breaks, you want to be troubleshooting the problem, not discovering your own topology.

## Why It Exists
When a network goes down, time is the enemy. If you haven't documented your infrastructure, you have two problems simultaneously: the outage *and* figuring out what you built. Good documentation collapses that second problem to zero. It also supports warranty claims, support cases, firmware audits, and lifecycle planning — all of which require knowing exactly what you have and where it lives.

## How It Works

### The Minimum Viable Spreadsheet
Start with one sheet. Call it **Devices and IP Addresses**. Don't wait for a perfect system.

| Column | Why It Matters |
|--------|----------------|
| Device Name | Human-readable label for quick identification |
| Model | Required for support cases and replacement ordering |
| Serial Number | Required when opening TAC cases with Cisco |
| Interface | Ties the IP to a specific port |
| MAC Address | Useful for switch port mapping and ARP troubleshooting |
| IP Address | Core of the doc — what lives where |
| Purchase Date | Tracks hardware age and lifecycle |
| In-Service Date | When it went live — differs from purchase |
| Warranty Expiry | Flags end-of-support risk before it bites you |
| Firmware/IOS Version | Critical for catching mismatch issues across similar devices |

### Collecting the Data — Two Key Commands

```
Router# show version
```
- Returns: IOS version, model, serial number, uptime, hardware specs
- Use it to fill: Model, Serial Number, Firmware/IOS Version

```
Router# show ip interface brief
```
- Returns: all interfaces, IP addresses, and up/down status at a glance
- Use it to fill: Interface, IP Address

```
Router# show interface [interface-id]
```
- Returns: hardware (MAC) address for a specific interface
- Use it to fill: MAC Address

### Document Formatting Matters
Clean headers, consistent formatting, and readable column widths aren't cosmetic — they determine whether documentation actually gets maintained. Ugly docs get ignored.

## Key Terms

| Term | Meaning |
|------|---------|
| IPAM | IP Address Management — the discipline of systematically tracking IP assignments, subnets, and device mappings across a network |
| Firmware | The software burned into a device that controls its hardware behavior; version mismatches between identical devices cause subtle, hard-to-diagnose failures |
| Serial Number | Unique hardware identifier required for Cisco TAC support cases and warranty claims |
| In-Service Date | The date a device was put into production — may differ from purchase date |
| Lifecycle | The planned operational lifespan of a device; tracked via purchase/warranty dates to avoid surprise end-of-life failures |

## Real-World Connection
In enterprise environments, a network engineer inheriting an undocumented network is one of the most stressful situations in IT. During an outage, every minute spent discovering topology is a minute not spent fixing the problem. Tools like SolarWinds, Infoblox, and NetBox are enterprise IPAM platforms — but they all start from the same foundation: someone decided to write down what device has what IP. Start with a spreadsheet. Graduate to a platform when the network earns it.

## Professional Habits (No Exam Section — This Is Real-World Discipline)
- **Document while you build** — not at project closeout, not "when you have time"
- **Pull data from the device** — don't rely on memory; `show version` and `show ip interface brief` give you ground truth
- **Standardize MAC address format** — Cisco uses dotted-hex (e.g., `aabb.ccdd.eeff`); normalize to one format across your sheet
- **One useful sheet beats twenty empty ones** — start small, let the doc grow with the network
- **Clean docs get maintained; cluttered docs get abandoned** — formatting is not vanity

## Commands Reference

```
! Identity and version info
show version

! Interface status and IP addresses (quick overview)
show ip interface brief

! Detailed interface info including MAC address
show interface [interface-id]

! Example: grab MAC address on a specific interface
show interface GigabitEthernet0/0
```

## Recall Questions
1. What is the single biggest risk of waiting until project closeout to document a network?
2. What does `show version` give you that `show ip interface brief` does not?
3. What is IPAM, and at what point does a simple spreadsheet stop being sufficient?
4. Why does firmware version belong in a network documentation spreadsheet?
5. A support call with Cisco TAC opens. What two pieces of device information do they ask for first?
