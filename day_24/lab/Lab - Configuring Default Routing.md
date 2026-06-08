## Mission Briefing

The Coffee House router can reach the Fallout Shelter, but it still needs a gateway to the wider Internet. Jeremy wants you to add the ISP link, install a default route, and prove that unknown destinations are forwarded toward the service provider. Once the default route is in place, the site is ready for public access testing.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Prepare `Cafe-RT1` for the Internet uplink by confirming the WAN interface is present and the ISP-provided address is applied.
- Add a default route on `Cafe-RT1` that points to the ISP router so unfamiliar destinations have a path.
- Validate that the Coffee House LAN host can reach an Internet-facing test address and that a local host with a missing route now succeeds once the default route exists.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`
- The routers may place you directly at a privileged `#` prompt after login. If you land at `>`, use `enable` and the privileged password.

---

## Task 0 - Prepare the ISP Link

Ensure the Coffee House router has the correct hardware and addressing for the WAN connection.

**Steps:**

- Confirm `Cafe-RT1` includes the prewired public uplink on `Ethernet0/2`; there is no hardware installation step inside the Interactive Console.
- Configure the interface with the ISP-provided IP address `216.0.5.2/30` while keeping the LAN and branch links intact (`192.168.1.0/24` and `192.168.2.0/30`).
- Verify the interface summary so Ethernet0/2 is up/up, labeled appropriately, and can reach the ISP next-hop `216.0.5.1`.

---

## Task 1 - Configure the Default Route

Tell `Cafe-RT1` where to send traffic when no specific route exists.

**Steps:**

- Record the ISP next-hop address (for example, `216.0.5.1`) supplied by the provider.
- Add a default route (`0.0.0.0/0`) on `Cafe-RT1` that forwards unknown traffic to the ISP next-hop.
- Verify the routing table shows the default route with the expected next-hop and the "Gateway of last resort" entry is populated.

---

## Task 2 - Validate Internet Reachability

Prove the Coffee House LAN has a working path to an external test address.

**Steps:**

- From the `Cafe-PC` console tab, attempt to reach the simulated Internet test IP before installing the default route: `ping -c 5 8.8.8.8`. It should fail because `Cafe-RT1` has no default route yet.
- After adding the default route, repeat `ping -c 5 8.8.8.8` from `Cafe-PC` and confirm it succeeds.
- Save the router configuration once verification is complete.
---

## Completion Check

- Ethernet0/2 on `Cafe-RT1` carries the ISP IP address, and the routing table lists a default route pointing to the provider.
- `Cafe-PC` regains external connectivity once the default route is in place, evidenced by successful `ping -c 5 8.8.8.8` results.
- Router configurations are saved, and the operations log notes the default route details for future reference.