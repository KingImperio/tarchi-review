# Nubit.md - Core Inference Primitive for TArchI
# draft, 2026-10-21

## 1. Nubit Definition

```rust
// Nubit: unit of PROJECTION — one slot in a state's fan-out of possible legal NEXT states
struct Nubit {
    target: NodeId,      // The resulting state node ID if this Nubit is selected
    polarity: Trit,       // Supports (+1), opposes (-1), or undetermined (0)
    confidence: Baldec, // Integer trust: [-5, +5] (Baldec scale)
    legality: Legality,  // First-class verdict: Legal or Illegal with rule reference
}

enum Legality {
    Legal,
    Illegal { rule: &'static str }, // Any violation is unrepresentable at runtime
}

// A set of projected successors from one origin node
struct NubitSet {
    origin: NodeId,
    slots: Vec<Nubit>,  // Exact fan-out; all members are checked before use
}
```

**Field semantics:**
- `target`: Hash of the successor state (deterministic mapping from substrate transition)
- `polarity`: Trit representing the Nubit's stance toward the anchor (-1 oppose, 0 undetermined, +1 support)
- `confidence`: Baldec integer confidence from substrate axioms, derived from derived certainty
- `legality`: First-class field preventing unrepresentable states; illegal Nubits are excluded from consideration entirely

**Why first-class legality?** Violation becomes a type error rather than a probabilistic hallucination. Illegal states are pruned from the universe before they can be reasoned about, preserving semantic integrity.

## 2. Why Project Everything

**Contrast with LLM autoregression:** LLMs predict ONE next token sequentially from a probability distribution, sampling a single path through time. They cannot parallelize over the full successor set because each prediction conditions on the previous sample.

**Contrast with masked diffusion:** Diffusion models predict ALL masked positions in parallel over a token distribution, but lack explicit legality constraints. They can generate tokens that violate substrate invariants, with no mechanism to rule them out without sampling and rejection.

**TArchI projection is third kind:** DETERMINISTIC enumeration of REACHABLE states with polarity and confidence, computed directly from the substrate. Every successor is checkable against hard constraints.

**Cost/benefit:**
- **Bounded fan-out:** Limited to local neighborhood (legal moves from current state)
- **Parallel computation:** Full set computed simultaneously
- **Checkable members:** Each Nubit's legality is verifiable before use
- **Analogy:** Rubik's position's legal-turn set = NubitSet; Chess position's legal-move set = branching strategy

## 3. The Rubik Layer — Deterministic Verifier

The Rubik Layer validates EVERY projected Nubit against hard substrate constraints before any reasoning occurs.

**Constraint sources:**
- Axiom set (semantic integrity)
- Type constraints (node/edge topology)
- Uniqueness (no duplicate NodeIds)
- Structural invariants (e.g., no cycles in DAG)

**Verification procedure:**
1. For each Nubit in a NubitSet, apply all constraint checks
2. If any constraint fails, mark Nubit as Illegal with rule ID
3. Only Legal Nubits survive to Chess planning layer

**Violation reporting:** Illegal Nubits carry rule ID allowing precise citation. Example: `Illegal { rule: "axiom:symmetry" }`

**Re-verification on load:** Every fact in a .tarchi package undergoes full re-verification against current axiom set. Package integrity is hash-verifiable, semantic integrity is constraint-verifiable (weights can never have semantic weight — only hash weight).

## 4. The Chess Layer — Adversarial Planner

Best-first search over Nubit fan-out with structural value estimation, no learned policy network.

**Value function Q(t) (trinary adaptation):**
```rust
// Three trit components, each -1/0/+1
struct Coverage { fraction_of_anchor_neighbors_grounded }
struct ChainCompleteness { exists_grounded_path_from_foundational_node }
struct ContradictionAbsence { no_active_conflict_on_path }

// Convert to Baldec [-5,+5]
fn compute_q(coverage: Trit, completeness: Trit, contradiction: Trit) -> Baldec {
    let sum = coverage as i8 + completeness as i8 + contradiction as i8;
    (5 * sum / 3).clamp(-5, 5)  // round(5 * mean(components))
}
```

**Planner components:**
- **COVERAGE:** Fraction of anchor's neighbors that are grounded/held
- **CHAIN COMPLETENESS:** Existence of complete grounded path from FOUNDATIONAL node
- **CONTRADICTION ABSENCE:** No active conflict on the path

**Planner vs Optimizer split:**
- **Rubik = Micro execution:** Shortest deterministic path within a branch (selection of successor states)
- **Chess = Macro strategy:** Which branch to explore (value-based ordering)

## 5. Derivation Traces

A resolution is a PATH, not just an answer. Each step is citable:

```rust
struct DerivationStep {
    edge: EdgeId,          // Which Nubit edge was taken
    nubit: Nubit,          // The selected Nubit with polarity/confidence
    axioms: Vec<AxiomId>,   // Axioms that licensed this transition
    timestamp: Instant,
}

type DerivationTrace = Vec<DerivationStep>;
```

**Trace properties:**
- Claims are citable: "See step 3, edge Nubit #7, rule axiom:identity"
- Packages that only memorize answers score zero on derivation suite
- Realizer renders traces into human-explainable explanations

**Example trace (propositional logic):**
```
1. [premise] P (axiom:axiom_001) polarity:+1 confidence:+5 legality:Legal
2. [premise] Q (axiom:axiom_002) polarity:+1 confidence:+5 legality:Legal
3. [inference] P ∧ Q (edge:Nubit_017 polarity:+1 confidence:+4 legality:Legal) using modus_ponens
4. [inference] ¬(P ∧ Q) (edge:Nubit_023 polarity:-1 confidence:+3 legality:Legal) using De_Morgan
5. [contradiction] P ∧ Q ⇒ ⊥ (edge:Nubit_042 polarity:-1 confidence:+5 legality:Legal) contradiction injection
```

## 6. Active Front — The Frontier

**Node selection:** Frontier is the node with highest charge/tension among unexhausted, non-contradicted nodes connected to the anchor.

**Hot-page residency:** Frontier pins its pages (axioms plus active chain) in resident working set. This is a reasoning decision, not LRU cache eviction.

**Process:**
1. Evaluate charge/tension of all frontier candidates
2. Select highest (fixed tie-break rule: smallest NodeId)
3. Materialize relevant axioms and path
4. Push onto working set
5. Begin derivation steps

## 7. Stall, Exhaustion and Contradiction

**EXHAUSTED nodes:** Stop moving when all outgoing Nubits have been explored or have confidence ≤ -5.

**CONTRADICTED nodes:** Cannot resolve until contradiction is discharged via upstream propagation draining the contradiction trit.

**CONTRADICTION injection:** From human oversight channel reopens RESOLVED nodes when new contradiction evidence appears, triggering recomputation of affected subgraphs.

**STALL detection:** Nodes that have no Legal outgoing Nubits or have confidence ≤ -5 enter STALL state.

## 8. Worked Example — Propositional Logic Domain

**Substrate state:**
```rust
// Anchor node A
struct NodeA {
    facts: ["P", "Q"],  // Premises
    inferred: [],        // Empty
    contradictions: [],  // Empty
}
```

**Nubit projection (fan-out):**
```
Nubit_017: target=N_B (P∧ Q), polarity:+1, confidence:+4, legality:Legal (axiom:and_intro)
Nubit_023: target=N_C (¬(P∧ Q)), polarity:-1, confidence:+3, legality:Legal (axiom:De_Morgan)
Nubit_042: target=N_D (⊥), polarity:-1, confidence:+5, legality:Legal (axiom:contradiction)
```

**Rubik verification:** All three Nubits legal — no constraint violations.

**Chess search path:**
1. Evaluate Nubit_017: Q = compute_q(+1, +1, -1) = +1 Baldec
2. Evaluate Nubit_023: Q = compute_q(-1, 0, +1) = -1 Baldec  
3. Evaluate Nubit_042: Q = compute_q(-1, -1, +1) = -2 Baldec
4. Select Nubit_017 (highest Q)

**Resulting derivation trace:**
```
[step1] P∧ Q (Nubit_017) via axiom:and_intro
[step2] ⊥ (Nubit_042) via contradiction injection
[step3] Resolution: P∧ Q ⇒ ⊥ (Nubit_042) with contradiction discharge
```

## 9. Invariants and Property Tests

### checklist for Rust crate `nubit`:

1. **Legality coverage:** Every projected Nubit is either `Legal` or carries a `rule` in `Illegal` variant.

2. **Deterministic fan-out:** Given a substrate state and axiom set, the NubitSet is identical across runs.

3. **Re-verification consistency:** Re-loading a package and re-verifying against axioms reproduces identical legality verdicts.

4. **Trace reproducibility:** Multiple runs with identical inputs produce identical DerivationTrace sequences.

5. **Deterministic frontier selection:** Under fixed tie-break rule, frontier node selection is deterministic across runs.

6. **Illegal exclusion:** No Illegal Nubit appears in any successful DerivationTrace.

7. **Confidence bounds:** Baldec values always within [-5, +5].

8. **Polarity integrity:** Trit values restricted to {-1, 0, +1}.

**Test scenarios:**
- Property-based testing: Generate random substrate states, verify all projections are deterministic
- Exhaustive verification: Small domain where all possible NubitSets can be enumerated
- Trace replay: Execute DerivationTrace independently, verify outcome matches

See also: charter.md, tridak.md, package-format.md, ../discussion/discussion.json
