## Mission Briefing

Jeremy just brewed a fresh pot in the Castle Rysen bunker and wants you to prove you can read a routing table like it's a survival map. The routers already talk to each other; now you must interpret every clue the table gives you so you can troubleshoot at a glance, choose the best path, and keep coffee flowing to the wasteland.

## Training Objectives

For this deployment to be successful, you must complete the following:

- Decode the fields of a routing table entry on `Cafe-RT1`, including codes, prefix information, and exit details.
- Compare administrative distance and metric values to understand which route the router believes first.
- Demonstrate how variable subnetting and host routes appear so you can separate real reachability from bookkeeping entries.

**Access Credentials**

- Router console / VTY: `cisco` / `cisco`
- Username: `cisco`
- Password: `cisco`
- Privileged access: `CrC0ffee!`
- The router may place you directly at a privileged `#` prompt after login. If you land at `>`, use `enable` and the privileged password.

---

## Task 0 - Decode the Routing Table Story

Analyze the `Cafe-RT1` routing table entry for the Fallout Shelter network and break down each part of the listing.

**Steps:**

- Display the current routing table on `Cafe-RT1` and focus on the line that leads to the `192.168.3.0/24` network.
- Identify what the route code, prefix length, administrative distance, and metric reveal about how the path was learned and trusted.
- Record the next-hop, route age, and exit interface so you can explain how the packet leaves the router.

---

## Task 1 - Compare Administrative Distance and Metric

Introduce a competing path so you can watch the router pick the most believable option, then restore the baseline.

**Steps:**

- Document the existing EIGRP-learned route to `192.168.3.0/24`, noting its believability score.
- Add a temporary static route toward the same destination using the reachable point-to-point next hop, then observe how the routing table entry changes when a more trusted source appears.
- Remove the temporary static route and verify the dynamic entry returns as the active path.

---

## Task 2 - Spot Variably Subnetted and Host Routes

Use loopback networks to see how the routing table reports classful versus subnetted entries and why host routes appear alongside them.

**Steps:**

- Create two loopback interfaces on `Cafe-RT1` within the 10.0.0.0 private block using a Class C mask to simulate carved-up subnets.
- Review the routing table and note how the router lists the parent network as variably subnetted while also generating internal host routes.
- Remove the temporary loopbacks once you understand the listing so the training topology returns to its original state.

---

## Completion Check

- You can explain every field of the `192.168.3.0/24` routing table line, including how the router learned it and where traffic exits.
- You proved how administrative distance overrides metrics when two routes target the same prefix and restored the dynamic path afterward.
- You recognized variably subnetted messages and host routes as internal artifacts, not extra remote networks, and cleaned up the temporary interfaces.