## Mission Briefing

Training day inside Castle Rysen's command bunker: Jeremy wants you to bolt a simulated internet onto the caf network so the recruits can practice NAT without needing a real ISP. You'll stand up an ISP edge router, secure it with bunker-standard credentials, and light up a WAN link that feeds the caf's default route. Once the scaffolding is in place, you'll emulate public DNS services so every future drill has somewhere "out there" to reach.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Deploy and interconnect an ISP router that represents the public internet for upcoming NAT exercises.
- Apply Castle Rysen's baseline identity and access controls to the new router.
- Configure the emulated ISP with WAN and loopback addressing so the caf edge can verify external reachability.

---

## Task 0 - Roll Out the ISP Router

In the bunker sim, power up the pre-staged internet router and confirm the WAN link is ready. Start at the ISP router user EXEC prompt once the device settles.

**Steps:**

1. Open the console for `ISP-Rtr` and wait for the user EXEC prompt. The live router may start as `inserthostname-here>` before you set the hostname.
2. Run `show ip interface brief` to spot `Ethernet0/0` and note its current (administratively down) status before you configure it in Task 2.
3. Verify the existing topology link between `ISP-Rtr Ethernet0/0` and `Cafe-Rtr Ethernet0/1`, confirming the cable is already in place.

---

## Task 1 - Lock Down the ISP Router

Harden the new router with Castle Rysen's default identification and access controls so only authorized engineers can touch it.

**Steps:**

1. Escalate from user EXEC to the privileged prompt on `ISP-Rtr`.
2. Enter global configuration mode and set the device name to `ISP-Rtr`.
3. Apply console and remote access passwords of `Cisco`, require logins, and secure the enable secret with the same value.

---

## Task 2 - Provision the WAN Edge

Give the ISP handoff its public addressing so `Cafe-Rtr` knows exactly where to forward default traffic.

**Steps:**

1. Configure `ISP-Rtr` Ethernet0/0 with the address 216.0.5.1/30 to match the caf's upstream circuit.
2. Activate the interface so the physical and protocol states report operational.
3. From the privileged prompt, send echo tests to 216.0.5.2 on `Cafe-Rtr` and confirm clean replies.

---

## Task 3 - Simulate Public DNS Targets

Create loopbacks on the ISP router that behave like real-world DNS services so the caf network can prove outbound paths.

**Steps:**

1. Build Loopback1 with the address 1.1.1.1/32 and Loopback2 with 8.8.8.8/32 on `ISP-Rtr`.
2. Note in your field log that Loopback1 represents Cloudflare DNS and Loopback2 represents Google DNS.
3. Check the interface status summary to confirm both loopbacks appear up and reachable locally.

---

## Task 4 - Validate Caf Reachability

Prove the caf edge can reach the simulated internet while recognizing why the PCs still need NAT to succeed.

**Steps:**

1. From `Cafe-Rtr`, use the existing default route to send echo probes toward 1.1.1.1 and 8.8.8.8, watching for successful responses.
2. From the `PC1` console tab, log in with `cisco` / `cisco` and attempt the same probes; observe that replies fail because `ISP-Rtr` has no route back to 192.168.1.0/24 yet.
3. Record the return-path gap as the reason the next lesson layers on NAT before allowing the caf floor real internet access.

---

## Completion Check

- `ISP-Rtr` Ethernet0/0 runs 216.0.5.1/30 in an up/up state alongside Loopback1 (1.1.1.1/32) and Loopback2 (8.8.8.8/32).
- `Cafe-Rtr` reaches both loopbacks using its default route toward 216.0.5.1, proving the simulated internet path works.
- `PC1` echo tests fail exactly because the ISP edge lacks a route to 192.168.1.0/24, setting the stage for NAT in the next drill.