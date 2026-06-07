
## Mission Briefing

Coffee House Beta just linked to the Fallout Shelter, but the routers only recognize the networks directly attached to them. Jeremy wants you to stand up the new segment, confirm the connected routes, and prove each router understands its own LAN and the point-to-point link. You'll run the checks from both sides so the rollout crew can move on to static and dynamic routing with confidence.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Bring the inter-site link online by installing the extra interface module, assigning the /30 address, and verifying the LAN and WAN interfaces on `Cafe-RT1`.
- Configure `Fallout-RT1` with the Coffee House-facing /30 address and the local LAN subnet so it reports each connected network in its routing table.
- Validate the point-to-point link with pings and document the connected routes on both routers before moving to higher-level routing.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`
- The routers may place you directly at a privileged `#` prompt after login. If you land at `>`, use `enable` and the privileged password.

---

## Task 0 - Prepare `Cafe-RT1` for the New Link

Ensure the Coffee House router has the additional interface and the LAN side is ready for routing.

**Steps:**

- Verify the preloaded Ethernet interfaces are present on `Cafe-RT1`; there is no hardware installation step inside the Interactive Console.
- Configure Ethernet0/1 for the new point-to-point subnet `192.168.2.1/30`, and keep Ethernet0/0 as the Coffee House LAN with address `192.168.1.1/24`.
- Verify, using the interface summary, that the LAN port is up/up, the new WAN port is administratively up (status depends on the remote side), and Ethernet0/1 shows the correct description or note.

---

## Task 1 - Configure the Fallout Shelter Router

Bring `Fallout-RT1` onto both the point-to-point link and its local LAN.

**Steps:**

- Confirm the preloaded Ethernet interfaces are present on `Fallout-RT1` before assigning addresses.
- Assign Ethernet0/1 the point-to-point IP `192.168.2.2/30` and leave the interface unshut so it can negotiate with `Cafe-RT1`.
- Configure Ethernet0/0 for the shelter LAN `192.168.3.1/24`, keep Ethernet0/2 (unused) shut down, and tag your interfaces with meaningful descriptions.

---

## Task 2 - Inspect the Connected Routes

Prove each router now recognizes its directly attached networks.

**Steps:**

- On `Cafe-RT1`, review the connected routing table entries and confirm you see `192.168.1.0/24` and `192.168.2.0/30`.
- On `Fallout-RT1`, repeat the inspection and ensure you see connected routes for `192.168.3.0/24` and the shared `/30` link.
- Note that neither router yet knows about the other's LAN; jot this down for the next lesson on static/dynamic routing.

---

## Task 3 - Validate the Point-to-Point Link

Run tests across the link to prove the addressing and interfaces are truly live. Save the end-to-end testing for the static routing lesson.

**Steps:**

- From `Cafe-RT1`, ping `192.168.2.2` to confirm the point-to-point interface is functioning.
- From `Fallout-RT1`, ping `192.168.2.1` and record the successful response.
- If the first ping drops a few probes while the link settles, repeat the ping after a few seconds and confirm it succeeds.
- Save the configurations on both routers once you've recorded the outcomes so the baseli