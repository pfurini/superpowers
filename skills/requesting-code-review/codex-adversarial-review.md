# Cross-Model Adversarial Review (Codex)

A read-only review by a *different model* — Codex (GPT-5.x) through the installed
`codex` plugin. Same-model review shares the author's blind spots: a Claude
reviewer reasoning over a Claude-authored spec/plan/diff can reach the same wrong
conclusion the author did. A different model breaks that correlation, so this is
the strongest independent pass available. It is **read-only** — Codex reports,
you write every fix — so it fits the doctrine like any other delegated review.

**Optional and degradable.** It needs the `codex` plugin installed and the Codex
CLI set up. If either is missing, skip it and rely on the in-model lenses —
never block on it. The discovery command below printing nothing means
unavailable.

## Invoke it

Discover the companion script (version-independent — it self-resolves its own
paths, so no plugin env vars are needed) and run the adversarial review:

```bash
SCRIPT=$(find "$HOME/.claude/plugins" -path '*openai-codex*' -name codex-companion.mjs 2>/dev/null | sort -V | tail -1)
[ -z "$SCRIPT" ] && { echo "Codex plugin not installed — skip the cross-model pass"; exit 0; }
node "$SCRIPT" adversarial-review [--base <ref>] "<FOCUS TEXT>"
```

- **Scope.** The review is git-diff-scoped and **counts untracked files**. For a
  spec or plan you just wrote (uncommitted), omit `--base` — the default
  working-tree scope picks the new file up. For committed work (the
  executing-plans code gate), pass the branch base: `--base "$(git merge-base main HEAD)"`.
- **`<FOCUS TEXT>`** steers Codex's (code-shaped) adversarial prompt onto the
  artifact — say what kind of artifact it is and what to attack. Templates below.
- **Output.** JSON on stdout: `verdict` (approve | needs-attention), `summary`,
  `findings[]` (`severity` / `file` / `line_start` / `line_end` / `confidence` /
  `recommendation`), `next_steps`.
- **Slow runs.** A Codex review can take minutes. For anything beyond a tiny
  artifact, launch it with `run_in_background: true` and collect the result with
  `node "$SCRIPT" status` / `node "$SCRIPT" result` (or `/codex:status`). Don't
  block a turn waiting on a large review.

## Focus text per artifact

- **Design / spec** (brainstorming): "The change under review is a DESIGN SPEC
  document (`docs/superpowers/specs/…`), not application code. Attack the design:
  vague or unfalsifiable acceptance criteria, unstated assumptions, failure modes
  it ignores, missing scope bounds / non-goals, hidden dependencies, and
  decisions that won't survive real-world load. Cite the spec's line ranges."
- **Plan** (writing-plans): "The change under review is an IMPLEMENTATION PLAN
  (`docs/superpowers/plans/…`), not application code. Attack the plan:
  design-coverage gaps, tasks that can't be executed as written, missing
  failure/edge handling, ordering and dependency hazards, missing rollback, and
  unverified boundary assumptions. Cite the plan's line ranges."
- **Code** (executing-plans gate): no doc framing needed — Codex's default attack
  surface (auth, data loss, races, rollback, idempotency, schema drift) already
  fits a code diff. Add a task-specific focus only if one applies.

## Consolidate its findings

Fold Codex's findings into your review consolidation, tagged **`[codex]`**. A
finding Codex *and* an in-model lens both raise is the highest-signal class —
independent agreement across models. Rank it with the rest (Blocker / Important /
Nice; a `needs-attention` finding with high `confidence` is usually a Blocker),
apply the fixes yourself per superpowers:receiving-code-review, and re-run the
pass after fixes if it raised blockers. Cross-model does not mean infallible —
push back with reasoning on a finding that is wrong, exactly as you would with an
in-model reviewer.
