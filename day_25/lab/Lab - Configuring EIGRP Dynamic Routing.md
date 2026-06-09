## Mission Briefing

Your static routes kept Coffee House Beta talking to Fallout Shelter, but Jeremy wants that link to heal itself the next time someone forgets a configuration. You'll replace the manual entries with EIGRP so the routers discover each other, advertise their LANs, and keep the WAN link resilient. Once dynamic routing is up, the rollout team can scale the design without editing every table by hand.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Remove the existing static routes between `Cafe-RT1` and `Fallout-RT1` so you can start with a clean slate.
- Enable EIGRP (autonomous system 1) on both routers, advertising their respective LANs and the point-to-point link.
- Verify the routers form an adjacency, exchange routes, and restore end-to-end connectivity.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`
- The routers may place you directly at a privileged `#` prompt after login. If you land at `>`, use `enable` and the privileged password.

---

## Task 0 - Clear the Static Routes

Clear the manual routes so you can prove dynamic routing takes over.

**Steps:**

- On `Cafe-RT1`, identify the static route pointing to the Fallout LAN and record its details so you can confirm removal.
- Remove that static route and check the routing table to ensure the entry disappears.
- Repeat the removal on `Fallout-RT1` for the Coffee House LAN route. The default routes stay in place; only remove the LAN-to-LAN static routes that EIGRP will replace.

---

## Task 1 - Enable EIGRP on `Cafe-RT1`

Bring the coffee shop router into the EIGRP autonomous system.

**Steps:**

- Enter EIGRP configuration (autonomous system 1) and advertise the Coffee House LAN (`192.168.1.0/24`) and the point-to-point WAN (`192.168.2.0/30`).
- Confirm the ISP-facing interface is excluded so Internet routes are not shared with the service provider.
- Check for the newly formed EIGRP neighbor once `Fallout-RT1` is configured.

---

## Task 2 - Enable EIGRP on `Fallout-RT1`

Bring the fallout router into the same EIGRP autonomous system and advertise its LAN.

**Steps:**

- Activate EIGRP autonomous system 1, advertising the Fallout LAN (`192.168.3.0/24`) and the shared WAN (`192.168.2.0/30`).
- Ensure no other interfaces are included in the advertisement list.
- Verify the adjacency with `Cafe-RT1` forms and the Fallout router learns the Coffee House LAN dynamically.

---

## Task 3 - Validate Dynamic Routing

Confirm the routers exchanged routes and LAN hosts can reach each other again.

**Steps:**

- From `Cafe-RT1`, verify the EIGRP-learned route to `192.168.3.0/24` appears in the routing table (code `D`).
- From `Fallout-RT1`, verify it now has a `D` route to `192.168.1.0/24`.
- Run end-to-end tests between the live Linux host tabs to ensure connectivity is restored, then save both router configurations.

---

## Completion Check

- Static routes between the Coffee House and Fallout networks are removed, and EIGRP routes replace them (code `D`).
- Both routers show an EIGRP neighbor relationship and advertise their connected LANs over the point-to-point link.
- LAN-to-LAN pings succeed, proving dynamic routing restored two-way connectivity, and configurations are saved for future troubleshooting.

 copy on select