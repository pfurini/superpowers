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

## File Structure

Before defining tasks, map out which files will be created or modified and what each one is responsible for. This is where decomposition decisions get locked in.

- Design units with clear boundaries and well-defined interfaces. Each file should have one clear responsibility.
- You reason best about code you can hold in context at once, and your edits are more reliable when files are focused. Prefer smaller, focused files over large ones that do too much.
- Files that change together should live together. Split by responsibility, not by technical layer.
- In existing codebases, follow established patterns. If the codebase uses large files, don't unilaterally restructure - but if a file you're modifying has grown unwieldy, including a split in the plan is reasonable.

This structure informs the task decomposition. Each task should produce self-contained changes that make sense independently.

## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.

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

## Self-Review

After writing the complete plan, look at the spec with fresh eyes and check the plan against it. This is a checklist you run yourself — not a subagent dispatch.

**1. Spec coverage.** Check the plan against the spec's goals, acceptance criteria, non-goals, and open questions:
- **Goals / requirements** — can you point to a task that implements each? List any gaps.
- **Acceptance criteria → validation map** — give each criterion a named verification at the *cheapest* layer that already confirms it: an existing test or E2E, the type-checker, or a documented manual check all count. Coverage is collective — one assertion or one existing run can satisfy several criteria. Add a *new* test only for a criterion nothing already proves (TDD governs that new behavior inside its task). Never assume one criterion = one new test. When the criteria are few this is a mental check; when there are enough to lose track, write the map into the plan as a small table (*criterion / how verified / new or existing*).
- **Non-goals** — confirm no task implements one; that's scope creep.
- **Open questions** — confirm none that are load-bearing remain unresolved. If one blocks a task, surface it rather than planning around it.

**2. Placeholder scan:** Search your plan for red flags — any of the patterns from the "No Placeholders" section above. Fix them.

**3. Type consistency:** Do the types, method signatures, and property names you used in later tasks match what you defined in earlier tasks? A function called `clearLayers()` in Task 3 but `clearFullLayers()` in Task 7 is a bug.

If you find issues, fix them inline. If you find a spec requirement with no task, add the task. This self-review is your first pass — the independent coverage review below is the gate.

## Independent Coverage Review

Your self-review is the author grading their own work: it catches the obvious gaps, not your blind spots, and it leans on the spec backbone. Before handoff, dispatch a fresh **read-only** reviewer to verify the plan covers the *entire* design — the spec backbone **and** the design sections (architecture, data model, data flow & error handling, file table, testing, rollback) — and is buildable. Writing stays with you: the reviewer only reads and reports; you fix.

For any multi-task plan this review is mandatory. A trivial single-task plan whose self-review already showed full coverage may skip it — say so explicitly.

Dispatch a `general-purpose` subagent, filling [plan-document-reviewer-prompt.md](plan-document-reviewer-prompt.md). Give it:
- the plan file path and the design/spec file path (and the `GLOSSARY.md` / ADR paths if the plan honors any),
- a model scaled to the plan's size and risk — a short plan does not need the most capable model; a broad multi-component plan does. Specify it explicitly, or it silently inherits your session's most expensive model.

Act on the findings yourself, per superpowers:receiving-code-review (verify before changing; push back with reasoning if a finding is wrong). When the reviewer names a design element with no covering task, add the task. Re-dispatch a fresh reviewer after fixes until it returns **Approved**, then hand off.

## Execution Handoff

After saving the plan, hand off to execution:

**"Plan complete and saved to `docs/superpowers/plans/<filename>.md`.**

- **REQUIRED SUB-SKILL:** Use superpowers:executing-plans

The plan executes on the main agent task-by-task: each task is implemented, self-reviewed, committed, and gated behind a fresh read-only review pass before the next begins."
