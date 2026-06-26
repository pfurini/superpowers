# Plan Reviewer — Architecture Lens

One of up to three read-only plan-review lenses. This lens asks **"will it
work?"** — data flow, failure modes, edge cases, the test matrix, and rollback
safety. It is orthogonal to the coverage lens (which asks "is the whole design
covered?"): a plan can cover every design element and still be architecturally
unsound. It does not edit anything — you act on its findings.

**Dispatch only when the plan has real integration or state** — external calls
(DB / queue / HTTP / third-party), shared mutable state, concurrency, migrations,
or an ordered multi-service deploy. Skip it for pure local logic with a test
oracle, or for prose/config-only plans (skill docs, static config).

**Dispatch after:** the complete plan is written and self-reviewed.

```
Subagent (general-purpose):
  description: "Review plan — architecture lens"
  model: [MODEL — REQUIRED: scale to the plan's size and risk; an omitted model
         silently inherits the session's most expensive one]
  prompt: |
    You are reviewing an implementation plan along the architecture axis: will
    it work? Read the plan and the design it came from once each. Your review is
    read-only: do not edit any file or mutate git state.

    **Plan to review:** [PLAN_FILE_PATH]
    **Design / spec for reference:** [SPEC_FILE_PATH]

    Score each dimension 0-10 against the anchors. Every score carries at least
    one finding that cites a plan task number or section — an uncited score is
    advice, not review. A dimension that genuinely does not apply to this plan is
    `n/a` (do not score it 10). A dimension scoring ≤4 is almost always a blocker.

    ## Dimensions

    1. **Data flow** — who owns each piece of data at each step; read/write
       ordering across services; eventual-consistency boundaries named.
       (10 = unambiguous from the plan alone. 5 = mostly clear, gaps in ordering
       or ownership. 0 = contradicts itself or the design.)

    2. **Failure modes** — for *each* external call (DB / queue / API): is the
       failure path named? timeouts? retry + backoff + idempotency? a
       circuit-breaker / fallback / fail-closed decision?
       (10 = every external interaction has a named failure path. 5 = some do.
       0 = happy-path only.)

    3. **Edge cases** — empty / max-size / unicode / boundary inputs; concurrency
       and optimistic locking; partial failure (1 of N writes); replays /
       idempotency.
       (10 = enumerated AND covered by acceptance criteria or tasks. 5 = named,
       not covered. 0 = unconsidered.)

    4. **Test matrix** — every task has a named test command; unit / integration /
       contract tests differentiated; negative tests present; the tests map onto
       the failure modes and edge cases line for line.
       (10 = tests track the failure/edge analysis. 5 = happy-path tests only.
       0 = no named tests.)

    5. **Rollback safety** — every high-risk task (prod data / shared schema /
       public API / ordered deploy) has a one-line rollback; a destructive or
       irreversible step is flagged NOT POSSIBLE and reshaped behind a flag /
       dual-write / backfill.
       (10 = every risky task has a rollback or a flagged reshape. 5 = some.
       0 = irreversible steps with no rollback.)

    **Cross-dimension consistency:** the test-matrix score should track the
    failure-modes score. If tests score far higher than failure modes, that is
    itself a finding — the tests cover non-architectural things, or the
    architecture has gaps the tests don't exercise.

    ## Output Format

    ## Architecture Review

    **Scores:** data-flow X/10 · failure-modes X/10 · edge-cases X/10 ·
    test-matrix X/10 · rollback X/10  (use `n/a` where a dimension doesn't apply)

    **Findings** (each cites a task#/section; mark the ones you consider blocking):
    - [dimension] Task N: [what's wrong] — [why it matters] — [blocking? y/n]

    **Verdict:** Sound | Needs work — [1-2 sentence technical assessment]
```

**Placeholders:**
- `[MODEL]` — REQUIRED: scale to the plan's size and risk
- `[PLAN_FILE_PATH]` — the plan under review
- `[SPEC_FILE_PATH]` — the design/spec the plan implements

**Reviewer returns:** per-dimension scores (or `n/a`), cited findings with a
blocking flag, and a verdict.
