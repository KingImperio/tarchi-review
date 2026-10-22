# TArchI — How to run the three-pass adversarial review (M0.6)

Target reviewer: **GPT-6.1 Sol** in the **ChatGPT desktop app → Codex view** (not Chat, not Work).
Reasoning effort: **xhigh** for Pass A and Pass C, **high** for Pass B.
Local Codex CLI (0.146.0, `gpt-5.4-mini` — retired 2026-08-31) is **not** used.

Everything is pre-split so you never have to hand-assemble a prompt.
**The three complete copy-paste prompts live in `PROMPTS.md`** — they name the workspace folder,
the reasoning effort, the exact repo files to visit and in what order, and the output path.
Each `REVIEW.md` also carries the repository reading order internally, so the reviewer still gets
it if you only paste the file.

| Pass | File | What it contains | Repo access |
|---|---|---|---|
| A (blind) | `passA/REVIEW.md` | raw intent + reviewer contract | **none — must stay blind** |
| B | `passB/REVIEW.md` | intent + locked decisions (D01–D20) + contract | read-only, `/home/fola/tarchi` |
| C | `passC/REVIEW.md` | intent + decisions + known open items + contract | read-only, `/home/fola/tarchi` |

Outputs are written to `out/passA.md`, `out/passB.md`, `out/passC.md`.

---

## Before you start (2 min)

1. Update the ChatGPT desktop app (Menu → Check for Updates) if **GPT-6.1 Sol** is missing from the
   model picker. Sol is rollout-gated: it appears under **Work/Codex**, not Chat.
2. `cd /home/fola/tarchi-review && mkdir -p out` (the folders already exist; `out/` may not).

---

## Pass A — blind re-derivation (do this first, alone, before reading anything else)

**Goal:** the reviewer derives its own architecture from the intent, *before* it sees ours.
Anchoring is the failure mode this pass exists to prevent.

1. Open Codex view → pick **GPT-6.1 Sol**, effort **xhigh**.
2. Open workspace folder: `/home/fola/tarchi-review/passA` — this folder contains **only**
   `REVIEW.md`. Do not open `/home/fola/tarchi` as the workspace for this pass.
3. Send exactly this:

   > Read `REVIEW.md` in this folder and execute it exactly. Do not read, search, or list anything
   > outside this folder. Do not access any other project on this machine. When the report is
   > complete, write the full report to `/home/fola/tarchi-review/out/passA.md` and tell me it is
   > done.

4. Walk away. Do not answer clarifying questions with design hints — reply only with "make an
   assumption and state it".
5. Check `out/passA.md` exists. **Save it. Do not read it yet** — read A only after C is in, so
   the passes stay independent. (Reading A now is fine for *you*, but do not feed it to Pass B/C.)

**If the reviewer asks to see the repo or other files: refuse.** Blind means blind. Note the
request in your triage notes — a reviewer that immediately wants the repo is itself a finding.

---

## Pass B — contradiction + hidden-assumption hunt

1. Same model/effort (**high**). Open workspace folder: `/home/fola/tarchi` (the repo — it needs to
   read `docs/` and `discussion/discussion.json`).
2. Bring the pass file in. Either:
   - **paste** the whole of `passB/REVIEW.md` into the composer (simplest), or
   - `cp /home/fola/tarchi-review/passB/REVIEW.md /home/fola/tarchi/.tarchi-review-passB.md`
     (gitignored; delete after the pass).
3. Send:

   > Execute the attached review brief. The repository at this workspace is read-only for you:
   > read files, do not modify, create, or delete anything. Write your full report to
   > `/home/fola/tarchi-review/out/passB.md` and tell me it is done.

4. Delete `.tarchi-review-passB.md` from the repo when it finishes.

---

## Pass C — gap hunt + verdict

1. Same model, effort **xhigh**. Workspace folder `/home/fola/tarchi` again.
2. Attach `passC/REVIEW.md` (paste, or `cp` to `/home/fola/tarchi/.tarchi-review-passC.md`).
3. Send the same instruction as Pass B, with `out/passC.md` as the target.
4. Delete the temp file.

**If the Codex view's file access is restricted to the workspace** and absolute paths outside it fail:
copy the file into the repo (option above) and also copy the repo somewhere the reviewer can read —
but for Pass A the sandbox folder `passA/` must stay the only thing in scope.

---

## After all three passes

Bring all three outputs back into the Kilo session — drop them in `out/` and say
*"triage the three review passes"*. Reconciliation rules (from the plan):

- Finding overturns a locked decision → **new superseding ADR**, creator decides. No silent edits.
- Finding opens a semantic gap → added to **M0.5** with an owner and an exit criterion.
- Finding invalidates a size/metric claim → revise `charter.md` / `package-format.md` /
  `discussion/discussion.json`, or mark the claim explicitly unachievable.
- Finding out of scope → recorded in Out of scope with a reason.
- **Nothing lands in the repo until the creator signs off.**

The reviewer's report format is fixed (`F<NN> / severity / what breaks / why our reasoning fails /
recommended change / D-number overturned / cost`) so triage is mechanical.

---

## Troubleshooting

| Symptom | Fix |
|---|---|
| GPT-6.1 Sol not in the picker | update the desktop app; Sol is plan-gated and shows under Work/Codex |
| Codex can't see `/home/fola/tarchi` from the `passA` workspace | expected — Pass A is blind; keep it that way |
| Codex refuses to write `out/passX.md` | let it print the report and copy it into `out/passX.md` yourself |
| Reviewer starts writing code | reply: "Reviewer role: no implementation, sketch only" and restart the pass if it edits the repo (`git status` in `/home/fola/tarchi` afterwards) |
| Reviewer asks clarifying questions | "State the assumption and proceed" |
