## Field Briefing

Shelter Delta’s relay is dark, and the only thing keeping the bunker stocked is the slim uplink through Coffee House Beta. Jeremy drops you at the fallout shelter with a thermos and a mission: bring the routers online without hand-holding, stitch the branch back to headquarters with static routes, and light up the Internet edge so survivors can submit ration requests. No demos. No hints. Just you, the routers, and a wasteland that runs on caffeine.

## Objectives

For this deployment to be successful, you must complete the following:

- Verify every interface needed for the Coffee House–Shelter–ISP path is present, addressed, and up so connected routes populate correctly.
- Replace any lingering dynamic routing remnants with precise static routes between `Cafe-RT1` and `Fallout-RT1` to restore inter-shelter reachability.
- Establish the default Internet gateway on `Cafe-RT1`, ensuring outbound traffic rides the ISP link while LAN traffic follows the static path.
- Validate routing-table trust decisions and prove the LAN hosts can reach both the remote shelter services and a simulated Internet target.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`

---

## Task 1 — Reestablish the Castle Link

**Objective:** Confirm the physical and logical paths are alive before you rely on routing decisions.

- On `Cafe-RT1`, confirm Ethernet0/0, Ethernet0/1, and Ethernet0/2 exist, carry the assigned addresses above, and sit in an up/up state; correct any shutdowns or mislabels before moving on.
- On `Fallout-RT1`, verify Ethernet0/0 and Ethernet0/1 align with the LAN and point-to-point networks, and ensure no rogue addressing remains from prior drills.
- Capture the connected routes each router advertises (`C` and `L` entries) so you have a baseline snapshot for the engineering log.

## Task 2 — Seal the Inter-Shelter Routes

**Objective:** Replace guidance-system training wheels with deliberate static routing between the sites.

- Audit both routers for leftover `router eigrp 1` or other dynamic processes; remove any entries so only connected networks remain before you add static routes.
- Configure `Cafe-RT1` with a static route to `192.168.3.0/24` via `192.168.2.2`, and mirror the path on `Fallout-RT1` toward `192.168.1.0/24` via `192.168.2.1`.
- Verify the routing tables display `S` entries with the correct next-hops and that end-to-end pings between `Cafe-PC1` and `Shelter-SRV` succeed in both directions.

## Task 3 — Anchor the Internet Edge

**Objective:** Deliver a resilient default path that keeps the Coffee House online when traffic leaves the local prefixes.

- Confirm `Cafe-RT1`’s ISP-facing interface is addressed as `216.0.5.2/30` and the provider gateway (`216.0.5.1`) responds.
- Install a default route (`0.0.0.0/0`) on `Cafe-RT1` pointing to `216.0.5.1`, ensuring the gateway of last resort registers in the routing table.
- From `Cafe-PC1`, verify that pings to `203.0.113.8` fail before the default route, succeed once it is in place, and that the return path traverses `Cafe-RT1` without disturbing the shelter static routes.

## Task 4 — Verify Route Trust and Failover Logic

**Objective:** Prove the router selects the intended paths and understands which entries to prefer under pressure.

- Record the `[administrative distance/metric]` details for the static inter-shelter routes and the default route on `Cafe-RT1`, confirming the static LAN entries outrank the default path.
- Create a temporary floating static default on `Fallout-RT1` toward `192.168.2.1` with an administrative distance of 5, document how it stays dormant while the ISP remains reachable, then remove it to restore the baseline.
- Log the final routing table snapshot on both routers, highlighting the `C`, `S`, and `S*` codes, so Jeremy can corroborate your path-selection analysis.

---

## Completion Check

- `Cafe-RT1` and `Fallout-RT1` list the correct connected networks and reciprocal static routes, with `Cafe-PC1` and `Shelter-SRV` exchanging traffic successfully.
- `Cafe-RT1` advertises a default route toward `216.0.5.1`, and `Cafe-PC1` reaches the simulated Internet address through the ISP link while still honoring the static LAN route.
- Final routing tables document the expected administrative distances (`S` routes at AD 1, floating default at AD 5 during testing) and no dynamic routing processes remain active.

not started