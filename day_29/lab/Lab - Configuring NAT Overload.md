
## Mission Briefing

It's the final NAT drill in Castle Rysen's bunker: the caf floor is overflowing with random tablets, scanners, and IoT gizmos that can't all get a dedicated public IP. Jeremy's directive is simple-collapse every private address behind the ISP edge using NAT overload so thousands of devices can sip the same upstream address. You'll disable the old dynamic pool mapping, wire the access list to the WAN interface, bring the links back up, and prove the entire caf can burst through a single public identity on demand.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Deactivate the old dynamic NAT pool mapping so it does not conflict with overload.
- Reuse the caf access list while redirecting translations to the ISP-facing interface with overload enabled.
- Verify that all caf hosts share the WAN interface address while maintaining successful outbound connectivity.

---

## Task 0 - Disable Dynamic NAT

Begin at `Cafe-Rtr#` and remove the existing dynamic mapping so nothing conflicts with the overload configuration.

**Steps:**

1. Display the current running configuration to spot the command that ties access list 1 to pool `Cafe-Public`.
2. Enter configuration mode and delete that dynamic rule.
3. Confirm the NAT translation table empties once the rule is gone.

---

## Task 1 - Freeze the Interfaces

Prevent new translations from forming by disabling both caf router interfaces before you retool NAT.

**Steps:**

1. From `Cafe-Rtr#`, enter configuration mode and shut down Ethernet0/0.
2. Shut down Ethernet0/1 so no traffic traverses the WAN while you reconfigure NAT.
3. Exit configuration mode once both interfaces report an administratively down state.

---

## Task 2 - Enable NAT Overload

Rewire the router so access list 1 overloads onto the ISP-facing interface instead of using the public pool.

**Steps:**

1. Enter configuration mode on `Cafe-Rtr`.
2. Apply the NAT rule that maps access list 1 to interface Ethernet0/1 with the overload keyword.
3. Ensure Ethernet0/0 retains `ip nat inside` and Ethernet0/1 retains `ip nat outside` for proper directionality.

---

## Task 3 - Restore Connectivity

Bring the caf links back online and let the hosts resume testing through the overloaded address.

**Steps:**

1. From configuration mode, enable Ethernet0/0 to restore LAN access.
2. Enable Ethernet0/1 to reestablish the ISP handoff.
3. Confirm the interfaces return to an up/up state before proceeding.

---

## Task 4 - Inspect Overloaded Translations

Verify that multiple caf hosts now share the single WAN interface address through port-based NAT overload.

**Steps:**

1. From `PC1` and `PC2`, sign in with username `cisco` and password `cisco`, then run five-packet connectivity tests toward 1.1.1.1 to generate traffic.
2. On `Cafe-Rtr#`, inspect the NAT translation table to confirm both inside hosts map to 216.0.5.2 with unique ports.
3. Review NAT statistics to observe how many active translations are sharing the interface address.

---

## Completion Check

- The dynamic pool rule is removed and replaced with `ip nat inside source list 1 interface Ethernet0/1 overload`.
- Ethernet0/0 and Ethernet0/1 operate as the inside/outside NAT boundaries with the links restored to up/up status.
- `show ip nat translations` displays both caf hosts sharing 216.0.5.2 via unique port numbers, proving NAT overload is live.

 copy on select