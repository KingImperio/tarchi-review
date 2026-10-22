# TArchI Specification — Tridak (draft, 2026-10-21)

## 1. THE TRIDAK

The **Tridak** is the atomic digit of TArchI, representing the base‑3 balanced radix digit in `{-1, 0, +1}`.

| State | Symbol | Role | Name | Physical storage (i8) |
|-------|--------|------|------|----------------------|
| -1    | `0xFF` | falsum | **NEG** | bit‑pattern `10000000` (two's complement) |
| 0     | `0x00` | nullum | **ZERO** | all bits zero |
| +1    | `0x01` | verum | **POS** | bit‑pattern `00000001` |

*Invariant*: A Tridak value is exactly one of the three states; out‑of‑range writes are undefined.

*Negation*: Flip the value: `-x = -1 * x`. Since there is no sign bit, negation is a single-bit logical NOT across the sign‑magnitude representation and costs O(1).

## 2. THE BALRADIX FAMILY

For any base `B ≥ 2`, the **balanced radix** digit range is `[⌊-B/2⌋, ⌊B/2⌋]` where `⌊·⌋` denotes the floor (balanced with one extra negative if B is even).

*Why balanced beats signed‑magnitude & two’s complement for intelligence*:
- No dedicated sign bit – digit values alone encode sign.
- Negation is free (`-d = d'`). No two’s complement conversion cost.
- Single canonical zero (`0`).
- Symmetric arithmetic – `d1 + d2 = -(d1 + d2)'`.

**Named members**:
- **Tridak** (base 3): `{-1, 0, +1}` (1 trit).
- **Baldec** (base 10): `{-5, …, +5}` (1 baldec, 3.3219 bits).
- **Other bases**:
  - B=2 balanced → `{-0?}` (degenerate – never used).
  - B=5 → `{-2, -1, 0, +1, +2}` (1.32193 bits per digit).
  - B=8 → `{-4, -3, …, +3, +4}` (1.32193 bits per digit, more bit‑packed slots for same total bits).

## 3. BALANCED ADDITION AND CARRY

For base `B`, given digit sum `s ∈ [-(B‑1), B‑1]`:
```
c = trunc(s / B)          // integer division toward zero
c = floor(s/B) if s≥0 else ceil(s/B)
d = s - c * B
```

**Base‑3 worked table**:
| + | - | sum | carry | digit |
|---|---|-----|--------|-------|
| +1| +1|  2  |   1    |  -1   |
| +1|  0|  1  |   0    |  +1   |
| +1| -1|  0  |   0    |   0   |
|  0| +1|  1  |   0    |  +1   |
|  0|  0|  0  |   0    |   0   |
|  0| -1| -1 |   0    |  -1   |
| -1| +1|  0  |   0    |   0   |
| -1|  0| -1 |   0    |  -1   |
| -1| -1| -2 |  -1    |  +1   |

*Example*: `+1 + +1 = -1` with carry `1`.

**Base‑10 balanced example**:
```
6 + 7 = 13
c = trunc(13/10) = 1
d = 13 - 1*10 = 3
```
Result: baldec `3` with carry `1`.

*Why carries are smaller*: Max carry is `±1` vs `±(B‑1)` in unbalanced; this enables a packed substrate with minimal overflow.

## 4. PACKING AND DENSITY

Theoretical bits per trit: `log2(3) = 1.58496`.

| k (trits) | States | Bits (ceil(log2 N)) | Realized bits/trit |
|-----------|--------|---------------------|-------------------|
| 3         | 27     | 5                   | 1.666…            |
| 5         | 243    | 8                   | 1.6              |
| 8         | 6561   | 13                  | 1.625            |
| 12        | 531441 | 20                  | 1.6667           |

*Sub‑byte addressing*: 8 trits map directly to a single byte (0‑255) with 6561 possible states, i.e., a 26.2:1 fan‑out – enough to page a knowledge substrate into RAM.

**State‑space comparisons**:
- Rubik’s cube: `4.3252×10¹⁹` states → `66` bits vs `42` trits.
- Chess: `4.8×10⁴⁴` positions → `149` bits vs `94` trits.
- Generic: `bits = ceil(log2 N)`, `trits = ceil(log3 N)`.

## 5. BALDEC AS A SCALAR

Baldec values `{-5,…,+5}` replace floating‑point weights for edge strength and confidence.

**Float → Baldec canonical mapping**:
```
s = round(10 * w) - 5   // w in [0,1]
```
| w      | s (Baldec) | Interpretation |
|--------|-----------|----------------|
| 1.0    | +5        | definite support |
| 0.5     | 0         | neutral / undecided |
| 0.0     | -5        | definite opposition |

*Zero semantics*: `0` = neutral/undetermined – no separate contradiction flag needed.
*Positive*: supporting/strengthening.
*Negative*: opposing/weakening.

## 6. TRIT LOGIC (KLEENE STRONG THREE‑VALUED)

Truth tables for the four orthogonal roles (same digit values, different semantic overlays):

**Truth (proposition)**: `-1` = false, `0` = unknown, `+1` = true.
- NOT: `¬x = -x`
- AND: `x ∧ y` = `min(x, y)` (idempotent, commutative)
- OR: `x ∨ y` = `max(x, y)`
- IMPLIES: `x → y` = `(+1 if y=+1; -1 if x=+1∧y=-1; else 0)`

**Polarity (edge)**: `-1` = opposes, `0` = undetermined, `+1` = supports.
**Epistemic (ledger)**: `-1` = rejected, `0` = candidate, `+1` = held.
**Balance (charge)**: `-1` = deficit, `0` = equilibrium, `+1` = surplus.

All four layers share the same trinary digit set – no axis overloading.

## 7. CONVERSIONS AND CANONICALIZATION

**Trit → bits** (little‑endian, packed):
```rust
fn pack_trits(trits: &[i8]) -> u64 {
    let mut word = 0u64;
    for (i, &t) in trits.iter().enumerate() {
        let idx = (t + 1) as usize; // map -1→0, 0→1, +1→2
        word |= (idx as u64) << (i * 2);
    }
    word
}
```

**Bits → trits** (inverse):
```rust
fn unpack_trits(word: u64, k: usize) -> Vec<i8> {
    let mut trits = Vec::with_capacity(k);
    for i in 0..k {
        let idx = (word >> (i * 2)) & 0x03;
        trits.push((idx as i8) - 1); // 0→-1,1→0,2→+1
    }
    trits
}
```

**Float → Baldec** (see §5).

**Baldec → Float**: `w = (s + 5) / 10.0`.

**Integer → balanced radix** (standard algorithm):
```rust
fn to_balanced(x: i64, B: i64) -> Vec<i64> {
    let mut res = Vec::new();
    while x != 0 {
        let r = x % B;
        if r <= -B/2 { r += B; x += B; }
        else if r > B/2  { r -= B; x -= B; }
        res.push(r);
        x /= B;
    }
    res.reverse();
    res
}
```

## 8. INVARIANTS AND PROPERTY TESTS

Checklist for Rust crate `tridak`:
- [ ] Negation involution: `¬(¬x) == x`
- [ ] Canonical zero uniqueness: `to_balanced(0, B) == [0]`
- [ ] Carry round‑trip: `pack_trits(unpack_trits(word, k)) == word`
- [ ] Distributivity: `x ∧ (y ∨ z) == (x ∧ y) ∨ (x ∧ z)` for truth values
- [ ] Packing bijection for `k ∈ {3,5,8,12}` – verify `2^{ceil(k*log2 3)} == 3^k`
- [ ] Baldec↔float monotonicity & round‑trip error ≤ `1/20`
- [ ] No operation ever writes outside `[-ceil(B/2), floor(B/2)]`
- [ ] Element‑wise operations preserve trinary digit set

## 9. PERFORMANCE NOTES

- **SIMD‑friendly**: 4 trits fit in an i16 lane; 8 trits in an i32 lane – pure integer addition and range checks, no multiply‑heavy operations.
- **Inner‑loop**: Trit arithmetic uses only signed adds and saturating comparisons; weights are trinary (`-1,0,+1`) so `weight * value` is a single conditional move.
- **Packing cost model**: Pack once at load, operate in trit space, unpack only at substrate edges (e.g., JSON export).

## 10. WHY NOT QUANTUM

Objection: "Qubits exist in 0 and 1 simultaneously → richer state space."

TArchI position:
- Superposition is unnecessary for deterministic intelligence; probabilistic collapse adds latency and verification cost.
- Balanced radix `-1,0,+1` gives three orthogonal axes (truth, polarity, epistemic) without physical indeterminacy.
- Deterministic operations are verifiable, debuggable, and can be formally proven – essential for safety‑critical inference.
- A phone can already execute trinary arithmetic in sub‑nanosecond cycles; quantum control requires cryogenic infrastructure.

Trade: sacrifice quantum parallelism for simplicity, verifiability, and deterministic correctness.

---
**See also**: `charter.md`, `nubit.md`, `package-format.md`, `../discussion/discussion.json`
