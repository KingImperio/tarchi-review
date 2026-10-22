# TArchI — True Archaion Intelligence

A trinary (-1, 0, +1) intelligence: engine + portable knowledge package ("Astra"), designed to carry
**100B-param-equivalent knowledge operable on 2 GB RAM**.

## Context

`oracule-core` (Python/networkx) already prototypes a triune tension tensor `T = [info, contra, reframe]`
as continuous floats, plus Matter persistence, graduation rules and `Q(t)` scoring
(`engine/dynamics.py:22`, `learning.py:173`, `main.py:137`). It is a float/binary-era prototype.
TArchI is the successor: a **non-parametric, trinary, self-verifying** intelligence that reasons by
**derivation** (search + verification over explicit state) rather than by gradient descent, and that
**never uses LLMs** — at any layer, ever.

Key insight driving the architecture: an LLM's bytes are mostly redundancy, syntax and unverified
trivia, not facts. An explicit, deduped, schema-compressed, verified substrate is **denser per unit of
knowledge** than parametric storage, and every claim it makes is citable and checkable.

## Locked decisions

| Decision | Value |
|---|---|
| Location | New standalone project `/home/fola/tarchi`, its own git repo (home dir is itself a repo — do not commit tarchi into it) |
| Numeric substrate | Trinary: Tridak `-1/0/+1`; Balradix family (Tridak = base-3 digit, Baldec = base-10 balanced digit `-5..+5`) |
| Trit roles | 4 orthogonal, layered: **Truth** (proposition), **Polarity** (edge), **Epistemic** (ledger), **Balance** (charge) |
| Runtime | Standalone & offline-first. No LLM in any layer. Rust core + Python research bindings |
| Knowledge unit | **1 Astra = 27B-param-equivalent capability**, measured on knowledge recall **and** derivation suites |
| Package | `.tarchi` package = TArchI's "weights": axioms + rules + facts + index + quarantine + integrity, paged, composable, delta-able |
| Size target | 1.35 GB per Astra; 3.7 Astra (100B-equivalent) ≈ **5 GB package**, **≤2 GB resident working set** |
| Generation | Constructive Realizer + `Realizer` trait adapter. No LLM renderer |
| Learning | Curriculum: extraction → canonicalization → verification → promotion → decay (+ minimal schema mining in v0.1) |
| v0.1 scope | Closed-domain resolve + explain + cite, curated corpus with ground truth |
| Validation | TArchI-native metrics + bit-vs-trit A/B (oracule-core frozen as binary control) |
| `oracule-core` | Frozen as the binary control group; fix `learning.py:211` `ACTIVE` NameError first; later upgraded to consume `.tarchi` packages |

## Non-goals (v0.1)

LLMs of any kind · open-domain QA · conversational/charming prose · image/audio perception ·
code perception via LSP · multi-user/cloud · ILP induction beyond minimal schema mining.

## Architecture

```
 Perception ──────────► Substrate (trinary, paged) ──────────► Realizer ──► Output
 lexicon/parser        axioms · rules · facts · index          discourse plans
 extraction            polarity/strength/truth/epistemic/balance
        │                       │
        └─ Curriculum ──────────┘        Nubit projection (Rubik verify + Chess search)
        (extract→canon→verify→promote)   legality · fan-out · value · derivation trace
```

**Role mapping** (one mechanism, four readings — the "all four" resolution):

- Truth trit: `-1` false / `0` unknown / `+1` true → Kleene strong 3-valued logic.
- Polarity trit (edge): `-1` opposes / `0` undetermined / `+1` supports.
- Epistemic trit (ledger): `-1` rejected (quarantined) / `0` candidate / `+1` held.
- Balance trit (charge): `-1` deficit / `0` equilibrium / `+1` surplus → drives decay, stall, frontier.

Node = `(truth, epistemic, balance)` = **27 states**, mapped deterministically to lifecycle labels:
`FOUNDATIONAL` (held + in axiom set + equilibrium) · `ACTIVE` (held + non-zero charge) ·
`RESOLVED` (truth +1, equilibrium, path grounded in a FOUNDATIONAL node) · `EXHAUSTED` (held, truth 0, deficit) ·
`QUARANTINED` (epistemic −1). The mapping lives in **one source of truth** and is property-tested both ways.

**Balance aggregation** (Baldec strengths, sums quantized):
`support = Σ strength of +1 edges`, `oppose = Σ strength of −1 edges`;
truth = `+1` if support > oppose and support > 0; `−1` if oppose > support and oppose > 0; else `0`.
charge = sign(support − oppose).

**Q (quality)** — mirrors `oracule-core/engine/dynamics.py:129` so the A/B is apples-to-apples, but trinary:
`coverage_t`, `chain_t`, `contra_t` ∈ {−1,0,+1}; `Q_baldec = round(5 × mean)` ∈ [−5,+5]. The float control
keeps the original formula on the same graphs.

**Nubits** (Rubik-inspired projection): a node's fan-out of **all legal next states**, each Nubit =
`{target, polarity, confidence: Baldec, legality}`. Rubik layer = deterministic verifier (illegal states are
unrepresentable → hallucination is a type error, not a probability). Chess layer = best-first search over the
fan-out with a structural value heuristic (grounding, chain completeness, contradiction absence).

**Curriculum** (the training analogue): corpus → lexical/syntactic parse → template extraction → confidence
model → canonicalization (entity resolution, alias collapse, Balradix IDs) → verification against axioms →
promotion with Baldec strength → decay per session → graduation (stability + redundancy + load-bearing,
ported from `oracule-core/learning.py:173`) → schema mining into the Rule section. Low-confidence or
contradicting facts go to **quarantine** and never enter the active substrate.

**Numeral system (Balradix)**: base `B` balanced digits in `[−⌈B/2⌉, ⌊B/2⌋]`. Base-3 carry: sum `s ∈ [−2,2]`,
`carry = sign(s)` if `|s| > 1` else `0`, digit = `s − 3·carry`. Tridak packing: `k` trits → `⌈k·log₂3⌉` bits
(3→5, 5→8, 8→13, 12→20 bits ⇒ up to 1.667 bits/trit realized vs 1.585 theoretical).

## Package format `.tarchi` (Astra)

4 KiB pages; `mmap`-able; Tridak = atomic load/offload unit; Balradix-coded page table ⇒ sub-byte addressing
(8 trits = 6561 states per byte).

| Section | Contents | v0.1 |
|---|---|---|
| Header | magic, format version, tridak base, axiom-set hash, package id (Balradix), provenance, license, section table | ✅ |
| Axiom | foundational axioms + rule forms (trinary-encoded) | ✅ |
| Entity | canonical entity table, type, aliases (compressed) | ✅ |
| Fact | triples: Balradix (s,p,o) + polarity trit + strength Baldec + provenance id | ✅ |
| Rule | derived/induced generalizations | reserved, **must exist from day one** |
| Index | entity→facts, subject→facts, predicate→facts, trinary offsets | ✅ |
| Quarantine | rejected/contradicted claims — audit only, never used in inference | ✅ |
| Integrity | per-page Blake3, optional signature block | ✅ |

Density budget: **≤1.5 bytes/fact target, ≤2.5 acceptable** (drives 3.7 Astra ≈ 5 GB).
Composition: base + domain + personal packages merged deterministically by (canonical id, polarity, strength);
deltas ship as small patch packages. Integrity is two-tier: hash proves bits unchanged, and the Rubik
verifier **re-verifies every fact against the axiom set on load** — semantic integrity, impossible for weights.

**RAM operating model**: package on flash (5 GB), resident working set ≤ 1.6 GB, engine overhead ≤ 200 MB
(arena-allocated, no GC — Python is bindings-only, never the runtime). Pages holding axioms and the active
derivation chain are **pinned by the reasoning frontier**, not by LRU.

## Module layout

```
/home/fola/tarchi/
├── Cargo.toml                       # workspace
├── crates/
│   ├── tridak/      # Trit, BalWord<B>, Baldec, packing, no_std
│   ├── substrate/   # Node/Edge/Triple, graph, canonicalizer, paged store, quantizer
│   ├── axioms/      # axiom set, rule forms, semantics, 27-state mapping
│   ├── nubit/       # projection, legality (Rubik), planner (Chess), value heuristic
│   ├── curriculum/  # extraction templates, confidence model, verifier, promotion, decay, graduation, schema mining
│   ├── lexicon/     # lexicon, morphology, parser (perception)
│   ├── realizer/    # discourse plans, templated surface form, Realizer trait
│   ├── package/     # .tarchi read/write, pages, index, integrity, compose, deltas
│   ├── engine/      # ingest/query/explain orchestration, hot-set paging, frontier pinning
│   ├── tarchi-cli/  # ingest | ask | explain | inspect | verify | bench | pack
│   └── tarchi-py/   # pyo3 bindings (cargo feature, never the runtime)
├── docs/            # charter.md, tridak.md, nubit.md, package-format.md
├── domains/         # ported from oracule-core/data/domains
├── corpus/          # curated corpus + ground-truth claims
└── bench/           # A/B harness: trit vs float, Astra measurement
```

## Milestones

- [ ] **M0 — Charter & specs.** `docs/charter.md` (axioms, semantics, role mapping, explicit limits: no
      open-domain recall, no implicit generalities, no creative prose), `tridak.md`, `nubit.md`,
      `package-format.md`. Property tests for the trit algebra (Kleene truth tables, negation involution,
      Balradix carry round-trip). **Exit:** specs reviewed; algebra tests pass.
- [ ] **M1 — Core crates.** `tridak`, `substrate`, `axioms`, `nubit`, `engine` skeleton, CLI skeleton.
      **Exit:** build a domain graph, project Nubits, verify legality, resolve a query and print a derivation
      trace, all in RAM.
- [ ] **M2 — Package format.** `.tarchi` read/write, all sections, integrity, compose/merge, deltas.
      **Exit:** round-trip tests; **tamper test** (corrupt one page → detected + fact re-verified); merge
      determinism; per-section byte-density report.
- [ ] **M3 — Curriculum.** `lexicon` (lexicon/morphology/parser), extraction templates, classical confidence
      model, canonicalizer, verifier, promotion, quarantine, decay, graduation, **minimal schema mining**
      (Rule section non-empty). **Exit:** ingest curated corpus → facts with provenance; measured
      precision/recall vs ground truth; quarantine rejects known-bad claims.
- [ ] **M4 — Closed-domain loop.** Port the 4 oracule-core domains; curated corpus + ground truth; CLI
      `ingest/ask/explain/inspect`; minimal Realizer (templated answers with citations + derivation trace).
      **Exit:** end-to-end resolve + explain + cite; **unsupported-claim rate = 0** on the evaluated set.
- [ ] **M5 — A/B + Astra measurement.** Freeze oracule-core as control (fix `learning.py:211` ACTIVE
      NameError); port same domains/queries; implement the reference-LLM harness (dev-time) for the knowledge
      and derivation suites; compute the Astra rating of a sample package.
      **Exit:** metrics table with trit-vs-float deltas on identical formulas; ≥1 Astra-rated package.
- [ ] **M6 — Device.** WASM build + PWA shell (**"Code by Oracule" fixed bottom-left faint signature — standing
      rule**) runnable on a real 2 GB Android phone; then Rust arm64 via NDK.
      **Exit:** measured working set ≤2 GB, cold start, page-fault count; largest package built operates on
      the device.
- [ ] **M7 — Induction & generality.** Full ILP, rule induction from instances, more domains, LSP code
      perception adapter, MCP action adapter. **Exit:** bytes/fact trending to ≤1.5; derivation suite improves.

## Validation

Metrics (all reported for both trit engine and float control where applicable):

1. **State density** — bits/trits per distinguishable state (cube 66 b vs 42 trits; chess 149 b vs 94 trits).
2. **Knowledge density** — verified facts per KB resident; bytes/fact (target ≤1.5).
3. **Latency** — ingest ms/page, resolve ms/query, inferences/sec.
4. **Correctness** — resolution accuracy on the ground-truth claim set.
5. **Contradiction handling** — detection recall/precision on injected contradictions; self-correction rate;
   oversight-injection scenario ported from `oracule-core/main.py:137`.
6. **Citation coverage** — answers carrying a derivation path; unsupported-claim rate (target 0).
7. **Footprint** — RSS, cold start, working-set size, page-fault count (device).
8. **A/B deltas** — identical formulas, two representations; report per-metric trit-vs-float.
9. **Astra rating** — package vs reference LLM on the knowledge suite **and** the derivation suite.

Astra measurement needs a reference model (27B): candidate **Gemma-2-27B** (exact 27B; license-gated),
alternatives Qwen2.5-32B / Mistral-Small-24B. Reference runs dev-time only, never on device.

## Risks

- **Bytes/fact > 2.5** (schema compression insufficient) ⇒ 10× target unreachable; mitigation: M3 ships
  schema mining early; if M5 confirms the miss, pull M7 induction forward and re-scope the Astra size claim.
- **Derivation suite may not match the reference** ⇒ if derivation parity fails, split the unit into
  "knowledge-Astra" and "reasoning-Astra" rather than overstate.
- **Extraction recall** on prose is the weakest link ⇒ curated corpus with ground truth; domain templates;
  quarantine absorbs the long tail.
- **No implicit generality** ⇒ TArchI cannot match an LLM on open-domain fluency; stated as a limit in the
  charter, not hidden.
- **Android SDK absent locally** ⇒ M6 blocked until `cmdline-tools` + NDK installed (user action); WASM/PWA
  path covers the demo meanwhile.
- **Reference-LLM license/download budget** for Astra measurement.
- **Python overhead** ⇒ runtime is Rust only; Python is bindings/benchmarks.
- **Home dir is a git repo** ⇒ `/home/fola/tarchi` gets its own repo; never commit it into the parent.

## Open questions (resolve at implementation start)

1. Which reference LLM defines 1 Astra (license acceptance + download budget)?
2. Which realization registers ship in v0.1: answer · explain · contradiction notice · compare · summary?
3. Confirm the browser/PWA as the first device target (recommended) before investing in the NDK path.

## Future integration

`oracule-core` later becomes a consumer of `.tarchi` packages (same domain schema, shared corpus), so its
float engine and TArchI's trinary engine can be A/B'd live, and its domains stay authoritative for both.
