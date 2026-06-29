# Cross-Model Adversarial Review (Codex)

A read-only review by a *different model* — Codex (GPT-5.x) via the `codex` CLI.
Same-model review shares the author's blind spots: a Claude reviewer reasoning
over a Claude-authored spec/plan/diff can reach the same wrong conclusion the
author did. A different model breaks that correlation, so this is the strongest
independent pass available. It is **read-only** (Codex runs in a read-only
sandbox and only reports; you write every fix), so it fits the doctrine like any
delegated review.

**Self-contained — no plugin.** This uses this skill's `scripts/codex-review`,
which shells out to `codex exec` directly. The only requirement is the `codex`
CLI installed and on `PATH` (plus `git` for code diffs). It works from any agent
that can run a shell — Claude Code, Cursor, others.

**Optional and degradable.** If `codex` isn't on `PATH`, the script exits `3`
with a skip notice — treat that as "no cross-model pass available" and rely on
the in-model lenses. Never block on it. (First-time setup, if needed:
`npm install -g @openai/codex` then `codex login`.)

## Invoke it

```bash
# Design / spec review (brainstorming):
skills/requesting-code-review/scripts/codex-review --kind spec --file docs/superpowers/specs/<file>.md

# Plan review (writing-plans):
skills/requesting-code-review/scripts/codex-review --kind plan --file docs/superpowers/plans/<file>.md

# Code review (executing-plans per-task gate — small diff, runs quickly):
skills/requesting-code-review/scripts/codex-review --kind code --base <BASE recorded before the task>
```

- **Scope code review per task, not per branch.** Pass the per-task BASE (the
  commit recorded before that task) so the diff stays small — a whole-branch base
  like `git merge-base main HEAD` chokes Codex on a large changeset, and doing it
  per task spreads the cost across execution instead of one long terminal loop.
- `--kind` bakes in the right attack framing (a spec is attacked for unfalsifiable
  criteria and missing failure modes; a plan for coverage gaps and infeasible
  tasks; code for auth / data-loss / races / rollback). Add `--focus "<text>"`
  only for extra project-specific weighting, and `--model <m>` to pin a model.
- The script is **synchronous** — it blocks until Codex finishes (a real review
  takes a minute or two, ~15-20k tokens for a small artifact). For a large
  artifact, launch it with `run_in_background: true` and read its output when it
  completes; there are no jobs to poll.
- **Output** (stdout): JSON matching `scripts/codex-review.schema.json` —
  `verdict` (`approve` | `needs-attention`), `summary`, `findings[]` (`severity`
  / `title` / `body` / `file` / `line_start` / `line_end` / `confidence` /
  `recommendation`), `next_steps`. For a document, findings cite its line
  numbers; for code, the diff's file and lines.

## Consolidate its findings

Fold Codex's findings into your review consolidation, tagged **`[codex]`**. A
finding Codex *and* an in-model lens both raise is the highest-signal class —
independent agreement across models. Rank it with the rest (Blocker / Important /
Nice; a `needs-attention` finding with high `confidence` is usually a Blocker),
apply the fixes yourself per superpowers:receiving-code-review, and re-run the
pass after fixes if it raised blockers. Cross-model does not mean infallible —
push back with reasoning on a finding that is wrong, exactly as you would with an
in-model reviewer.
