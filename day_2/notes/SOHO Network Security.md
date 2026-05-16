
## Simple Explanation

A home or small office network runs on one device that handles routing, switching, firewall, and wireless all at once. This convenience creates serious security risks that require deliberate hardening to address. The same security mindset used in enterprise networks applies here — just at a smaller scale.

---

## What is SOHO?

**Small Office Home Office** — a network where one consumer device does everything:

- Router (connects to internet)
- Switch (connects wired devices)
- Wireless Access Point (WiFi)
- Firewall (basic traffic filtering)

```
SOHO:
[All-in-one Router] ← does everything
   WiFi devices, wired devices, IoT, guests
   all on one flat network by default

Enterprise:
[Firewall] → [Core] → [Distribution] → [Access]
   Dedicated devices, segmented networks,
   redundancy, IDS/IPS, proper VLAN design
```

Your home network is effectively a tiny branch office — especially if you work remotely. Same risks, far fewer defenses.

---

## The Four Threat Areas

```
1. Internet      → your network   (external attacks)
2. Your devices  → your network   (internal IoT threats)
3. Wireless      → your network   (over-the-air attacks)
4. Home          → your company   (remote work exposure)
```

---

## Threat 1 — Internet to Your Network (External Attacks)

Your ISP assigns your router a public IP address. Anyone who knows it can probe it for weaknesses.

How attackers find you:

- Scan ranges of public IP addresses automatically
- Find open ports → look up known exploits → attack

Tool attackers use: **Nmap** (free, open source network scanner)

```bash
nmap -sT <public-ip>             # scan for open ports
nmap --script vuln <public-ip>   # scan for known vulnerabilities
```

### Hardening Checklist

|Action|Why|
|---|---|
|Enable firewall|First line of defense|
|Disable port forwarding|Every open port is a hole in your network|
|Disable remote management|Removes external login attack surface|
|Change default credentials|admin/admin is the first thing attackers try|
|Update firmware|Patches known vulnerabilities in router software|
|Disable WAN ping response|Stops router announcing itself to scanners|

**Critical:** Port forwarding is a vulnerability — not a security tool. Every port forward is a deliberate hole punched through your firewall. Close all of them unless you have a specific, understood reason for each one.

---

## Threat 2 — IoT Devices Inside Your Network (Internal Threat)

This is the most underestimated threat vector. Compromised devices inside your network can attack other devices without ever triggering inbound firewall rules.

### The Mechanism

```
[Compromised IoT] ──outbound──► [Attacker Server]
[Attacker Server] ──response──► [Compromised IoT]
                    ↑ firewall allows this —
                    the device initiated the connection

[Compromised IoT] ──lateral──► [Laptop / NAS / Files]
                    ↑ all on the same flat network
```

Your firewall blocks unsolicited **inbound** connections. It does **not** block outbound-initiated traffic. A compromised device uses outbound connections as a backdoor, then moves laterally across your flat LAN to reach valuable devices.

### The Fix — Network Segmentation

Split devices into separate VLANs so a compromised device has no path to your trusted devices:

```
VLAN 10 — Trusted devices (laptop, phone, tablet)
VLAN 20 — IoT devices (smart bulbs, TV, voice assistants)
VLAN 30 — Guest devices

Firewall rule: VLANs cannot communicate with each other
Result: compromised IoT has no path to your laptop
```

**Client Isolation** — an additional layer. Prevents devices on the **same** VLAN from talking to each other directly. Your smart bulb cannot probe your smart TV even if they share a VLAN.

---

## Threat 3 — Wireless Network (Over-the-Air Attacks)

**Wardriving** — attackers scan for WiFi networks from a vehicle, looking for vulnerable targets.

### Why Default SSIDs Are Dangerous

A default SSID like "TP-Link_A3F2" tells an attacker:

- Exact router brand and model
- Which known vulnerabilities to research
- Which default credentials to try

### Wireless Hardening

|Setting|What to Do|
|---|---|
|Encryption|WPA2 minimum — WPA3 if supported|
|SSID|Change to something that reveals nothing about your hardware|
|Password|Long, random, 20+ characters|
|Guest network|Separate isolated VLAN — never mix with main network|
|Client isolation|Enable on guest and IoT networks|

---

## Threat 4 — Home to Company (Remote Work Exposure)

Working from home makes your home network part of your company's attack surface. Unprotected traffic over the public internet exposes company data to interception.

Solution: **VPN (Virtual Private Network)** — creates an encrypted tunnel between your home and the company network.

### Two VPN Approaches

**Option 1 — Remote Access VPN**

Software installed on your laptop (e.g. Cisco AnyConnect, OpenVPN). You initiate the connection manually when needed.

```
[Your Laptop] ══ encrypted tunnel ══► [Company HQ]
```

**Option 2 — Site-to-Site VPN**

A physical firewall appliance provided and managed by your company. Sits inline on your home network. Always-on encrypted tunnel. Company controls both ends.

```
[Home devices] → [Company Firewall Appliance] ══ tunnel ══► [HQ]
                  ↑ company-managed hardware
```

Site-to-site is more secure — the company controls the hardware and security policy at your location, not just their end.

---

## When Basic Gear Isn't Enough

Some security features — VLANs, client isolation, IDS/IPS — require hardware that goes beyond a basic consumer router.

|Option|What It Provides|
|---|---|
|DD-WRT custom firmware|VLANs, better firewall on existing supported hardware|
|Cisco gear (eBay)|Full enterprise features, excellent for home lab|
|UniFi (prosumer)|VLAN support, IDS/IPS, traffic visibility, VPN|

### IDS vs IPS

```
IDS — Intrusion Detection System
      Monitors traffic, detects threats, alerts you.
      Passive — tells you something is wrong.

IPS — Intrusion Prevention System
      Monitors traffic, detects threats, blocks them.
      Active — takes action automatically.
```

---

## Key Terms

|Term|Meaning|
|---|---|
|SOHO|Small Office Home Office network|
|Public IP|Address your ISP assigns your router — visible to the internet|
|Port forwarding|Allowing external traffic through your firewall to an internal device|
|Wardriving|Scanning for WiFi networks from a moving vehicle|
|SSID|WiFi network name|
|WPA2/WPA3|WiFi encryption standards (WPA2 minimum, WPA3 preferred)|
|Client isolation|Prevents devices on the same network from talking directly to each other|
|Segmentation|Splitting devices into separate VLANs to contain threats|
|IDS|Intrusion Detection System — passive monitoring and alerting|
|IPS|Intrusion Prevention System — active detection and blocking|
|Remote access VPN|Software-based VPN on the end device|
|Site-to-site VPN|Appliance-based permanent encrypted tunnel between two locations|
|Nmap|Free open-source network scanner used for port scanning and vulnerability detection|
|Lateral movement|Attacker moving from one compromised device to other devices on the same network|

---

## Exam Traps

1. **Port forwarding is a vulnerability — not a security feature.** The fix is to disable it, not configure it more carefully.
2. **Firewall blocks unsolicited inbound connections — not outbound.** IoT threats exploit outbound-initiated return traffic to bypass firewall rules.
3. **MPLS uses label-based logical separation — not encryption.** Site-to-site VPN uses actual encryption. Do not confuse the two.
4. **WPA2 is the minimum acceptable wireless standard.** WEP is completely broken and must never be used.
5. **Changing your SSID is a security measure.** Default SSIDs reveal your hardware model and invite targeted attacks.
6. **Remote access VPN ≠ site-to-site VPN.** Remote access runs software on the end device. Site-to-site uses a dedicated hardware appliance and is always on.

---

## Recall Questions

1. What four areas represent the main threat vectors for a SOHO network?
2. Your firewall is enabled. How does a compromised smart bulb still threaten your laptop? Explain the mechanism.
3. What is wardriving and why does a default SSID make it worse?
4. What is the difference between IDS and IPS?
5. What is the difference between remote access VPN and site-to-site VPN? Which gives the company more control and why?
6. Why is port forwarding a security risk rather than a security feature?
7. What is client isolation and when should you use it?
8. You want to connect two switches together. Your friend says use a straight-through cable. Is that right? Why or why not?