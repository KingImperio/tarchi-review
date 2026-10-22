# TArchI Charter

**Status:** Draft, 2026-10-21

## Author
Fola Adam (creator)

## Core Thesis

**Archaion = "Intelligence From The Beginning"**
- TArchI starts from foundational axioms and DERIVES conclusions, instead of starting from a corpus and interpolating
- TArchI is NON-PARAMETRIC intelligence. It never uses LLMs — at any layer, ever
- Reasoning is search + verification over an explicit, inspectable state space (classical "knowledge-based + search" school), not gradient descent

**Technical Substrates**
- Numeric substrate: Trinary (-1, 0, +1) Tridak digits, packaged in "Balradix" balanced-numeral family
- Every claim is citable and machine-verifiable; hallucination is a type error, not a probability
- Target: 100B-param-equivalent verifiable knowledge operating on 2 GB of RAM, package on flash, demand-paged

## Four Orthogonal Trit Roles

One mechanism, four readings, each in its own layer:

| Role | Layer | Values | Logic |
|------|-------|--------|-------|
| **Truth** | Proposition level | -1 false, 0 unknown, +1 true | Kleene strong three-valued logic (NOT/AND/OR/IMPLIES) |
| **Polarity** | Edge level | -1 opposes, 0 undetermined, +1 supports | Weighted relationship direction |
| **Epistemic** | Ledger level | -1 rejected/quarantined, 0 candidate, +1 held | Trust verification status |
| **Balance** | Charge level | -1 deficit, 0 equilibrium, +1 surplus | Decay, stall detection, frontier selection |

**Node States**: (truth, epistemic, balance) = 3³ = 27 states

| Lifecycle Label | Truth | Epistemic | Balance |
|----------------|-------|----------|--------|
| **FOUNDATIONAL** | +1 | +1 | 0 | Axiom set entry, equilibrium |
| **ACTIVE** | Any | +1 | ≠0 | Held, non-zero charge |
| **RESOLVED** | +1 | +1 | 0 | Grounded in FOUNDATIONAL |
| **EXHAUSTED** | 0 | +1 | -1 | Held, zero truth, deficit |
| **QUARANTINED** | Any | -1 | Any | Rejected, in quarantine ledger |

## Intelligence Capability Model

Seven first-class components:

1. **Perceive** - Classical NLP pipeline: lexer → morphology → lexicon → constraint parser → semantic extraction
2. **Reason** - Chess-layer search (Nubit fan-out) + Rubik-layer deterministic verification
3. **Generate** - Constructive Realizer from resolved derivation traces, precise and cited
4. **Act** - Tool/MCP-compatible adapter
5. **Remember** - Paged substrate + per-session decay
6. **Learn** - Curriculum: extraction → canonicalization → verification → promotion → decay → graduation → schema mining
7. **Self-correct** - Contradiction injection + upstream propagation, plus human oversight

## Curriculum (Replaces Gradient Descent)

Continuous knowledge growth through:
1. **Corpus extraction** - Deterministic template extraction (patterns + classical classifier)
2. **Canonicalization** - Entity resolution, alias collapse, Balradix-coded IDs
3. **Verification** - Against axiom set (Rubik layer)
4. **Schema induction** - ILP precedent (Aleph/Progol)
5. **Promotion** - Baldec strengths, graduation on stability + redundancy
6. **Decay** - Per-session degradation
7. **Quarantine** - Low-confidence or contradicting facts never enter active substrate

**Key property:** Knowledge grows by adding pages, not re-running gradient descent. Wrong facts fixed by editing one node.

## Knowledge Unit: Astra

**1 Astra** = verifiable capability of a 27B-param reference LLM, measured on:
- Knowledge-recall suite (verifiable questions with distractors, injected contradictions)
- Derivation suite (multi-step questions derived from axioms + rules)

**Critical property:** Database-only memorization scores zero on derivation.

**Size metrics:** 3.7 Astra = 100B-param-equivalent
- Package size: 5 GB vs 50 GB INT4-equivalent
- Target RAM: 2 GB resident working set, package on flash, demand-paged
- Storage efficiency: 1 Astra ≈ 1.35 GB TArchI vs 13.5 GB INT4-equivalent

**Technical specs:**
- log2(3) = 1.585 bits per trit
- Integer packing: ceil(k×1.585) bits (max 1.667 bits/trit)
- 8 trits address 6561 states in one byte
- Rubik's cube: 4.3252e19 states = 66 bits vs 42 trits
- Chess: ~4.8e44 positions = 149 bits vs 94 trits
- LLM storage: 10-15 bytes/fact vs TArchI target 1.5-2.5 bytes/fact

## Honest Limits

- **Trade-offs:** Open-domain recall and fuzzy similarity for verifiability, density, updateability, footprint
- **Generalization gap:** LLM weights carry implicit generalities (grammar, style) that no fact table expresses
- **Storage claim holds** for schema-heavy curated verified knowledge, not open-domain trivia
- **Language limitation:** Precise and cited, but not charming. No open-ended chat or creative prose in v0.1
- **Weakest link:** Extraction recall on prose
- **Fallback:** If 27B reference parity not met, split Astra into knowledge-Astra and reasoning-Astra

## Non-Goals (v0.1)

- LLMs of any kind
- Open-domain QA
- Conversational/charming prose
- Image/audio perception
- Code perception via LSP
- Multi-user/cloud deployment
- Full ILP induction

## Glossary

- **Tridak**: Trinary digit system (-1, 0, +1)
- **Balradix**: Balanced-numeral family packaging Tridak digits
- **Baldec**: Strength metric for promoted facts
- **Nubit**: Chess-layer search fan-out structure
- **Astra**: Knowledge unit, 27B-param-equivalent capability
- **Rubik layer**: Verification against axiom set
- **Chess layer**: Search over Nubit fan-out
- **Curriculum**: Knowledge growth pipeline (extraction → canonicalization → verification → promotion → decay → graduation → schema mining)
- **Substrate**: Underlying data structure + storage management
- **Realizer**: Constructive generation from derivation traces
- **.tarchi package**: TArchI distribution format

## Cross-References

- `tridak.md` - Tridak digit specification
- `nubit.md` - Nubit search structure
- `package-format.md` - Distribution format
- `decisions/ADRs.md` - Architecture decisions
- `../discussion/discussion.json` - Full decision record
- `PLAN.md` - Implementation plan

## Implementation Context

**Control group**: oracule-core (Python/networkx knowledge-graph engine) is frozen as the binary/float control group for A/B benchmarking.

**Known bug**: oracule-core `learning.py:211` uses ACTIVE without importing it. Must fix first.

## See Also

- oracule-core prototype (control group)
- TArchI design documents in sibling `/tarchi/` directory
