---
name: writing-plans
description: Use when you have a spec or requirements for a multi-step task, before touching code
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the writing-plans skill to create the implementation plan."

**Context:** If working in an isolated worktree, it should have been created via the `superpowers:using-git-worktrees` skill at execution time.

**Save plans to:** `docs/superpowers/plans/YYYY-MM-DD-<feature-name>.md`
- (User preferences for plan location override this default)

## Scope Check

If the spec covers multiple independent subsystems, it should have been broken into sub-project specs during brainstorming. If it wasn't, suggest breaking this into separate plans — one per subsystem. Each plan should produce working, testable software on its own.

## Honor Recorded Decisions and Canonical Terms

Before decomposing, read the project's architecture decision records (`docs/adr/` by convention) and its glossary (`GLOSSARY.md`, or wherever terms live). The plan must honor both:

- **Don't contradict an ADR.** Carry the spec's ADR references into the tasks that touch those decisions. Where the project cites ADRs inline in code (e.g. `// ADR-0006: …`), include the citation in the implementation step.
- **Use canonical terms.** Name types, tables, functions, and variables with the glossary's English identifiers — don't reintroduce the fuzzy synonyms the spec already sharpened away. Consistent names from spec → plan → code keep the three in sync.

## De-risk first: spike the unverified before you plan around it

If the spec carries a **Boundary assumptions** block — or the design rests on any unverified claim about an external system, tool, or platform — the plan's **first phase is spikes, not implementation.** A spike's *result changes the plan*; running it after the plan is committed is too late (the wrong choice is already baked in). For each open assumption, write a Phase-0 task whose deliverable is a yes/no answer:

- a 5-minute experiment — run the tool/flag/action once, write one throwaway test of the harness behavior, probe the API — or
- read the authoritative doc/source and cite it (`URL` / `path:line`).

Then decompose the implementation **on the spike's outcome**, not the assumption. (One-line spikes that would have pre-empted real post-merge failures: a release action that aborts on a tagless repo; a CI step whose output filename collided with a config file; a "no-op under test" import that actually throws.)

**Cold-start gets its own tasks.** For every required check, moving/channel tag, generated secret, or first build/release, add an explicit first-run/bootstrap task and document the order — the steady-state task never covers the run where the thing doesn't yet exist (e.g. a required status check can't be marked required until it has reported once).

**Tag any task whose correctness rests on an unverified boundary fact**, so the reviewer knows what to distrust. Review confirms internal consistency, not that an external system behaves as the task assumes.

This whole section is conditional on integration surface — a pure app-logic plan with a test oracle skips it.

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

**Mandatory reading for the executor.** If the plan leans on files the executor must read before Task 1 — a pattern to mirror, the types to import, a test to follow — list them once as a small table (*file / lines / why read*). It grounds the executor in real code instead of blind exploration, and it is where the per-task **Mirror** and **Imports** fields below come from. Get those `file:line` references from a read-only codebase-grounding pass ([../brainstorming/codebase-grounding.md](../brainstorming/codebase-grounding.md)), not from memory.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

Rough size signals: a plan under ~3 tasks is usually a TODO, not a plan; over ~30 tasks means the spec is too big (split it into sub-project plans) or the tasks are too small (fold setup/scaffolding into the deliverable that needs them).

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task on the main agent, with a read-only review pass gating each task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

## Global Constraints

[The spec's project-wide requirements — version floors, dependency limits,
naming and copy rules, platform requirements — one line each, with exact
values copied verbatim from the spec. Every task's requirements implicitly
include this section.]

---
```

## Task Structure

````markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Interfaces:**
- Consumes: [what this task uses from earlier tasks — exact signatures]
- Produces: [what later tasks rely on — exact function names, parameter
  and return types. A task's implementer sees only their own task; this
  block is how they learn the names and types neighboring tasks use.]

**Context (fill the lines that carry weight; omit the ones that don't):**
- **Mirror:** `path/to/exemplar.py:40-58` — the existing pattern this task copies. Paste the actual snippet and tag the source; reuse the codebase's convention instead of inventing one.
- **Imports & gotchas:** the exact import lines (with any quirks, e.g. `from zod/v4`) and the one pitfall that will bite — e.g. "use `results[0]`, not `.first()`".
- **Validate:** one command that confirms the whole task beyond its unit test — a type-check, lint, or focused run (e.g. `tsc --noEmit`). The cheapest layer that proves the task landed.

- [ ] **Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

- [ ] **Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

- [ ] **Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

- [ ] **Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

- [ ] **Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
````

## No Placeholders

Every step must contain the actual content an engineer needs. These are **plan failures** — never write them:
- "TBD", "TODO", "implement later", "fill in details"
- "Add appropriate error handling" / "add validation" / "handle edge cases"
- "Write tests for the above" (without actual test code)
- "Similar to Task N" (repeat the code — the engineer may be reading tasks out of order)
- Steps that describe what to do without showing how (code blocks required for code steps)
- References to types, functions, or methods not defined in any task

## Remember
- Exact file paths always
- Complete code in every step — if a step changes code, show the code
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits

## Risks and Rollback

For every task that touches production data, a shared schema, a public API contract, or an ordered multi-service deploy, write a one-line **rollback** — how to undo it — into the task. You authored the change, so you know *what* to undo; whoever runs the rollback later only knows *how*. If a task is irreversible (a destructive migration, a dropped column), write `Rollback: NOT POSSIBLE` and flag it for the architecture review — an irreversible step is safer reshaped behind a flag, dual-write, or backfill. A task whose risk you can't state a rollback for is a task whose risk you don't yet understand.

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage.** Check the plan against the spec's goals, acceptance criteria, non-goals, and open questions:
- **Goals / requirements** — can you point to a task that implements each? List any gaps.
- **Acceptance criteria → validation map** — give each criterion a named verification at the *cheapest* layer that already confirms it: an existing test or E2E, the type-checker, or a documented manual check all count. Coverage is collective — one assertion or one existing run can satisfy several criteria. Add a *new* test only for a criterion nothing already proves (TDD governs that new behavior inside its task). Never assume one criterion = one new test. When the criteria are few this is a mental check; when there are enough to lose track, write the map into the plan as a small table (*criterion / how verified / new or existing*).
  - **Classify the oracle, and flag the criteria nothing executable can prove.** Some surfaces — CI workflows, the release flow, infra-as-config — have *no* oracle: lint/type-check/review confirm syntax, never behavior, and their *first real run is their first test*. For each such criterion the validation is a **deliberate dry-run task** (exercise it on a throwaway basis before first real use), and the plan must label the surface **unproven until executed** instead of implying a test covers it. This is exactly where escaped defects cluster — don't let "reviewed" read as "proven" for a path no test touches.
- **Non-goals** — confirm no task implements one; that's scope creep.
- **Open questions** — confirm none that are load-bearing remain unresolved. If one blocks a task, surface it rather than planning around it.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

**4. One-pass confidence.** State a confidence (1-10) that an executor with no prior context could build this plan in one pass, plus a one-line rationale. Below ~7, the gap is yours to close *now* — usually a missing **Mirror**, **Imports**, or **gotcha**, or an unresolved open question — not the executor's to discover mid-task.

If you find issues, fix them inline. If you find a spec requirement with no task, add the task. This self-review is your first pass — the independent coverage review below is the gate.

## Plan Review

Your self-review is the author grading their own work: it catches the obvious gaps, not your blind spots. Before handoff, gate the plan behind a fresh **read-only** review. Independence is the value — the reviewers only read and report; *you* write every fix and consolidate the findings.

The review has up to three lenses, dispatched as parallel read-only sub-agents. They are orthogonal: a plan can cover the whole design (coverage) yet be unsound (architecture) or unusable (experience). Pick the lenses the plan's shape warrants:

- **Coverage** — [plan-reviewer-coverage.md](plan-reviewer-coverage.md). Does the plan cover the whole design, and add nothing it didn't? **Always dispatched.**
- **Architecture** — [plan-reviewer-architecture.md](plan-reviewer-architecture.md). Will it work — data flow, failure modes, edge cases, test matrix, rollback? Dispatch **only when the plan has real integration or state** (external calls, shared state, concurrency, migrations, ordered deploy).
- **Experience** — [plan-reviewer-experience.md](plan-reviewer-experience.md). Will anyone want to use it, and can a human operate it — across UI, CLI, API, and agent surfaces? Dispatch **only when the plan has a consumer-facing surface.**
- **Cross-model (optional)** — [../requesting-code-review/codex-adversarial-review.md](../requesting-code-review/codex-adversarial-review.md). A Codex (different-model) adversarial pass over the plan. The strongest *independent* lens — a different model has different blind spots — but degradable: skip it when Codex isn't installed. Use the plan focus text from that reference.

Decide which lenses apply before dispatching; don't run a lens that would score every dimension `n/a` (ceremony the agent learns to skip). A trivial single-task plan whose self-review already showed full coverage may run coverage alone, or skip the review — say which, explicitly.

Give each lens the plan file path and the design/spec path (plus `GLOSSARY.md` / ADR paths if the plan honors them), and a model scaled to the plan's size and risk — specify it explicitly, or it silently inherits your session's most expensive model.

**Consolidate the findings yourself** — this is writing, so it stays on the main agent:

1. Merge all findings into one list; **tag each by source** `[coverage]` / `[arch]` / `[exp]` / `[codex]`. A finding two lenses both raise becomes `[both]` and ranks higher — independent agreement is signal, and cross-model agreement (an in-model lens and `[codex]`) is the strongest signal of all.
2. Rank every finding **Blocker** (the plan can't execute, or executes into a wrong/unsafe result) / **Important** (executes but regrettable) / **Nice** (clarity only). A dimension a lens scored ≤4 is almost always a Blocker.
3. For each Blocker, decide with your human partner: **apply the fix**, or **skip it with a one-line written rationale carried into the plan**. A skipped Blocker with no recorded rationale is how a review silently becomes advisory — the rationale is the receipt.
4. Apply fixes yourself, per superpowers:receiving-code-review (verify before changing; push back with reasoning if a finding is wrong). When a lens names a design element with no covering task, add the task.
5. **Re-read the whole plan** after surgical edits — fixes drift: a renumbered task breaks a `Blocked by: Task 4` reference, a deleted task strands an interface. Then re-dispatch the applicable lenses until they come back clean.

Calibration guard: if every lens scores everything 9-10, the review was lazy — re-dispatch with sharper instruction. If you find yourself skipping every Blocker, you have the wrong lenses or the wrong plan; stop and reconsider.

## Execution Handoff

After saving the plan, hand off to execution:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`.**

- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans

The plan executes on the main agent task-by-task: each task is implemented, self-reviewed, committed, and gated behind a fresh read-only review pass before the next begins."
