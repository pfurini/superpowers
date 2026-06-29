# Local cross-model review loop

A bounded, severity-gated, **local** review loop — no PR, no CI, no cloud review
tokens. Run it after a wave/task and before opening a PR. It runs two
complementary lenses, merges them deterministically, then *adjudicates* each
finding (you verify, you don't perform), fixes the survivors, and re-reviews
until the high-severity set is empty — or a cycle cap stops it.

**Why a loop and not a single pass:** an LLM asked to "find something" will find
something. So the loop never terminates on *"the reviewer went quiet"* — it
terminates on **zero open *adjudicated* P0/P1**, capped, with stall-escalation.
Adjudication is the bias sink: a fabricated or speculative finding dies at triage
with a one-line refutation and never counts.

## The two lenses (complementary, run both)

- **adversarial** — `codex-review --kind code --base <BASE>`: holistic "break
  confidence" review; catches design/boundary weaknesses.
- **rubric** — `codex-review --lens rubric --base <BASE>`: the exact codex-native
  patch-bug rubric; surgical, P0–P3, "prefer no findings".

For a real cross-model consensus, run a lens on a second provider too
(`--model <other>`), and label it when merging. Lenses are read-only; run them in
parallel.

## The loop

```
BASE=<merge-base or the wave's recorded base SHA>
CAP=3 ; round=0 ; prev_high=∞
```

Each round:

1. **Review (parallel).** Run the lenses, capture each to its own file:
   ```bash
   codex-review --kind code --base "$BASE"                 > /tmp/rv-adv.json   &
   codex-review --lens rubric --base "$BASE"               > /tmp/rv-rub.json   &
   wait
   ```
   (Add `--model <other-provider>` runs here for consensus.) A lens exiting 3 =
   codex unavailable → degrade to the lenses you have; never block.

2. **Merge (deterministic).** 
   ```bash
   review-merge adversarial:/tmp/rv-adv.json rubric:/tmp/rv-rub.json > /tmp/rv-merged.json
   ```
   Read the `REVIEW_MERGE high=N …` line from stderr. `high` = candidate P0+P1.
   `consensus` findings (raised by ≥2 lenses) are the highest-signal — adjudicate
   them first.

3. **Adjudicate every finding — verify, don't perform.** For each, assign exactly
   one status against the actual code (not the reviewer's confidence):
   - **confirmed** — reproduced / provably affected; keep.
   - **refuted** — one-line evidence it's wrong (cite the code). Drop it.
   - **already-fixed** — present code already handles it. Drop.
   - **wont-fix** — intentional or out-of-scope; record why. Drop.
   Only **confirmed P0/P1** are "open". (P2/P3 are logged, never loop-blocking.)

4. **Fix the open P0/P1**, one at a time, test each (a behavior change includes
   its test). Re-run only the affected tests/gate.

5. **Converge or iterate.**
   - `open_high == 0` → **done.** (Zero findings at round 0 is a valid, expected
     outcome — ship it.)
   - `round+1 >= CAP` → **stop**, report the remaining open P0/P1 to the human.
   - `open_high >= prev_high` (not dropping) → **stall → escalate to the human**:
     either a genuinely broken change to re-scope, or reviewer noise. Don't spin.
   - else `round++`, `prev_high=open_high`, **re-review** (the next round verifies
     *resolution* of the prior P0/P1; do not open-endedly hunt new scope).

## Rules

- **Terminate on adjudicated severity, never on raw finding count.** `high` from
  the merge is *candidates*; the loop gate is *confirmed* P0/P1 after step 3.
- **"No findings" is permitted and made easy** — both lens prompts say so. Don't
  manufacture work to justify another round.
- **Stay local.** This loop pushes nothing. Open the PR *after* it converges; let
  the cloud `@codex review` be a single confirmation pass, not the loop.
- **Push back in writing.** A refuted finding gets a one-line reason in the
  adjudication ledger — that ledger is the audit trail that the loop converged on
  real issues, not reviewer noise.
