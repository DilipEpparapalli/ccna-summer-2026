## Mission Briefing

Coffee House Beta and the Fallout Shelter can see their own LANs, but neither router knows how to reach the other's network yet. Jeremy wants you to stitch the sites together with static routes so the baristas and the shelter crew can talk before we introduce more advanced routing protocols. You'll build the two routes, verify connectivity end to end, and document the results for the rollout log.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Inspect both routers and confirm their WAN and LAN interfaces are up with the correct addressing.
- Configure a static route on `Cafe-RT1` so it can reach the Fallout LAN through the point-to-point link.
- Configure a mirror static route on `Fallout-RT1`, then validate full reachability between the LAN hosts.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`
- The routers may place you directly at a privileged `#` prompt after login. If you land at `>`, use `enable` and the privileged password.

---

## Task 0 - Verify the Baseline

Confirm each router already knows about its directly connected networks before you add static routes.

**Steps:**

- On `Cafe-RT1`, review `show ip interface brief` and `show ip route` to ensure Ethernet0/0 (`192.168.1.1/24`) and Ethernet0/1 (`192.168.2.1/30`) are up.
- On `Fallout-RT1`, repeat the check for Ethernet0/0 (`192.168.3.1/24`) and Ethernet0/1 (`192.168.2.2/30`).
- Document any mismatches or shutdown interfaces so you can fix them before you proceed.

---

## Task 1 - Add the Static Route on `Cafe-RT1`

Tell the Coffee House router how to reach the Fallout LAN.

**Steps:**

- Identify the remote network (`192.168.3.0/24`) and confirm the next-hop address on the point-to-point link (`192.168.2.2`).
- Configure a static route on `Cafe-RT1` pointing `192.168.3.0/24` toward `192.168.2.2`.
- Verify the route appears in `show ip route` with an "S" code and the correct next-hop.

---

## Task 2 - Add the Static Route on `Fallout-RT1`

Teach the Fallout router how to get back to the Coffee House LAN.

**Steps:**

- Identify the remote network (`192.168.1.0/24`) and the next-hop address (`192.168.2.1`).
- Configure a static route on `Fallout-RT1` pointing `192.168.1.0/24` toward `192.168.2.1`.
- Confirm the static entry appears in the routing table with the expected next-hop.

---

## Task 3 - Validate End-to-End Connectivity

Test across the LANs to prove the static routes work both ways.

**Steps:**

- From `Cafe-RT1`, ping the Fallout server (`192.168.3.100`) to verify traffic can leave the Coffee House side. The first ping may drop one probe while ARP resolves.
- Open the `Fallout-SRV` console tab and ping the Coffee House LAN host (`192.168.1.50`) with `ping -c 5 192.168.1.50` to confirm the return path works from a real host.
- Save the router configurations after successful tests so the static routes persist.

---

## Completion Check

- `Cafe-RT1` lists a static route to `192.168.3.0/24` via `192.168.2.2`, and `Fallout-RT1` lists a static route to `192.168.1.0/24` via `192.168.2.1`.
- Router-to-server and Fallout-SRV-to-Coffee-House-host pings succeed using the console tabs available in this NetAcad lab.
- Router configurations are saved, and the site log notes which static routes were added.