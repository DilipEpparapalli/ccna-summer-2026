## Field Briefing

Jeremy wants the Coffee House network tied into the Fallout shelter floor so both sites can exchange services before the main WAN cutover. The routers are racked with baseline credentials and interface labels, but every port is still shut down. Your job is to activate the LAN facing links, light up the point-to-point handoff, and prove end-to-end reachability using only local routing.

### Circuit Diagram:
![[Circuit_Diagram.png]]


## Objectives

For this deployment to be successful, you must complete the following:

- Confirm both routers meet Castle security standards (type 9 secrets, local console/VTY authentication) and are running software version 17.16.
- Bring `Cafe-RT1 Ethernet0/0` online as the `192.168.42.1/24` gateway while preserving its description.
- Bring `Fallout-RT1 Ethernet0/0` online as the `192.168.84.1/24` gateway while preserving its description.
- Light up the inter-router link on `Ethernet0/1` with the `10.8.0.0/30` addressing (`Cafe-RT1` at `.1`, `Fallout-RT1` at `.2`) and keep all annotations intact.
- Establish static reachability between the two LANs over the point-to-point segment and prove end-to-end connectivity.

**Access Credentials**

- Console / VTY username: `cisco`
- Console / VTY secret: `cisco`
- Privileged access: `CrC0ffee!`

---

## Task 1 — Baseline Each Router

**Objective:** Confirm security posture and prepare the LAN interfaces.

- Connect to `Cafe-RT1`, confirm the hostname, verify the software release is 17.16, and ensure both console and VTY entry require the Castle credentials before privileged access unlocks with `CrC0ffee!`.
- Repeat on `Fallout-RT1`, including verification that successful login events are logged for auditing.
- Capture the pre-change operational state of `Ethernet0/0` and `Ethernet0/1` on both routers without altering their descriptions.

## Task 2 — Stand Up the LAN Gateways

**Objective:** Activate the inside interfaces and confirm local connectivity.

- On `Cafe-RT1`, assign the `192.168.42.1/24` gateway address to `Ethernet0/0`, return the interface to service, and verify it reports an `up/up` state.
- On `Fallout-RT1`, assign the `192.168.84.1/24` gateway address to `Ethernet0/0`, restore service, and confirm the interface likewise settles at `up/up`.
- Document the interface statistics immediately before activation and again once traffic flows to confirm a clean turn-up.

## Task 3 — Build the Point-to-Point Link

**Objective:** Bring up the inter-router connection for route exchange.

- On `Cafe-RT1`, apply the `10.8.0.1/30` address to `Ethernet0/1`, return the interface to service, and ensure the existing description is untouched.
- On `Fallout-RT1`, apply the `10.8.0.2/30` address to `Ethernet0/1`, restore service, and verify both ends of the link report `line protocol up`.
- Review the interface summaries on each router to ensure the LAN and point-to-point links all show `up/up` with their expected addressing.

## Task 4 — Establish Local Routing

**Objective:** Provide reachability to the remote LANs.

- On `Cafe-RT1`, create a static path to the Fallout LAN by directing `192.168.84.0/24` traffic toward the Fallout router’s point-to-point address.
- On `Fallout-RT1`, mirror the static path so `192.168.42.0/24` traffic targets the Coffee House router across the same link.
- Inspect each routing table to confirm the new entries resolve via `Ethernet0/1`.
- Prove reachability by testing traffic from each router to the opposite LAN gateway and capture the resulting neighbor table updates.

## Task 5 — Finalize and Document

**Objective:** Lock down the changes for Jeremy’s review.

- Re-run interface statistics on `Ethernet0/0` and `Ethernet0/1` for both routers, watching for errors or drops.
- Save the validated configuration state to nonvolatile storage on each router.
- Compile your notes: software versions, interface states, static routing details, neighbor discoveries, and confirmation that every description remains in place.

---

## Completion Check

- `Cafe-RT1` and `Fallout-RT1` require the Castle credentials, operate on version 17.16, and record successful login events for auditing.
- `Cafe-RT1 Ethernet0/0` serves `192.168.42.1/24`, `Fallout-RT1 Ethernet0/0` serves `192.168.84.1/24`, and both adapters remain in the up state with their original descriptions.
- The point-to-point link (`Ethernet0/1` on each router) runs on `10.8.0.0/30`, static routes are installed, and cross-site pings succeed.
- Interface counters show no anomalies after the cutover, and the final configurations are saved for the change log.

Submit your validation notes to Jeremy once the checks pass.

not started