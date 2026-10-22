# `.tarchi` — TArchI Package Format

Status: Draft, 2026-10-21
Author: Fola Adam
Project: TArchI — True Archaion Intelligence

> **Note for whoever wires this up:** this document was authored as `docs/package-format.md` in the
> TArchI repo. It is staged here so it can be copied into place:
> `cp tarchi-package-format-spec.md <repo>/docs/package-format.md`

---

## A. The package *is* the knowledge

TArchI has no trained weights and no gradient descent. Its knowledge lives in a single artifact: the
`.tarchi` package.

```
package ::= header, axioms, rules, facts, entities, index, quarantine, integrity
```

The package is therefore the exact counterpart of an LLM's weight file, with three properties weights
can never have:

1. **Verifiable** — hash integrity *and* semantic integrity (every fact re-checked against the axiom set
   on load).
2. **Composable** — base + domain + personal packages merge deterministically.
3. **Updatable by delta** — correcting one fact ships a patch, not a new release.

## B. The Astra unit

**1 Astra** = the verifiable **capability** of a 27B-param reference LLM, measured on two suites:

| Suite | What it measures | Scoring |
|---|---|---|
| **Knowledge recall** | verifiable factual questions, with distractors and injected contradictions | correctness **with citations** |
| **Derivation** | multi-step questions that must be *derived* from axioms and rules | the **derivation path**, not just the final string |

A package that merely memorises answers scores zero on derivation, so a database cannot pass as an
intelligence. **3.7 Astra = 100B-param-equivalent capability.**

Storage is a **separate, hard constraint**, measured in bytes and in INT4-param-equivalents (INT4 =
0.5 B/param, the deployment-realistic phone baseline):

| Unit | Reference (INT4-equivalent) | TArchI target | Reduction |
|---|---|---|---|
| 1 Astra (27B-equiv.) | 13.5 GB | **1.35 GB** | 10x |
| 3.7 Astra (100B-equiv.) | 50 GB | **~5 GB** | 10x |

And the binding operational constraint: **the whole thing must operate within a 2 GB resident working
set**, package on flash, demand-paged.

## C. Page layout and residency

- **Page size 4 KiB**, `mmap`-able, read-only on the device.
- **Balradix-coded page table** — page numbers stored as balanced-radix digits, enabling sub-byte
  addressing. 8 trits address 6561 states in one byte.
- **The Tridak is the atomic load/offload unit** — residency is managed per page, at trit granularity.
- **Hot pages are pinned by the reasoning frontier**: the axiom pages plus the pages of the active
  derivation chain. Cold pages are evicted. Residency is a *reasoning* decision, not LRU.
- Engine overhead target **<= 200 MB**; resident working set **<= 2 GB**.
- **Python is bindings-only and never the runtime.** The core is Rust (arena-allocated, no GC);
  Python's per-object overhead would blow the budget at this substrate size.

## D. Section table

All eight sections exist in v0.1 **even when empty** — format stability, so v0.2 can ship induced rules
without breaking existing readers.

| # | Section | Contents | v0.1 |
|---|---|---|---|
| 0 | **Header** | magic, format version, flags, tridak base, axiom-set hash, package id (Balradix), creation provenance, license/attribution, section offset table | ✅ |
| 1 | **Axiom** | foundational axioms and rule forms, trinary-encoded | ✅ |
| 2 | **Entity** | canonical entity table: id, type, compressed alias list | ✅ |
| 3 | **Fact** | triples as Balradix-coded (subject, predicate, object) ids + polarity trit + Baldec strength + provenance id | ✅ |
| 4 | **Rule** | derived / induced generalizations | **reserved, empty** |
| 5 | **Index** | entity→facts, subject→facts, predicate→facts, trinary-encoded offsets | ✅ |
| 6 | **Quarantine** | rejected and contradicted claims — **audit only, never used in inference** | ✅ |
| 7 | **Integrity** | per-page Blake3 hash, optional signature block | ✅ |

**Why Quarantine is a first-class section:** it makes the anti-hallucination mechanism *visible*. You can
read what TArchI refused to believe and why. An LLM cannot tell you what it discarded, because it has no
discard step.

## E. Two-tier integrity

| Tier | Guarantees | Failure behaviour |
|---|---|---|
| **Hash** (per-page Blake3) | the bits did not change | page rejected, load aborts for that page |
| **Semantic** (Rubik re-verification) | every fact still agrees with the axiom set | facts flagged, **package loads degraded** — inference excludes them, never silently |

Weights can only ever be hash-checked. Semantic re-verification is what a `.tarchi` adds.

## F. Composition and deltas

Packages compose: `base` + `domain` + `personal`.

Merge is **deterministic**, keyed on `(canonical id, polarity, strength)`:

- stronger verified claim wins;
- equal-strength disagreement escalates to **quarantine in both** — never a silent overwrite, never a
  coin-flip.

Deltas are small patch packages declaring `base_package_id` + `base_version`, plus per-section byte
patches. Correcting one fact must never require shipping a new full release.

## G. Density budget

**Target ≤ 1.5 bytes/fact; ≤ 2.5 acceptable.** Sources of density:

| Source | Saving |
|---|---|
| Balradix-coded compact IDs | 12 trits ≈ 19 bits vs a 64-bit hash |
| Polarity + strength | 2 bits polarity + 4 bits Baldec (range −5..+5) |
| Provenance | ~8 bits (page + extraction id) |
| Trinary→bit packing | 3→5, 5→8, 8→13, 12→20 bits (up to 1.667 bits/trit realized vs 1.585 theoretical) |
| Per-page entropy coding | entropy coding over Balradix digit streams |
| Entity dedup / alias collapse | `1` / `one` / `unity` become one id |
| **Schema compression** | most facts stored as *applications of rules*, not raw triples |

**Schema compression is why rule induction is on the critical path**, not a later extra: without it the
10x claim is unreachable. And be honest about the boundary — the 10x holds for **schema-heavy, curated,
verified knowledge**, not for open-domain trivia breadth.

## H. `.tarchi` vs an LLM weight file

| Dimension | `.tarchi` package | LLM weight file |
|---|---|---|
| Knowledge form | explicit, cited facts + derivation traces | implicit, diffuse weights |
| Storage (100B-equiv.) | ~5 GB | ~50 GB (INT4-equiv.) |
| Retrieval | exact lookup + derivation path | probabilistic, no citation |
| Correction | edit one node → small delta | full retrain → new blob |
| Verification | hash + semantic re-verification on load | impossible |
| Coverage | high precision, curated domain | excellent recall, noisy |
| False claims | quarantined, never asserted | fluent and confident |
| Distribution | composable domain packages | one giant blob |

## I. Header layout

Valid, compiling Rust:

```rust
pub const MAGIC: [u8; 8] = *b"TARCHI01";

#[derive(Clone, Copy, Debug, PartialEq, Eq)]
#[repr(u8)]
pub enum SectionKind {
    Header    = 0,
    Axiom     = 1,
    Entity    = 2,
    Fact      = 3,
    Rule      = 4,
    Index     = 5,
    Quarantine = 6,
    Integrity = 7,
}

#[derive(Clone, Copy, Debug)]
#[repr(C)]
pub struct SectionDesc {
    pub kind: SectionKind,
    /// Balradix-coded byte offset (variable width, up to 8 trits).
    pub offset: [u8; 8],
    /// Balradix-coded byte length.
    pub length: [u8; 4],
}

#[derive(Clone, Debug)]
#[repr(C)]
pub struct HeaderV01 {
    pub magic: [u8; 8],          // b"TARCHI01"
    pub version: u16,            // 0x0001
    pub flags: u16,              // bit0 signature present, bit1 entropy-coded pages
    pub tridak_base: u8,         // 3
    pub header_len: u16,         // total header bytes including the section table
    pub reserved: u32,           // must be zero
    pub axiom_set_hash: [u8; 32],// Blake3 of the Axiom section
    pub package_id: [u8; 20],    // Balradix-coded canonical identifier
    pub provenance_len: u16,     // bytes of the UTF-8 provenance string
    pub license_id: u16,         // 0 = unspecified, 1 = MIT, 2 = CC0, 3 = proprietary
    pub section_count: u16,      // 8 in v0.1
    pub sections: [SectionDesc; 8],
}
```

Byte offsets of the header fields (before the section table):

| Offset | Size | Type | Meaning |
|---|---|---|---|
| 0 | 8 | `[u8; 8]` | magic `TARCHI01` |
| 8 | 2 | `u16` | format version |
| 10 | 2 | `u16` | flags |
| 12 | 1 | `u8` | tridak base |
| 13 | 1 | `u8` | reserved |
| 14 | 2 | `u16` | header length |
| 16 | 4 | `u32` | reserved |
| 20 | 32 | `[u8; 32]` | axiom-set hash (Blake3) |
| 52 | 20 | `[u8; 20]` | package id (Balradix) |
| 72 | 2 | `u16` | provenance length |
| 74 | 2 | `u16` | license id |
| 76 | 2 | `u16` | section count |
| 78 | 2 | `u16` | reserved (alignment) |
| 80 | 8 × 16 | `[SectionDesc; 8]` | section descriptor table |

The header is padded/aligned to 64 bytes, and the section table follows. Section offsets and lengths are
Balradix-coded for compactness; a reader that meets an unknown `SectionKind` skips it (treats length 0).

## J. Worked numbers

### J.1 Sample package (a sample, not the full Astra)

1,000,000 facts · 50,000 entities · 12,000 aliases · 300 axioms · 5,000 rules · 40,000 quarantined claims.

| Section | Count | Unit cost | Size |
|---|---|---|---|
| Axiom | 300 | ~1 KB | 300 KB |
| Entity | 50,000 | ~40 B | 2.0 MB |
| Alias | 12,000 | ~20 B | 240 KB |
| Fact | 1,000,000 | 1.5 B | 1.5 MB |
| Rule | 5,000 | ~30 B | 150 KB |
| Index | ~1.1 M entries | ~3 B | 3.3 MB |
| Quarantine | 40,000 | 2 B | 80 KB |
| Integrity | ~700 pages | 32 B/page | 22 KB |
| **Total** | | | **≈ 7.6 MB** |

This is a **sample** package on the order of megabytes — not the full Astra.

### J.2 Full 3.7-Astra package

~3.3 billion facts at the same per-fact density ⇒ **~5 GB**, against **50 GB** INT4-equivalent for a
100B-param reference: the **10x** reduction.

Working-set story on a 2 GB RAM device:

| Component | Resident |
|---|---|
| Package | mapped read-only from flash — **not** resident |
| Hot set (axioms + frontier-pinned derivation pages) | ≤ ~1.6 GB |
| Engine (Rust, arena, no GC) | ≤ 200 MB |
| **Total resident** | **≤ 2 GB** |

## K. Versioning and migration

- `version` in the header: readers **reject an unknown major version**, accept a newer minor.
- Section kind ids are extensible; unknown kinds are skipped by readers.
- Adding a new section kind is **non-breaking**. Changing the layout of an existing kind requires a new
  major version.
- Delta packages declare `base_package_id` (Balradix) and `base_version` (u16); a reader must refuse a
  delta whose base does not match, and must never apply a delta to a different base silently.

## L. Tooling

```bash
tarchi pack --input corpus/ --base base.tarchi --output pkg.tarchi
tarchi inspect pkg.tarchi                 # summary + per-section sizes
tarchi show pkg.tarchi axiom              # dump a section
tarchi verify pkg.tarchi                  # hashes + Rubik re-verification
tarchi compose base.tarchi domain.tarchi personal.tarchi -o merged.tarchi
tarchi diff old.tarchi new.tarchi -o patch.tarchi
tarchi stats pkg.tarchi                   # density, bytes/fact, working-set projection
tarchi bench pkg.tarchi                   # Astra measurement (reference-LLM harness)
```

All operations are **deterministic**; every output carries hashes and provenance for auditability.

## M. Validity checklist — what `tarchi verify` checks

1. **Per-page Blake3** matches the Integrity section.
2. **Axiom re-verification** — every fact is re-checked against the axiom set; offending facts are
   flagged and logged.
3. **Index consistency** — every referenced entity id exists; every referenced fact id is present.
4. **Alias canonicalization** — each alias resolves to exactly one canonical id; no alias appears under
   two entities.
5. **Quarantine separation** — no quarantined claim appears in the active Fact section.
6. **Orphan references** — no fact references a non-existent entity.
7. **Header invariants** — magic, version, reserved fields, and the section table's offsets/lengths are
   within the file and non-overlapping.

Verification failure ⇒ the package loads **degraded**; affected facts are quarantined and excluded from
inference. It never loads silently.

## See also

`charter.md` · `tridak.md` · `nubit.md` · `decisions/ADRs.md` · `PLAN.md` · `../discussion/discussion.json`
