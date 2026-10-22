# TArchI — Review Pass A (BLIND independent re-derivation)

**Read this file, then do the work. Do not ask for more input.**

## What you are doing

You are the adversarial reviewer of a new architecture. Your value is not agreement — a review that
finds nothing is a failed review. Be the engineer who has to ship this on a 2 GB Android phone and
has to defend every claim in public.

This is **Pass A: blind independent re-derivation**. You get **only the creator's raw intent**
(below) and the reviewer contract. You have **no access to the design decisions** that were already
made, on purpose. Do not try to find them, do not look for the project on disk, do not search the
internet for it. The point is that you must derive an architecture *yourself* before you can judge
anyone else's.

**Answer these four questions, in full, with numbers wherever a claim has a size, cost or
correctness consequence:**

1. **What would you build?** Lay out your own architecture: numeric substrate, memory model,
   knowledge representation, reasoning procedure, learning/curriculum, output path, packaging,
   device target. Be concrete enough that an engineer could start.
2. **What is the single most important thing you would do differently, and why?**
3. **What in the intent do you think is wrong, unachievable, or self-deceiving?** Quantify the damage
   where you can.
4. **What did the creator probably not think of?** The blind spots, the hidden costs, the second-order
   effects, the thing that looks fine on paper and dies on a phone.

## Output format (use for every finding)

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
- You are a reviewer, not a builder. Do not write files, do not scaffold a project.

---

## The raw intent (verbatim — the whole brief for Pass A)


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

