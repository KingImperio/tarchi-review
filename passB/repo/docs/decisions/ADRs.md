# Architecture Decision Record Log

## ADR-0001 — Standalone Project Structure

**Status:** Accepted

**Context:** TArchI is a new standalone project at `/home/fola/tarchi` (its own git repo), not a rewrite of the existing Python prototype oracule-core.

**Decision:** Establish TArchI as an independent project with separate codebase, documentation, and contribution guidelines from oracule-core.

**Consequences:** Clean separation between prototype and production; allows independent evolution; increases maintenance burden; enables focused benchmarking.

**Alternatives considered:** Merge into oracule-core, continue as separate prototype, maintain both under monorepo.

---

## ADR-0002 — Trinary Value Layering

**Status:** Accepted

**Context:** The trinary value is layered across three levels at once: truth at proposition level, node triune state at node level, edge polarity at edge level.

**Decision:** Implement trinary logic with three simultaneous layers: proposition-level truth, node-level triune state, and edge-level polarity, enabling comprehensive representation of knowledge and relationships.

**Consequences:** Richer knowledge representation; increased complexity; 27 distinct node states; enables direct modeling of multi-dimensional information.

**Alternatives considered:** Single-layer trinary system; hierarchical decomposition; hybrid binary/trinary approach.

---

## ADR-0003 — Four Orthogonal Trit Roles

**Status:** Accepted

**Context:** Four ORTHOGONAL trit roles in four layers (not one overloaded axis): Truth (-1 false/0 unknown/+1 true), Polarity on edges (-1 opposes/0 undetermined/+1 supports), Epistemic in the ledger (-1 rejected/0 candidate/+1 held), Balance as charge (-1 deficit/0 equilibrium/+1 surplus). Node = (truth, epistemic, balance) = 27 states mapped deterministically onto FOUNDATIONAL/ACTIVE/RESOLVED/EXHAUSTED/QUARANTINED.

**Decision:** Implement four distinct trinary roles across different layers: Truth (proposition evaluation), Polarity (edge relationships), Epistemic (ledger confidence), Balance (node charge). Combine epistemic and balance dimensions into 27 node states with deterministic lifecycle mapping.

**Consequences:** Comprehensive knowledge representation; 27 distinct node states; deterministic progression; enables nuanced reasoning about certainty and stability.

**Alternatives considered:** Single overloaded truth axis; weighted combination approach; probabilistic interpretation.

---

## ADR-0004 — Complete Intelligence Stack

**Status:** Accepted

**Context:** TArchI is a complete self-contained intelligence stack (perceive, reason, generate, act, remember, learn, self-correct) — not a router to other models.

**Decision:** Build end-to-end intelligence stack from ground up: perception (classical NLP), reasoning (derivation), generation (constructors), action (modulations), memory (persistent storage), learning (continuous), self-correction (Rubik layer verification).

**Consequences:** Full autonomy; no external dependencies; larger codebase; comprehensive functionality; higher development complexity.

**Alternatives considered:** Router to existing models (LLMs, symbolic AI); modular plugin architecture; minimal core with external components.

---

## ADR-0005 — Archaion Principle

**Status:** Accepted

**Context:** "Archaion" = the principle: Intelligence From The Beginning — derivation from foundational axioms rather than interpolation from a corpus.

**Decision:** Design system that derives all knowledge from first principles and axioms, building upward through logical derivation rather than learning patterns from data.

**Consequences:** Traceable reasoning; verifiable conclusions; no hallucination; requires initial axiom corpus; harder to adopt for new domains.

**Alternatives considered:** Data-driven interpolation; hybrid approach; purely empirical learning.

---

## ADR-0006 — Staged Development

**Status:** Accepted

**Context:** Scope is staged into milestones M0-M7 rather than fixed up front; the layers interconnect, so each stage is separately verifiable.

**Decision:** Break down development into 8 sequential milestones (M0-M7) with incremental feature delivery. Each milestone builds on previous layers while maintaining independent verifiability of completed functionality.

**Consequences:** Manageable development; regular validation points; flexible scope adjustment; interdependency management complexity.

**Alternatives considered:** Waterfall with full specification; agile sprints; feature-flagged incremental delivery.

---

## ADR-0007 — Rust Core + Python Bindings

**Status:** Accepted

**Context:** Rust core + Python research bindings. Python is never the runtime (per-object overhead would break the 2 GB RAM budget). Rust gives no_std, SIMD, Android NDK and WASM targets.

**Decision:** Implement core system in Rust for performance and portability, with Python bindings for research and prototyping. Runtime will be Rust-only to maintain memory constraints and performance requirements.

**Consequences:** High performance; multiple platform targets; Python accessibility; compilation complexity; learning curve for Rust.

**Alternatives considered:** Python-first implementation; Go runtime; C++ core with Python bindings.

---

## ADR-0008 — Balanced Numeral Family

**Status:** Accepted

**Context:** Numeral family named Balradix: Tridak = base-3 balanced digit {-1,0,+1}; Baldec = base-10 balanced digit {-5..+5}. Rationale: 0-9 came together and negatives are the opposite representation of the same magnitudes; balanced radix needs no sign bit, negates for free and has a single canonical zero.

**Decision:** Adopt balanced numeral system: Tridak (base-3 balanced digits {-1,0,+1}) for atomic operations, Baldec (base-10 balanced digits {-5..+5}) for higher-level encoding. This eliminates sign bits and enables efficient negation.

**Consequences:** Efficient arithmetic; canonical zero representation; no separate sign; intuitive to humans familiar with decimal; specialized implementation required.

**Alternatives considered:** Binary representation; standard base-10; pure trinary without balancing; custom encoding schemes.

---

## ADR-0009 — Binary Control Group

**Status:** Accepted

**Context:** oracule-core is frozen as the BINARY/FLOAT CONTROL GROUP for the trit-vs-bit A/B, with its bug at learning.py:211 (ACTIVE used but never imported) fixed first; later it will be upgraded to consume .tarchi packages.

**Decision:** Use fixed oracule-core as control group for A/B testing against TArchI implementation. Fix the known bug (learning.py:211) first, then extend it to support .tarchi packages for comparative analysis.

**Consequences:** Direct performance comparison; bug fixation opportunity; backward compatibility; enables validation of improvements.

**Alternatives considered:** New reference implementation; independent comparison framework; differential testing.

---

## ADR-0010 — Constructive Generation

**Status:** Accepted

**Context:** Generation is CONSTRUCTIVE via a Realizer trait adapter (answers assembled from the resolved substrate plus derivation path). NO LLMs anywhere in the system, ever. Consequence: language is precise, factual and cited, but not charming; open-ended chat and creative prose are out of scope for v0.1.

**Decision:** Implement generation using constructive reasoning: Realizer trait adapter assembles answers from substrate facts plus derivation paths. Use purely symbolic reasoning with no machine learning components.

**Consequences:** Precise, verifiable outputs; no hallucination; limited expressiveness; not suitable for creative writing; requires explicit knowledge base.

**Alternatives considered:** Neural generation; symbolic generation only; hybrid approaches; template-based generation.

---

## ADR-0011 — Curriculum-Based Learning

**Status:** Accepted

**Context:** The CURRICULUM replaces gradient descent: corpus -> deterministic template extraction (patterns + classical classifier; precedent TextRunner 2007: 11M triples from 500M sentences at ~91% precision with no neural net) -> canonicalization (entity resolution, alias collapse, Balradix IDs) -> verification against axioms (Rubik layer) -> induction of rules/schemas (ILP precedent Aleph/Progol) -> promotion with Baldec strengths + graduation on stability/redundancy/load-bearing -> per-session decay. Low-confidence or contradicting facts are QUARANTINED and never enter the active substrate. Consequence: there is no training step and no model in the LLM sense; the substrate IS the knowledge; learning is continuous, GPU-free, no catastrophic forgetting; a wrong fact is fixed by editing one node.

**Decision:** Replace model training with curriculum-driven knowledge acquisition pipeline. Process: corpus extraction → template mining → canonicalization → axiom verification → rule induction → promotion with quality metrics → ongoing decay and quarantine management.

**Consequences:** No neural network training; continuous learning; verifiable facts only; GPU-free operation; single-fact correction; complex pipeline with multiple stages.

**Alternatives considered:** Traditional ML training; active learning; reinforcement learning; rule extraction from trained models.

---

## ADR-0012 — Closed-Domain v0.1

**Status:** Accepted

**Context:** v0.1 scope: closed-domain RESOLVE + EXPLAIN + CITE over ported domains, deliberately comparable to the binary control on identical queries.

**Decision:** Initial version (v0.1) will focus on closed-domain question answering with RESOLVE (find answers), EXPLAIN (provide reasoning), and CITE (reference sources) capabilities over a curated set of domains. This enables direct comparison with binary control systems.

**Consequences:** Limited domain coverage; high precision requirement; comparable baseline; smaller initial implementation; clear evaluation metrics.

**Alternatives considered:** Open-domain questions; broader feature set; conversational mode; incremental capability expansion.

---

## ADR-0013 — Minimal Schema Mining

**Status:** Accepted

**Context:** v0.1 ships the Curriculum WITHOUT full ILP induction, but WITH minimal schema mining, because schema compression is what makes the 10x size target reachable.

**Decision:** For v0.1, implement only minimal schema mining instead of full ILP induction. Focus on schema compression to achieve the 10x size reduction target while maintaining core functionality.

**Consequences:** Smaller initial package; easier implementation; less comprehensive knowledge extraction; size target achievable; partial rule induction.

**Alternatives considered:** Full ILP induction; no schema mining at all; incremental schema development.

---

## ADR-0014 — Curated Knowledge Corpus

**Status:** Accepted

**Context:** v0.1 knowledge source: a curated corpus plus ground truth (the 4 prototype domains as prose plus real Wikipedia/technical pages on logic, arithmetic and mechanics), so extraction recall/precision and citation coverage are measured rather than guessed.

**Decision:** Initial knowledge base will be curated: prototype domain prose + Wikipedia + technical documentation on logic, arithmetic, and mechanics. This allows measurement of extraction quality and citation coverage.

**Consequences:** High-quality initial facts; measurable performance; domain-specific focus; curation overhead; limited initial coverage.

**Alternatives considered:** Raw web scraping; user-contributed content; semi-automated curation; academic databases.

---

## ADR-0015 — Portable Knowledge Packaging

**Status:** Accepted

**Context:** Knowledge IS packaged: the .tarchi package is TArchI's weights — axioms, rules, facts, index, quarantine, integrity; 4 KiB mmap-able pages; Balradix-coded page table for sub-byte addressing; Tridak as the atomic load/offload unit; two-tier integrity (per-page hash plus semantic re-verification of every fact against the axiom set on load); composable (base + domain + personal) and updatable by delta.

**Decision:** Knowledge packaging format (.tarchi) uses memory-mapped 4 KiB pages with Balradix-coded page tables for sub-byte addressing. Atomic operations use Tridak encoding. Two-tier integrity: page hashes + semantic verification against axioms. Packages are composable and delta-updatable.

**Consequences:** Efficient memory usage; sub-byte addressing; integrity guarantees; composable knowledge; update efficiency; complex format specification.

**Alternatives considered:** Binary format; text-based format; database format; compressed archive format.

---

## ADR-0016 — Astra Unit Definition

**Status:** Accepted

**Context:** "1 Astra" = the verifiable CAPABILITY of a 27B-param reference LLM, measured on BOTH a knowledge-recall suite (verifiable questions with distractors and injected contradictions, scored with citations) AND a derivation suite (multi-step questions scored on the derivation path). Storage is a separate hard constraint in bytes and INT4-param-equivalents. A package that only memorises answers scores zero on derivation, so a database cannot pass as an intelligence.

**Decision:** Define Astra unit as measurable intelligence capability: capability to perform as well as 27B-param LLM on both knowledge recall and derivation tasks. Requires performance on both dimensions, not just memorization.

**Consequences:** Strict intelligence definition; prevents database-only solutions; enables comparison with LLMs; complex evaluation framework; two-dimensional measurement.

**Alternatives considered:** Simple parameter count comparison; knowledge-only measurement; reasoning-only measurement; composite scoring.

---

## ADR-0017 — Size and Memory Targets

**Status:** Accepted

**Context:** Size and RAM targets (INT4 yardstick, 0.5 bytes/param): 1 Astra = 13.5 GB INT4-equivalent reference vs 1.35 GB TArchI target; 3.7 Astra (=100B-param-equivalent) = 50 GB vs ~5 GB package; must operate within a 2 GB resident working set with the package on flash and demand-paged; bytes-per-fact target <=1.5 (acceptable <=2.5); hot pages pinned by the reasoning frontier, not LRU. Tridak density: log2(3)=1.585 bits/trit, integer packing realizes up to 1.667 bits/trit (3->5, 5->8, 8->13, 12->20 bits); 8 trits address 6561 states per byte.

**Decision:** Target TArchI efficiency: 1.35 GB for 1 Astra (vs 13.5 GB reference), ~5 GB for 3.7 Astra (vs 50 GB reference). Must operate in 2 GB RAM with package on flash, demand-paged. Target: <=1.5 bytes/fact, hot pages pinned by reasoning frontier.

**Consequences:** Significant storage reduction; memory-efficient operation; complex addressing; performance optimizations required; aggressive compression.

**Alternatives considered:** More relaxed targets; tiered efficiency goals; optional compression; hardware-specific optimizations.

---

## ADR-0018 — Validation Metrics

**Status:** Accepted

**Context:** Validation is TArchI-native metrics plus a bit-vs-trit A/B on identical code paths: state density, knowledge density, latency, correctness, contradiction detection and self-correction, citation coverage and unsupported-claim rate, device footprint, and the Astra rating.

**Decision:** Validate using TArchI-native metrics: state density, knowledge density, latency, correctness, contradiction detection/self-correction, citation coverage, unsupported-claim rate, device footprint, and Astra rating. Also run A/B tests against binary implementations on identical code paths.

**Consequences:** Comprehensive validation; direct comparison capability; multiple quality dimensions; complex testing infrastructure; performance benchmarking.

**Alternatives considered:** Industry-standard benchmarks; academic evaluation; user testing; manual verification.

---

## ADR-0019 — Rubik and Chess Layers

**Status:** Accepted

**Context:** Rubik's Cube and Chess map onto two layers: the Rubik layer is the deterministic VERIFIER (closed finite group action, illegal states unrepresentable, zero hallucination by construction); the Chess layer is the adversarial PLANNER (branching, value-estimating). Nubits are the generalization: project ALL legal successor states of a state at once, each with polarity and confidence, instead of sampling one next token.

**Decision:** Implement two reasoning layers: Rubik layer for verification (deterministic, finite-state, zero hallucination), Chess layer for planning (adversarial, value estimation). Generalize to Nubits: project all legal successor states with polarity and confidence.

**Consequences:** Deterministic verification; zero hallucination; adversarial planning; comprehensive state exploration; complex state management; strategic reasoning capabilities.

**Alternatives considered:** Single verification layer; pure search-based planning; Monte Carlo tree search; heuristic evaluation.

---

## ADR-0020 — Classical Perception

**Status:** Accepted

**Context:** Perception without LLMs uses classical NLP (lexer -> morphology -> lexicon -> constraint parser -> semantic extraction) and reads ground truth from language servers for code (rust-analyzer, pyright) instead of guessing from raw text. Image/audio perception is out of scope for v0.1.

**Decision:** Implement perception using classical NLP pipeline: lexer → morphology → lexicon → constraint parser → semantic extraction. Use language servers (rust-analyzer, pyright) for code ground truth. Exclude vision/audio perception from v0.1.

**Consequences:** Precise extraction; verifiable results; no hallucination; dependency on language servers; limited input modalities.

**Alternatives considered:** Neural perception; rule-based NLP; hybrid approaches; multimodal input.

---

## Open questions

### (Q1) which reference LLM defines 1 Astra — candidates Gemma-2-27B (exact 27B, license-gated), Qwen2.5-32B, Mistral-Small-24B; recommendation Gemma-2-27B, dev-time only; blocks M5.

### (Q2) which realization registers ship in v0.1 — answer, explain-with-trace, contradiction notice, compare, summary; recommendation answer + explain + contradiction notice; blocks M4.

### (Q3) confirm browser/PWA as the first device target before investing in the NDK path; blocks M6.

