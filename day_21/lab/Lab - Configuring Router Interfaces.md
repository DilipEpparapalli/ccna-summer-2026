## Mission Briefing

Coffee House Beta just brought its router online, and you're stepping in to finish the job. Jeremy wants the LAN-facing interface activated, the WAN-facing port documented for the carrier hand-off, and the router hardened so field techs can authenticate safely. The switches already have their templates; now it's time to make sure `Cafe-RT1` behaves the same way.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Secure `Cafe-RT1` with Castle Rysen credentials so console, VTY, and privileged access require authentication.
- Bring Ethernet0/0 into service for the LAN, label the interfaces, and keep Ethernet0/1 reserved for the future WAN circuit.
- Verify the router's LAN link and reach `Cafe-SW1` across the management subnet, then save the working state.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`

---

## Task 0 - Apply the Router Baseline

Secure the blank router so the field crew can connect without compromising the site.

**Steps:**

- Connect to `Cafe-RT1`, verify the prompt, and move to privileged mode.
- Configure the router so console and VTY logins require the standard credentials username/password, and set the enable secret.
- Save the running configuration to preserve the baseline.

---

## Task 1 - Inspect Available Interfaces

Understand which hardware ports are present and their default state.

**Steps:**

- Review the interface summary so you can verify all eight onboard Ethernet interfaces (Et0/0-Et0/3 and Et1/0-Et1/3) are present and currently administratively down.
- Note that only Ethernet0/0 will be activated in this lab and Ethernet0/1 will stay reserved for the WAN hand-off while the remaining ports remain unused.
- Record the administrative state of the interfaces in your site notes.

---

## Task 2 - Activate the LAN Interface

Bring the inside port online and document both LAN and WAN interfaces.

**Steps:**

- Configure Ethernet0/0 with the Coffee House LAN description, assign it the `192.168.42.1/24` management address, and bring it out of the shutdown state.
- Leave Ethernet0/1 shut but label it for the future WAN hand-off so field techs know it is pending a carrier circuit.
- Confirm the interface summary now shows Et0/0 as up/up and Et0/1 as administratively down.

---

## Task 3 - Verify the Neighbor Link and Test Connectivity

Prove the router's LAN interface is connected and can reach `Cafe-SW1` across the management subnet.

**Steps:**

- Check CDP from `Cafe-RT1`. In this lab, CDP is enabled on Ethernet0/0, but the switch may not appear as a CDP neighbor.
- Use `show ip interface brief` and `show interfaces description` to confirm Ethernet0/0 is up/up and labeled for the Coffee House LAN.
- Test reachability from `Cafe-RT1` to the switch's management IP (`192.168.42.2`) and note the outcome. The first ping may drop a few probes while ARP and the link settle; repeat the ping if needed and confirm it succeeds.
- Save the configuration once connectivity is confirmed.

---

## Completion Check

- Ethernet0/0 is up/up with description `## CoffeeHouse-LAN`, IP `192.168.42.1/24`, and ping reachability to `Cafe-SW1` (`192.168.42.2`) after ARP/link settling.
- Ethernet0/1 remains administratively down, labeled `## WAN-Pending`, ready for the carrier hand-off, and the other Ethernet interfaces stay shutdown and unlabeled.
- Console and VTY sessions require the Castle credentials, the enable secret is `CrC0ffee!`, and the configuration is saved.