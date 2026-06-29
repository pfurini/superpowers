# Changelog

This file documents the customizations carried by this fork (`pfurini/superpowers`,
branch `personal`) on top of upstream `obra/superpowers`. It does **not** track upstream
releases — see the upstream repository for those.

Fork base: `896224c` (merge-base with `upstream/main`).

The fork carries two themes. **(1)** Hardening the
`brainstorming → writing-plans → using-git-worktrees` pipeline with a spec / glossary /
ADR discipline and two worktree-timing fixes — skill-content only. **(2)** Reworking plan
execution into a single main-agent path (`executing-plans`) and removing agent-driven
development — this theme also touches `tests/`, `docs/`, and the plugin manifests, not
just `skills/`.

## Unreleased (fork-local)

### Added

- **Cross-model adversarial review via the `codex` CLI** (`a869862`, 2026-06-26).
  A self-contained read-only adversarial reviewer that shells out to `codex exec`
  (read-only sandbox, `--output-schema`) for a *different-model* pass over a spec,
  plan, or code diff: `skills/requesting-code-review/scripts/codex-review --kind
  spec|plan|code …` returns structured JSON findings, consolidated as `[codex]`.
  Depends only on the `codex` CLI on PATH (no third-party plugin), works from any
  shell-capable agent, and degrades to a skip when absent. Wired as an optional
  lens into the brainstorming spec review, the writing-plans plan review, and the
  executing-plans **per-task** review gate — per task, not over the whole branch,
  so each diff stays small (a whole-branch pass chokes on a large changeset) and
  the cost spreads across execution. (Landed plugin-based at `41c9e07`, then
  ported to the CLI-only mechanism.)

- **Read-only codebase-grounding pass** (`d28a55b`, 2026-06-25).
  `skills/brainstorming/codebase-grounding.md`: scope → entry points → trace &
  read → present-tense `file:line` artifact ("grep finds where, reading confirms
  what"). The mechanism brainstorming's "verified current state" and
  writing-plans' "mandatory reading" / per-task Mirror & Imports now draw from,
  instead of memory. Delegable as a read-only sub-agent.

- **Per-task executable context in plans** (`6ab9557`, 2026-06-25).
  The `writing-plans` task template gains optional `Mirror` (paste the exemplar +
  tag `path:line`), `Imports & gotchas`, and `Validate` (a task-level command),
  plus a plan-level "mandatory reading" table — context travels with each task so
  the executor doesn't re-derive it mid-flight.

- **Spec / glossary / ADR discipline in `brainstorming`** (`8fc21f7`, 2026-06-07).
  Four new reference docs under `skills/brainstorming/`:
  - `spec-format.md` — mandatory spec backbone (one-line summary, goals, non-goals,
    falsifiable constraints, pass/fail acceptance criteria, open questions) plus optional
    design sections, and a division-of-labor table so each decision lands in exactly one
    of spec / ADR / glossary.
  - `grilling.md` — reframes clarifying questions as an interview: map the decision tree
    silently and ask root questions first, carry a recommended answer on every question,
    and check the code before asking.
  - `ADR-FORMAT.md` — Architecture Decision Records in `docs/adr/`, the 3-criteria test
    (hard to reverse, surprising without context, real trade-off), and dedup rules.
  - `GLOSSARY-FORMAT.md` — canonical project terminology, single- vs multi-area repos,
    and consolidation of existing informal glossaries.

  `SKILL.md` checklist expands from 9 to 11 steps (adds "Create isolated workspace" and a
  conditional "Capture durable decisions" step) with a matching decision-tree loop in the
  process-flow graph.

- **Base-branch freshness check in `using-git-worktrees`** (`d98033d`, 2026-06-08).
  New "Step 0.5: Verify the Base Branch Is Current" — handles `fresh` vs `head` baseRef,
  fetches `origin`, warns on unpushed commits on the tracking branch, fast-forwards a
  behind local HEAD, and asks before forking from an ahead/diverged base. Adds matching
  quick-reference and "do NOT" entries.

- **ADR / glossary awareness in `writing-plans`** (`1137c03`, 2026-06-08).
  New "Honor Recorded Decisions and Canonical Terms" section: plans must not contradict
  ADRs and must use glossary identifiers. Spec-coverage review now maps each acceptance
  criterion to the cheapest existing verification (existing test, type-checker, or manual
  check), adding a new test only when nothing already proves the criterion. Plan-reviewer
  prompt updated to match.

### Changed

- **`executing-plans` adds a per-task simplify step** (2026-06-27).
  After each task's self-review and before commit/review, a focused
  behavior-preserving simplification pass over *only* the code that task touched
  (cut nesting/indirection, dedup, clarity-over-brevity, no nested ternaries;
  don't over-simplify), with the task's covering tests re-run afterward. The
  rubric is ported from the official `code-simplifier` agent but kept **inline on
  the main agent** — the only shape that runs identically across Claude Code /
  Codex / Cursor, and consistent with "main agent writes; only read-only work is
  delegated" (no read-write sub-agent).

- **Plan review is now multi-lens** (`a955187`, 2026-06-25).
  Generalizes the earlier single design-coverage gate into three orthogonal
  read-only lenses — **coverage** (always; now bidirectional, also flagging
  gold-plating), **architecture** (conditional on real integration/state), and
  **experience** (conditional on a consumer surface) — scored with 0/5/10 anchors,
  then consolidated on the main agent (`[both]` / cross-model agreement ranks
  higher, Blocker/Important/Nice, forced skip-rationale, post-edit re-read). Lens
  prompt templates live beside `writing-plans`.

- **`brainstorming` gains an independent spec review** (`41c9e07`, 2026-06-26).
  Un-orphans the in-model `spec-document-reviewer` and adds the optional Codex
  cross-model pass between the spec self-review and the user gate; brainstorming
  previously had no independent spec review at all.

- **`writing-plans` risk and confidence discipline** (`6ab9557`, 2026-06-25).
  A `Risks and Rollback` section (author-written one-line rollback per risky task;
  irreversible → `NOT POSSIBLE` + flag for the architecture review), a one-pass
  confidence score in self-review, and task-count size signals (under ~3 tasks =
  a TODO; over ~30 = split or fold).

- **`executing-plans` execution discipline** (`3fffdea`, 2026-06-26).
  Blockers route to `systematic-debugging` (root-cause before fixing); the
  done-check goes beyond green tests (negative paths, a real run outside the test
  harness, cross-check the ask against the task text); and the progress-ledger
  entry names what *proves* a task done, so the evidence survives compaction.

- **`executing-plans` is now the single main-agent execution path** (2026-06-24).
  Ported every durable mechanism out of the (now deleted) subagent-driven-development
  path into `executing-plans`: the pre-flight plan-conflict scan, per-task self-review, a
  mandatory per-task **read-only review gate** (fresh reviewer subagent → fix via
  `receiving-code-review` → re-review until clean), ⚠️-item handling, the reviewer-prompt
  construction discipline (copied verbatim), the durable progress ledger, model selection
  scoped to review/research, and the final whole-branch review. Doctrine: the main agent
  always writes; only read-only review/research is delegated to subagents. `writing-plans`
  collapses from the SDD-or-inline execution choice to this single path. The review
  artifacts moved under `skills/executing-plans/` (`task-reviewer-prompt.md` — adapted to
  drop the implementer/report framing — plus `scripts/review-package` and
  `scripts/workspace`), and the scratch dir was renamed `.superpowers/sdd` →
  `.superpowers/exec`.

- **`writing-plans` gains an independent design-coverage gate** (2026-06-25).
  Wired the previously-orphaned `plan-document-reviewer-prompt.md` into `writing-plans` as
  a mandatory read-only review after the author's self-review: a fresh subagent verifies
  the plan covers the whole design before handoff, and the main agent fixes any gaps (per
  `receiving-code-review`) and re-reviews until approved. Widened the reviewer's rubric
  from the spec backbone (goals / acceptance criteria / non-goals / open questions) to the
  design sections too (architecture & components, data model / API shapes, data flow &
  error handling, file table, testing approach, rollback) — each design element must map
  to a task or have its absence justified. Closes two gaps: plan-vs-design coverage was
  previously self-review only, and scoped to the backbone rather than the full design.

- **Aligned ADR/glossary format docs with the OpenSpec `openspec-design` skill**
  (2026-06-23). Ported the CLI-independent refinements from `Fission-AI/OpenSpec`:
  - `ADR-FORMAT.md` — added a required `title` frontmatter field (mirrors the heading,
    readable without opening the file) and `Context` to the optional-sections list.
  - `GLOSSARY-FORMAT.md` — added the "Append, don't clobber" rule and sharpened
    "Be opinionated" to the "kill synonyms" framing; **removed** the multi-area
    `GLOSSARY-MAP.md` machinery in favour of a single root `GLOSSARY.md` (a `Location`
    section); adopted OpenSpec's `**Term**:` entry style (dropped the `## Terms`
    wrapper); and added optional i18n `code`/`label (<lang>)` fields with a "stop at the
    default label, delegate the rest to the i18n catalogue" rule (the `code` field is
    synonym-killing on the code axis, the one sanctioned identifier in a definition).
  - Deliberately did **not** import OpenSpec's CLI-coupled features (ADR registry /
    `openspec adr index`, the `change:` field, the `proposed`→`accepted` lifecycle) —
    superpowers is zero-dependency and cannot maintain them.

- **Acceptance criteria anchored on the invariant, not a test shape** in
  `spec-format.md` (`adbb99e`, 2026-06-08). Criteria state what is true when done; mapping
  a criterion to its verification is the plan's job, not the spec's.

- **Worktree created before any file write** in `brainstorming` (`53c50b8`, 2026-06-08).
  The worktree is now created before the first file write, so the entire feature loop
  (spec, glossary, ADRs, plan, code) lands inside the worktree rather than the current
  checkout. The `GLOSSARY.md` write is deferred until the worktree exists.

### Removed

- **`subagent-driven-development` and `dispatching-parallel-agents` skills** (2026-06-24).
  Agent-driven (writing) development is gone — the main agent now does all writing. The
  durable read-only mechanisms were ported into `executing-plans` (see Changed); the
  no-longer-applicable pieces were dropped (implementer dispatch, `task-brief`,
  implementer-status handling, parallel-implementer warnings). Updated every live
  reference: `README.md`, `requesting-code-review`, the antigravity / codex / gemini tool
  refs, `docs/porting-to-a-new-harness.md`, `docs/testing.md`, and the codex / kimi plugin
  manifests. Removed the SDD test suite and the SDD-specific explicit-skill-request
  prompts; repointed the workspace unit test and the multi-turn skill-trigger test to
  `executing-plans`. (Dated historical plans/specs under `docs/` retain the old names as a
  record.)
