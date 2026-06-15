## Field Briefing

District Theta’s café bunker is back online, but the fallout shelters upstream keep filing tickets: some endpoints can reach the emulated internet, others stall the moment they leave the LAN. Jeremy dispatches you to the site, puts you beside `Cafe-Rtr`, and expects you to rebuild the entire NAT stack without a script. The ISP emulator is on standby, the café floor is buzzing, and Castle Rysen needs one engineer who can go from raw router ports to a fully overloaded edge in a single push.

## Objectives

For this deployment to be successful, you must complete the following:

- Stand up the ISP-side infrastructure so the café has a functional upstream with public DNS loopbacks.
- Publish and protect the café hosts by progressing from static mappings to shared address pools.
- Finish with interface-based NAT overload that keeps every café device online through a single public identity.

**Access Credentials**

- Console/VTY password: `Cisco`
- Enable secret: `Cisco`

---

## Task 1 — Rebuild the ISP Edge

**Objective:** Deliver a live ISP emulator with secured access, an active WAN link, and public DNS targets.

- Position `ISP-Rtr` opposite `Cafe-Rtr`, connect `Ethernet0/0` to `Cafe-Rtr Ethernet0/1`, and assign 216.0.5.1/30 to the ISP side while confirming 216.0.5.2/30 is active on the café side.
- Apply Castle Rysen’s standard identity controls on `ISP-Rtr` (hostname `ISP-Rtr`, console/VTY logins, and enable secret all set to the site password) and ensure the WAN port is operational.
- Create loopbacks on `ISP-Rtr` for 1.1.1.1/32 and 8.8.8.8/32, then confirm `Cafe-Rtr` can reach both addresses before moving on.

## Task 2 — Publish the Café Menu via Static NAT

**Objective:** Expose `PC1` (192.168.1.50) to the public network with a one-to-one translation while keeping the LAN boundary clear.

- Confirm `Cafe-Rtr Ethernet0/0` is designated as the inside interface and `Ethernet0/1` as the outside boundary for address translation.
- Bind 192.168.1.50 to public address 216.0.5.20 and record the static entry in the translation table.
- Demonstrate that traffic sourced from `ISP-Rtr` toward 216.0.5.20 now lands on `PC1`, while `PC2` (192.168.1.51) remains unadvertised.

## Task 3 — Shift to Dynamic NAT Pooling

**Objective:** Allow the entire 192.168.1.0/24 café subnet to borrow public IPs from a defined pool.

- Remove the static mapping from Task 2 and maintain the inside/outside interface roles already assigned on `Cafe-Rtr`.
- Build a standard list that matches all hosts within 192.168.1.0/24, then create the `Cafe-Public` pool covering 216.0.5.50 through 216.0.5.100 with a /24 mask and associate the list to that pool for outbound translations.
- Generate simultaneous traffic from `PC1` and `PC2` and confirm each device consumes distinct pool addresses that appear in the translation table.

## Task 4 — Consolidate with NAT Overload

**Objective:** Collapse the café subnet behind the WAN interface so all hosts share 216.0.5.2 by port number.

- Withdraw the dynamic pool rule while keeping the access list available for reuse.
- Halt new translations by disabling `Cafe-Rtr` WAN and LAN interfaces, then apply NAT overload using access list 1 on interface `Ethernet0/1` before restoring the ports to service.
- Resume café traffic and document that multiple inside hosts now map to 216.0.5.2 with unique session identifiers, showing overload is active.

---

## Completion Check

- `ISP-Rtr` delivers 216.0.5.1/30 on the WAN link and responds to 1.1.1.1 and 8.8.8.8 reachability tests from `Cafe-Rtr`.
- Static, pool-based dynamic, and overloaded translations were each observed in the NAT table during the deployment, with the final state showing all café hosts sharing 216.0.5.2.
- `PC1` and `PC2` maintain stable connectivity to the simulated internet while `Cafe-Rtr` enforces the inside/outside boundaries required for NAT overload.

 copy on select