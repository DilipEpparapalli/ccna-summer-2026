## Simple Explanation
Computers operate in binary — everything is stored as 1s and 0s. Humans use decimal. To understand subnetting, you need to be able to move between the two fluently. Decimal to binary conversion is the mechanical skill that makes subnet masks and IP boundaries readable.

## Why It Exists
IPv4 addresses and subnet masks are stored as binary under the hood. When you see `192.168.1.10`, the router isn't reading those decimal numbers — it's working with four 8-bit binary values. If you can't convert, subnet math is just memorization. If you can, it's logic.

## How It Works
An IPv4 octet is 8 bits. Each bit position has a fixed value:

| Bit position | 8 | 7 | 6 | 5 | 4 | 3 | 2 | 1 |
|---|---|---|---|---|---|---|---|---|
| Value | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

**The bucket method** — work left to right, MSB to LSB:
1. Does 128 fit into your number? If yes → write `1`, subtract it. If no → write `0`.
2. Repeat for each value down to 1.
3. You always end up with exactly 8 digits.

**Example: 153**
```
128  64  32  16   8   4   2   1
  1   0   0   1   1   0   0   1
128 + 16 + 8 + 1 = 153 ✓
Result: 10011001
```

**Example: 210**
```
128  64  32  16   8   4   2   1
  1   1   0   1   0   0   1   0
128 + 64 + 16 + 2 = 210 ✓
Result: 11010010
```

**Example: 20**
```
128  64  32  16   8   4   2   1
  0   0   0   1   0   1   0   0
16 + 4 = 20 ✓
Result: 00010100
```

## Key Terms
| Term | Meaning |
|------|---------|
| Bit | A single binary digit — either 1 or 0 |
| Byte | 8 bits |
| Octet | One 8-bit section of an IPv4 address (same as a byte) |
| MSB | Most Significant Bit — the leftmost bit, value 128 |
| LSB | Least Significant Bit — the rightmost bit, value 1 |
| Binary | Base-2 number system using only 1s and 0s |

## Why 255 Is the Max
All 8 bits turned on = `11111111` = 128+64+32+16+8+4+2+1 = **255**.
That's why every octet in an IPv4 address is capped at 255 — there are no more bit positions to flip.

## Real-World Connection
Every IP address you've configured on your Pi — your Pi-hole, WireGuard, VLANs — the kernel stores those as binary. When your router decides whether `192.168.10.5` belongs to VLAN 10 or VLAN 20, it's ANDing binary values together. This conversion skill is the foundation of that logic.

## Exam Traps
1. **Divide-by-2 vs bucket method** — both produce correct binary, but the bucket method is faster for networking and builds pattern recognition for subnetting. Practice the bucket method.
2. **Always write all 8 bits** — `20` in binary is `00010100`, not `10100`. Leading zeros matter in networking contexts.
3. **2⁸ = 256 ≠ max value** — 256 is the *count* of possible values (0–255). The *maximum value* is 255. Know the difference; the exam will test both framings.

## Recall Questions
1. Convert 172 to binary using the bucket method. Show all 8 positions.
2. What is the maximum value of a single octet, and why?
3. What binary value does `00001111` represent in decimal?
4. Why do networking engineers use the bucket method instead of divide-by-2?
