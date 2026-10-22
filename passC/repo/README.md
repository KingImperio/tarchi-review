# TArchI — True Archaion Intelligence

A trinary (-1, 0, +1) non-parametric intelligence that reasons by derivation, never uses LLMs, and ships its knowledge as a portable, self-verifying package.

## What this is

Archaion means "Intelligence From The Beginning". Reasoning here is derived from foundational axioms rather than interpolated from a corpus, so the system is auditable from the ground up and does not depend on a frozen training snapshot. The substrate is trinary: every signal is one of three values, -1, 0, or +1, and four orthogonal trit roles carry meaning. Truth states what is; Polarity states direction; Epistemic states confidence; Balance states the counterweight. Together they give each fact a value, a sign, and a certainty without needing a floating-point weight vector.

Knowledge is stored explicitly as a paged `.tarchi` package, not as trained weights. The package is portable, self-verifiable, and re-derivable: a consumer can load it, check its integrity, and reason over it without any private runtime state. Reasoning is search plus verification over explicit state. The Chess layer proposes candidate derivations by searching the loaded knowledge graph. The Rubik layer verifies them against axioms and cross-domain consistency, rejecting anything that does not hold. Generation is constructive and citation-bearing: every output traces to a source fact in the package, so answers can be checked rather than trusted. No LLM is used in any layer, ever.

The trit encoding is the core efficiency claim. A trit carries log2(3) = 1.585 bits, so 8 trits address 6561 states per byte, compared to 256 states for a byte of bits. This is what makes the size targets in the table below possible.

## Key numbers

| Metric | Value |
| --- | --- |
| log2(3) | 1.585 bits per trit |
| 8 trits | address 6561 states per byte |
| Rubik's cube | 66 bits vs 42 trits |
| Chess | 149 bits vs 94 trits |
| 1 Astra | 27B-param-equivalent, 1.35 GB target vs 13.5 GB INT4-equivalent |
| 3.7 Astra | 100B-param-equivalent, ~5 GB package vs 50 GB |
| Working set | must operate within a 2 GB resident working set |
| Density | target <= 1.5 bytes per fact |

## Architecture at a glance

The system is a layered pipeline over the trit substrate. The bottom layer is the trit encoding and the four orthogonal roles (Truth, Polarity, Epistemic, Balance). Above it sits the package format: a paged, self-verifying `.tarchi` container for knowledge. The reasoning core has two layers — Chess, which proposes candidate derivations by search over explicit state, and Rubik, which verifies them against axioms and cross-domain consistency. Generation is the top layer and is always constructive and citation-bearing.

## Repository layout

- `docs/` — specifications: `charter.md`, `tridak.md`, `nubit.md`, `package-format.md`, `decisions/ADRs.md`, `PLAN.md`
- `discussion/discussion.json` — machine-readable decision record
- `crates/` — Rust workspace (to come)
- `domains/` — domain axiom sets and knowledge graphs
- `corpus/` — source material and extraction pipelines
- `bench/` — benchmark harness and outputs
- `tests/` — integration and regression tests

## Status

Specifications and the implementation plan are complete. Implementation has not started. Milestones M0 through M7 are defined in `docs/PLAN.md`. The repository is a skeleton: directories are in place, the decision record and plan are committed, and concurrent agents are filling the remaining specification files.

## Related work

The earlier Python prototype `oracule-core` is retained separately as the binary/float control group for A/B benchmarking.

## License

License: pending.
