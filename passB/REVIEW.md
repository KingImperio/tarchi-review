# TArchI — Review Pass B (contradiction + hidden-assumption hunt)

**Read this file, then do the work. Do not ask for more input.**

## What you are doing

You are the adversarial reviewer of a new architecture. Your value is not agreement — a review that
finds nothing is a failed review. Be the engineer who has to ship this on a 2 GB Android phone and
has to defend every claim in public.

This is **Pass B**. You get the creator's raw intent, the design decisions already locked
(below), and **read access to the repository** at `/home/fola/tarchi`. Read the repository files
before answering; do not review from this summary alone.

**Repository reading order** (your workspace is `/home/fola/tarchi`):

1. `README.md` — front door, 50 lines
2. `docs/charter.md` — thesis, the four trit roles, the 27 states, the capability model (Astra),
   the honest limits, the non-goals
3. `docs/tridak.md` — Tridak/Balradix/Baldec, carry rules, packing tables, Kleene truth tables,
   property-test checklist
4. `docs/nubit.md` — Nubit struct, Rubik verifier, Chess planner, `Q_baldec` derivation, derivation
   traces, worked example
5. `docs/package-format.md` — `.tarchi` sections, header byte layout, density budget, worked
   numbers, versioning, validity checklist
6. `docs/decisions/ADRs.md` — **D01–D20 with context/decision/consequences/alternatives, plus the
   3 open questions. This is the main object of the review: test each D-number.**
7. `discussion/discussion.json` — the machine-readable decision record: 20 decisions, 13 glossary
   entries, 4 narrative stages, 11 technical facts, 3 open questions, 5 non-goals, 5 honest limits
8. `docs/PLAN.md` — read **last**, only to check that the docs and the plan agree; if they
   disagree, that disagreement is itself a finding

Do not read, and do not need: `crates/`, `bench/`, `corpus/`, `domains/`, `tests/` — all empty
scaffolding. Do not read `.git` internals.

**Test each locked decision:**

- Does it hold under the intent?
- Does it hold at 3.3B facts on a 2 GB device?
- Does it contradict another decision?
- List every **unstated assumption** the design depends on.
- List every place the design is **self-consistent but wrong**.

**Output format (use for every finding):**

```
F<NN>  [fatal | major | minor | optional]
  Claim being tested:
  What breaks (be concrete — a scenario, numbers, a code path):
  Why our reasoning fails (or where it is merely unproven):
  Recommended change (concrete enough to implement):
  Does this overturn a locked decision? (which D-number, or "no")
  Cost of the change (memory, time-to-first-answer, complexity, or scope)
```

End with: **Top 5 findings**, **Decisions I would overturn** (D-number + one line each), **Decisions I
would keep and why**, and a **one-paragraph verdict**.

## Ground rules

- Challenge the foundations. The creator explicitly wants foundations challenged. If the no-LLM
  constraint, the 10x size claim, the Astra metric, or the trinary premise is wrong, say so plainly
  and quantify the damage.
- But do not stop at "that's hard" — give the alternative and its cost.
- Do not invent requirements the intent does not imply.
- **No code implementation.** Sketch at most.
- You are a reviewer, not a builder. Do not edit, write, or scaffold anything in the repository.
  Read-only.

---

---

## Part 1 — The raw intent (verbatim)


These are the creator's own words. Do not assume anything else. Re-derive an architecture from them.

> "I wanna create a new type of Intelligence. TArchI[True Archaion Intelligence]. And I wanna use a Triune
> System of -1,0, and 1. Instead of 0 and 1 bits."

> "Archaion simply means Intelligence from the beginning. True means True- as in truly intelligent and
> knowledgable and 'real'. I wanna create an Intelligence, which comes bundled with its own architecture and
> the other important stuff within."

> "My goal is to reduce the amount of stuff and most importantly, reduce size and increase speed and
> capabilities and eradicating weaknesses. I wanna be able to run the equivalent of a 100B model on a 2GB
> android phone. Thats why i suggested -1,0 and 1 instead of bits."

> "Tridaks and Nubits" — inspired by Rubik's cube: "to define and project all possible states/predictions/
> deductions/probabilities of whats next". "moving between 0 and 1 is not bad, we have been using it since
> forever. But scientists are trying to create Qubits where things can exist in 0s and 1s simultaneously. But
> that is simply wrong to me… I am only using -1, 0 and 1 for a test but if we can use -9 to 0 to 9 then that
> itself would be fantastic. Not decimals but a new form whose name we can think up."

> "Nubits, the ones i said are inspired by Rubiks. Do you know how many million possible states a rubik has
> compared to a chess board? Rubiks is lesser but its also better." — Rubik = closed, deterministic, tiny
> vocabulary of legal moves, colossal state space. Chess = branching, adversarial, value-estimating.
> Synthesis: Chess for macro strategy (which branch), Rubik for micro execution (shortest deterministic
> path).

> "AND REMEMBER: WE WILL NEVER USE LLMs in our system."

> "1 Astra is the equivalent of 27B Params worth of an LLM Model." Also: an Astra may be "a unified
> something" — a single portable artifact.

> From the prior art (`oracule-core`, a Python prototype): a knowledge graph whose nodes carry a triune
> tension tensor `T = [info, contra, reframe]` as continuous floats, with node states
> FOUNDATIONAL/ACTIVE/RESOLVED/EXHAUSTED, edge types factual/resonance/expansive, a Matter persistence
> layer, graduation rules, and a `Q(t)` quality score. A human-oversight scenario injects a contradiction
> and the system must self-correct.

**Hard constraints implied by the intent:**
- Numeric substrate is trinary: `-1, 0, +1` (balanced), extensible to a wider balanced radix.
- Operates on a **2 GB RAM** Android phone. Knowledge artifact lives on flash.
- **No LLMs, ever, in any layer.**
- Self-sufficient: perceive, reason, generate, act, remember, learn, self-correct.
- Must be *true* — every claim verifiable and citable; hallucination structurally impossible, not merely
  discouraged.

---


---

## Part 2 — What we have decided (locked decisions)


### 2.1 Substrate and roles
- **Tridak** = base-3 balanced digit `{-1,0,+1}`; atomic unit of state AND of weight load/offload.
- **Balradix** = family of balanced-radix numeral systems (base-B digits in `[-⌈B/2⌉, ⌊B/2⌋]`).
  **Baldec** = base-10 balanced digit `{-5..+5}`, used for edge strength and confidence.
- `log2(3) = 1.585` bits/trit; integer packing of k trits into `⌈k·1.585⌉` bits (3→5, 5→8, 8→13,
  12→20 bits) realizes up to 1.667 bits/trit; 8 trits address 6561 states in one byte.
- **Four orthogonal trit roles**, one per layer, not one overloaded axis:
  | Role | Layer | Values |
  |---|---|---|
  | Truth | proposition | -1 false / 0 unknown / +1 true (Kleene strong 3-valued logic) |
  | Polarity | edge | -1 opposes / 0 undetermined / +1 supports |
  | Epistemic | ledger | -1 rejected / 0 candidate / +1 held |
  | Balance | charge | -1 deficit / 0 equilibrium / +1 surplus (drives decay, stall, frontier) |
- Node = `(truth, epistemic, balance)` = **27 states**, mapped to lifecycle labels FOUNDATIONAL /
  ACTIVE / RESOLVED / EXHAUSTED / QUARANTINED.

### 2.2 Reasoning
- **Nubits** = the unit of projection: one slot in a state's fan-out of legal next states
  `{target, polarity, confidence, legality}`. Project ALL legal successors at once, not one next token.
- **Rubik layer** = deterministic verifier: every projected step is checked against hard substrate
  constraints; illegal states are unrepresentable, so hallucination is a type error.
- **Chess layer** = adversarial planner: best-first search over the Nubit fan-out, structural value
  heuristic, no learned policy net.
- Quality `Q` mirrors the prototype's `compute_Q` (coverage, chain completeness, contradiction absence)
  but trinary: three trits → `Q_baldec = round(5 · mean)` ∈ [-5,+5].
- A resolution produces a **derivation trace** (ordered steps, each citing edge/Nubit/axiom) — this is
  what makes claims citable and what the Realizer renders.

### 2.3 Knowledge acquisition — the "Curriculum" (replaces gradient descent)
corpus → deterministic template extraction (patterns + classical classifier) → canonicalization (entity
resolution, alias collapse, Balradix ids) → verification against axioms (Rubik layer) → schema/rule
induction (ILP precedent: Aleph/Progol) → promotion with Baldec strengths → per-session decay →
graduation (stability + redundancy + load-bearing) → **quarantine** for low-confidence or contradicting
claims. Consequence: no training step and no "model" in the LLM sense — the substrate IS the knowledge;
learning is continuous, GPU-free, no catastrophic forgetting; a wrong fact is fixed by editing one node.

### 2.4 The package (`.tarchi`) = TArchI's "weights"
4 KiB mmap-able pages; Balradix-coded page table (sub-byte addressing); Tridak = atomic load/offload unit;
hot pages pinned by the reasoning frontier, cold evicted; residency is a reasoning decision, not LRU.
Sections: **Header, Axiom, Entity, Fact, Rule (reserved, empty in v0.1), Index, Quarantine, Integrity**.
Two-tier integrity: per-page Blake3 (hash) + re-verification of every fact against the axiom set on load
(semantic). Composable (base + domain + personal), deterministic merge, delta packages.

### 2.5 The Astra unit and the size claim
- **1 Astra** = the verifiable **capability** of a 27B-param reference LLM, measured on TWO suites:
  knowledge recall (verifiable questions, distractors, injected contradictions, scored with citations)
  AND derivation (multi-step questions scored on the derivation path). 3.7 Astra = 100B-param-equivalent.
- Size (INT4 yardstick, 0.5 B/param): 1 Astra → 1.35 GB vs 13.5 GB reference; 3.7 Astra → ~5 GB vs
  50 GB. Ratio 10x. Must operate in ≤ 2 GB resident working set, engine ≤ 200 MB.
- Density budget ≤ 1.5 bytes/fact (≤ 2.5 acceptable) vs LLM marginal ~10–15 bytes/fact. Density comes
  from: Balradix ids, 2 bits polarity + 4 bits strength, ~8 bits provenance, trinary→bit packing, per-page
  entropy coding, alias dedup, and **schema compression** (facts stored as rule applications).
- State-space density examples: Rubik's cube 4.3252e19 states = 66 bits vs 42 trits; chess ~4.8e44 = 149
  bits vs 94 trits.

### 2.6 Stack and milestones
Rust core (no_std-capable, arena, no GC; Android NDK + WASM targets) + Python research bindings
(never the runtime). Crates: `tridak, substrate, axioms, nubit, curriculum, lexicon, realizer, package,
engine, tarchi-cli, tarchi-py`. Milestones M0 (specs, done) → M7 (induction, LSP perception, MCP action).
`oracule-core` frozen as the binary/float **control group** for the trit-vs-bit A/B (its bug:
`learning.py:211` uses `ACTIVE` without importing it).

### 2.7 Environment reality
Rust toolchain present; Python 3.12 + networkx/numpy; Node present; **no Android SDK locally**; Codex
CLI 0.146.0 (stale, default model retired); `gh` authenticated as `KingImperio`.

---


---

## Part 3 — What the reviewer must not do


- Do not implement. This is a design review.
- Do not recommend adding an LLM, a hosted API, or a network dependency — the no-LLM constraint is a
  founder-level decision, not a reviewer's to overturn silently (challenge it explicitly instead).
- Do not assume hardware, funding, or team size we have not stated.
- Do not propose features the intent does not need. The intent is: small, fast, verifiable, on a phone.
- Do not rewrite the intent in your own words and review that. Review *this* intent.

---

