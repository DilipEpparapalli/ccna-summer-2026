
## What Was Done

### Task 0 — Baseline Capture

Ran `show interface ethernet0/0` to establish a clean baseline before any changes.

**Key values recorded:**

| Field | Value |
|-------|-------|
| Status | up / up |
| Speed | 10 Mbps (BW 10000 Kbit/sec) |
| Duplex | Full (inferred from zero collisions) |
| txload / rxload | 1/255 — effectively idle |
| 5-min input rate | 1000 bits/sec, 1 packet/sec |
| 5-min output rate | 1000 bits/sec, 1 packet/sec |
| Input errors | 0 |
| CRC | 0 |
| Collisions | 0 |
| Late collisions | 0 |
| Interface resets | 2 |

Light background traffic confirmed — an embedded IP SLA echo to Cafe-PC1
(192.168.10.100) keeps the counters ticking even without manual traffic generation.

**On the 2 interface resets:** Zero errors and zero collisions alongside the resets
indicates these occurred during interface initialization — not a fault condition.
A reset counter climbing on a live, stable interface would warrant investigation.

---

### Task 1 — MAC Correlation to Switch

On Cafe-SW1, checked the MAC address table to confirm the router's MAC was being
learned on the expected port.

```
show mac address-table
```

Confirmed: Cafe-RT1's MAC (from `show interface ethernet0/0` output) was learned on
the switch port connected to `Eth0/0`. This proves the physical link is landing on
the correct switch port and the switch is actively learning from it.

**Why this matters:** In a real deployment, this is how you verify physical connectivity
without tracing cables — log into the switch, check the MAC table, confirm the router's
MAC appears on the expected port.

---

### Task 2 — Duplex Mismatch Analysis

Compared the clean lab baseline against a sample mismatch output from physical gear:

**Sample mismatch output:**
```
Ethernet0/0 is up, line protocol is up (half-duplex)
  0 input errors, 56 CRC, 89 collisions, 0 late collision
  7 interface resets
```

**Comparison:**

| Counter | Lab (healthy) | Sample (mismatch) |
|---------|--------------|-------------------|
| CRC errors | 0 | 56 |
| Collisions | 0 | 89 |
| Late collisions | 0 | 0 |
| Interface resets | 2 | 7 |

**Diagnostic logic — how to read the mismatch pattern:**

- **CRC errors alone** → suspect a physical problem: bad cable, poor termination,
  interference, cable too long
- **Collisions on a switched link** → red flag on its own; switched ports have
  dedicated segments so collisions should not occur
- **CRC + collisions together** → duplex mismatch until proven otherwise; one side
  is full-duplex, the other is half-duplex, causing both sides to corrupt and collide

Late collisions being zero does not rule out duplex mismatch — they are one possible
symptom, not a required indicator. The combination of CRC and collisions on a switched
link is the key pattern to recognize.

---

## What This Proved

- A healthy full-duplex link shows zero CRC, zero collisions, and zero late collisions
  regardless of traffic volume
- Interface resets alone are not an indicator of fault — context matters (zero errors
  alongside them = benign initialization events)
- Correlating the router MAC to the switch MAC table is a fast, cable-free way to
  verify physical connectivity lands on the right port
- CRC errors + collisions on a switched link = duplex mismatch; CRC errors alone =
  suspect physical cable problem
