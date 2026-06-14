## Mission Briefing

Castle Rysen's caf floor just swapped from one-off static mappings to a flood of random barista tablets hitting the network. Jeremy's assignment: rip out the one-to-one NAT entries and replace them with an address pool that any workstation can borrow from. You'll define the inside network with an access list, carve out a block of public IPs, stitch the pieces together, and then watch the translations rotate as multiple caf devices test the connection.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Remove the legacy static NAT bindings so dynamic translations can take over cleanly.
- Describe the inside address space with a standard access list and map it to a public pool.
- Validate that caf hosts consume pool addresses during outbound tests and observe pool limits.

---

## Task 0 - Clear the Static Translations

Start at `Cafe-Rtr#` to remove any leftover one-to-one mappings so the dynamic workflow has a clean slate.

**Steps:**

1. Display the running configuration and identify the lines that bind 192.168.1.50 and 192.168.1.51 to public addresses.
2. Enter configuration mode and delete each static translation along with any unused public IP references.
3. Confirm that the NAT table no longer lists static entries from the previous drill.

---

## Task 1 - Define the Inside Address List

Document the caf LAN addresses you intend to translate by creating a standard access list.

**Steps:**

1. From the privileged prompt, enter configuration mode on `Cafe-Rtr`.
2. Build a numbered standard access list that matches every host within 192.168.1.0/24.
3. Display the list so you can confirm the router is watching that entire subnet.

---

## Task 2 - Carve the Public Address Pool

Reserve a block of public IPs for dynamic use so each caf device can borrow one as it breaks out to the internet.

**Steps:**

1. Re-enter configuration mode on `Cafe-Rtr`.
2. Create an IP NAT pool named `Cafe-Public` that spans 216.0.5.50 through 216.0.5.100 with a /24-style mask.
3. Double-check the pool definition to make sure the addresses and mask are correct.

---

## Task 3 - Tie the Pieces Together

Connect the inside list to the public pool so dynamic translations can activate when hosts generate traffic.

**Steps:**

1. Enter configuration mode on `Cafe-Rtr` and apply the NAT rule that associates access list 1 with pool `Cafe-Public`.
2. Ensure the router uses `ip nat inside` on Ethernet0/0 and `ip nat outside` on Ethernet0/1, updating them if necessary.
3. Exit configuration mode and prepare to monitor the translation table as clients test connectivity.

---

## Task 4 - Observe Dynamic Allocation

Kick off reachability tests from both caf hosts and watch the translations consume addresses from the pool.

**Steps:**

1. From `PC1`, sign in with username `cisco` and password `cisco`, then run a five-packet ping toward 1.1.1.1 to generate outbound traffic.
2. On `PC2`, repeat the login with `cisco/cisco` and fire the same five-packet test so two devices are active simultaneously.
3. Monitor `Cafe-Rtr` for live translations and note how each inside host temporarily consumes a pool address.

---

## Completion Check

- Static NAT entries are gone, and a dynamic rule now references access list 1 and pool `Cafe-Public`.
- `show running-config | include ip nat pool` displays the 216.0.5.50-216.0.5.100 range that fuels translations.
- `show ip nat translations` reveals caf hosts consuming pool addresses as they reach 1.1.1.1, proving dynamic NAT is live.