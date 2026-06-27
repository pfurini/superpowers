---
name: executing-plans
description: Use when you have a written implementation plan to execute on the main agent, task-by-task, with a read-only review pass gating each task
---

# Executing Plans

## Overview

Execute a written plan on the main agent: load it, review it critically, then for each task write the code, test it, self-review, simplify, commit, and gate it behind a fresh read-only review pass before moving on. Track progress durably so the work survives compaction. Run a broad whole-branch review at the end.

**Announce at start:** "I'm using the executing-plans skill to implement this plan."

**Read-only delegation:** Writing always happens here, on the main agent — you implement every task yourself. Read-only passes are the only work you delegate: dispatch a fresh sub-agent for a task review, the final whole-branch review, or a research pass (understanding unfamiliar code, root-causing a failure). A read-only sub-agent returns findings; you do the writing.

**Narration:** between tool calls, narrate at most one short line — the ledger and the tool results carry the record.

## The Process

### Step 1: Load and Review Plan

1. Read the plan file. Note its context and Global Constraints.
2. **Pre-Flight Plan Review.** Before starting Task 1, scan the plan once for conflicts:
   - tasks that contradict each other or the plan's Global Constraints
   - anything the plan explicitly mandates that the review rubric treats as a
     defect (a test that asserts nothing, verbatim duplication of a logic block)

   Present everything you find to your human partner as one batched question —
   each finding beside the plan text that mandates it, asking which governs —
   before execution begins, not one interrupt per discovery mid-plan. If the
   scan is clean, proceed without comment. The review loop remains the net for
   conflicts that only emerge from implementation.
3. Review critically — identify any other questions or concerns. If concerns: raise them with your human partner before starting. If the plan rests on **undischarged boundary assumptions** (a claim about an external system/tool with no spike or citation behind it), treat that as a blocker like any other — spike or cite it before building on it; don't inherit an unverified assumption as fact.
4. Create todos for the plan items, start the progress ledger (see **Durable Progress**), and proceed.

### Step 2: Execute Tasks

For each task:

1. Mark the task `in_progress`. Record the BASE commit (`git rev-parse HEAD`) before you touch code — you need it for the review package, and it is never `HEAD~1` for a multi-commit task.
2. **Implement** exactly what the task specifies, following each bite-sized step. Follow TDD where the plan says to. While iterating, run the focused test for what you're changing; run the full suite once before committing, not after every edit.
3. **Self-review** with fresh eyes before you commit:
   - **Completeness:** did you implement everything in the task? Miss any requirement? Unhandled edge cases?
   - **Quality:** is this your best work? Are names clear and accurate (match what things do, not how they work)? Is the code clean and maintainable?
   - **Discipline:** did you avoid overbuilding (YAGNI)? Build only what was requested? Follow existing patterns?
   - **Testing:** do tests verify behavior, not mocks? Did you follow TDD if required? Is the test output pristine (no stray warnings or noise)?
   - **Falsification (integration tasks):** If the task rests on a claim about an external system/tool/platform, did you *verify* it (doc, source, or a quick spike) rather than trust the plan's wording? For any gate or conditional you wrote, trace the concrete scenario where it *must* fire and confirm it does — don't just re-read it. And if the change is a **no-oracle surface** (CI/workflow, release flow, infra-as-config), lint/type-green is **not** "done": its first real run is its first test, so a green run is part of completion — or a planned dry-run if it can't run before merge. Mark such surfaces *unproven until executed* in the ledger.
   - **Done means exercised, not just green.** Before marking a task done: run the **negative paths** (invalid input, missing field, unauthorized, empty result, max-size) and confirm each fails predictably, not with a crash; for any runnable surface, exercise it once **outside the test harness** (run the binary, hit the endpoint, trigger the real job) — a passing unit test is not a real run; and **cross-check the ask** — re-read the task text and map each thing it asked for to where you addressed it, or why you deferred it.

   Fix anything you find now, before committing.
4. **Simplify.** Make one focused pass over *only* the code this task touched, taking out complexity that crept in while making it work. This is the cheap moment — you hold the full context, and it lands before the review sees the diff:
   - Cut unnecessary nesting and indirection; inline a one-use abstraction; fold duplicated logic into one place (DRY, without inventing a premature abstraction for a single caller).
   - Prefer clarity over brevity — explicit code beats a dense one-liner, and no nested ternaries (an if/else chain or a switch reads better).
   - Delete dead code and comments that just restate what the code does.
   - **Don't over-simplify:** keep abstractions that earn their place, don't fold unrelated concerns together, and never trade readability for fewer lines. If a change makes the code harder to debug or extend, revert it.
   - This is behavior-preserving by definition: **re-run the task's covering tests after simplifying** and confirm they still pass before you commit. A simplification that changes behavior is a bug, not a cleanup — root-cause it with superpowers:systematic-debugging if a test goes red.
   - It extends the self-review's **Discipline** check, not a second competing list: YAGNI catches what you shouldn't have built; this catches complexity in what you did build.
5. **Commit** the task's work.
6. **Task review gate.** Dispatch a fresh read-only reviewer (see **The Read-Only Review Gate** below). If it reports spec ✅ and quality approved, continue. Otherwise fix the findings yourself and re-review until clean.
7. Mark the task complete in the todo list and the progress ledger.

### Step 3: Final Whole-Branch Review

After all tasks are complete and verified, run one broad review of the whole branch before finishing:

- Generate the review package for the full branch: run this skill's `scripts/review-package MERGE_BASE HEAD`, where MERGE_BASE is the commit the branch started from (e.g. `git merge-base main HEAD`). It prints the unique file path it wrote. Without bash, write the package yourself: `git log --oneline`, `git diff --stat`, and `git diff -U10` for the range, redirected to one uniquely named file.
- Dispatch a read-only final code reviewer using superpowers:requesting-code-review's [code-reviewer.md](../requesting-code-review/code-reviewer.md) template, on the most capable available model. Include the review-package path and the rolled-up Minor findings from the ledger so the reviewer can triage which must be fixed before merge.
- **Optionally** also run a Codex cross-model adversarial review over the branch — the `codex-review` script with `--kind code --base "$(git merge-base main HEAD)"`; see superpowers:requesting-code-review's [codex-adversarial-review.md](../requesting-code-review/codex-adversarial-review.md) for the exact path. It needs only the `codex` CLI on PATH (no plugin), and degrades to a skip if absent. A different model catches what the in-model reviewer's blind spots share with the author's; consolidate its `[codex]` findings with the final reviewer's (a finding both raise is the strongest signal). Degradable — skip it when Codex isn't installed.
- If the review returns findings, fix them all yourself in one pass (per superpowers:receiving-code-review), re-run the covering tests, and record the results. Do not spread the fixes across many context rebuilds.

### Step 4: Complete Development

After the final review is clean:
- Announce: "I'm using the finishing-a-development-branch skill to complete this work."
- **REQUIRED SUB-SKILL:** Use superpowers:finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice.

## The Read-Only Review Gate

The per-task review is a task-scoped gate; the broad review happens once, in Step 3. You wrote the code, so the reviewer must be a *fresh, independent* read-only sub-agent — your own self-review (Step 2.3) does not substitute for it, and a re-review after fixes is always a fresh dispatch. Independence is the entire value.

**Dispatch the task reviewer** with this skill's [task-reviewer-prompt.md](task-reviewer-prompt.md) template, giving it:
- the task's requirements (the task text from the plan) and the global constraints that bind it,
- the diff as a file: run this skill's `scripts/review-package BASE HEAD` and pass the reviewer the path it prints (or, without bash, write the package yourself with `git log --oneline`, `git diff --stat`, and `git diff -U10` for the range, redirected to one uniquely named file). The package never enters your own context; the reviewer reads the commit list, stat summary, and full diff in one Read call. Use the BASE you recorded before implementing — never `HEAD~1`, which silently truncates multi-commit tasks.

When you fill the reviewer template:

- Do not add open-ended directives like "check all uses" or "run race tests
  if useful" without a concrete, task-specific reason.
- Do not ask a reviewer to re-run tests you already ran on the same code —
  your self-review carries the test evidence.
- Do not pre-judge findings for the reviewer — never instruct a reviewer to
  ignore or not flag a specific issue. If you believe a finding would be a
  false positive, let the reviewer raise it and adjudicate it in the review
  loop. If the prompt you are writing contains "do not flag," "don't treat X
  as a defect," "at most Minor," or "the plan chose" — stop: you are
  pre-judging, usually to spare yourself a review loop.
- The global-constraints block you hand the reviewer is its attention
  lens. Copy the binding requirements verbatim from the plan's Global
  Constraints section or the spec: exact values, exact formats, and the
  stated relationships between components ("same layout as X", "matches
  Y"). The reviewer's template already carries the process rules (YAGNI,
  test hygiene, review method) — the constraints block is for what THIS
  project's spec demands.
- A dispatch prompt describes one task, not the session's history. Do not
  paste accumulated prior-task summaries ("state after Tasks 1-3") into a
  review dispatch — a fresh sub-agent needs the task's requirements, the
  diff, and the global constraints. Nothing else.
- A finding labeled plan-mandated — or any finding that conflicts with
  what the plan's text requires — is the human's decision, like any plan
  contradiction: present the finding and the plan text, ask which governs.
  Do not dismiss the finding because the plan mandates it, and do not apply
  a fix that contradicts the plan without asking.

**Acting on findings.** Fix Critical and Important findings yourself, on the main agent, following superpowers:receiving-code-review (verify before implementing; no performative agreement; push back with technical reasoning if a finding is wrong). Record Minor findings in the progress ledger as you go, and point the final whole-branch review at that list so it can triage which must be fixed before merge — a roll-up nobody reads is a silent discard. After fixing, re-run the tests covering your change and record the results, then re-generate the review package and re-dispatch a fresh reviewer. Repeat until the review is clean on both verdicts (spec compliance AND code quality).

**Handling reviewer ⚠️ items.** The task reviewer may report "⚠️ Cannot verify
from diff" items — requirements that live in unchanged code or span tasks.
These do not block the rest of the review, but you must resolve each one
yourself before marking the task complete: you hold the plan and cross-task
context the reviewer lacks. If you confirm an item is a real gap, treat it as
a failed spec review — fix it and re-review.

## Model Selection (review and research dispatches)

The sub-agents you dispatch are read-only review and research passes. Use the
least powerful model that can handle each, to conserve cost and increase speed.

- **Review tasks:** choose the model scaled to the diff's size, complexity, and
  risk. A small mechanical diff does not need the most capable model; a subtle
  concurrency change does. The final whole-branch review is the exception —
  dispatch it on the most capable available model.
- **Research tasks:** scale to the depth of investigation needed.
- **Always specify the model explicitly when dispatching.** An omitted model
  inherits your session's model — often the most capable and most expensive.
- **Turn count beats token price.** Wall-clock and context cost scale with how
  many turns a sub-agent takes; the cheapest models routinely take 2-3× the
  turns on multi-step work. Use a mid-tier model as the floor for reviewers.

## Durable Progress

Conversation memory does not survive compaction. Track progress in a ledger
file, not only in todos — a session that loses its place can re-do completed
tasks, the single most expensive failure mode.

- At skill start, check for a ledger:
  `cat "$(git rev-parse --show-toplevel)/.superpowers/exec/progress.md"`. Tasks
  listed there as complete are DONE — do not redo them; resume at the first
  task not marked complete.
- When a task's review comes back clean, append one line to the ledger in the
  same message as your other bookkeeping:
  `Task N: complete (commits <base7>..<head7>, review clean) — verified: <what proves it, e.g. 14/14 + negative paths + ran the CLI>`.
  Name what *proves* the task done, not just that it's done — a behavioral or
  no-oracle check that lives only in your context is lost at compaction; in the
  ledger it survives, and the final review can trust it.
- The ledger is your recovery map: the commits it names exist in git even when
  your context no longer remembers creating them. After compaction, trust the
  ledger and `git log` over your own recollection.
- `git clean -fdx` will destroy the ledger (it's git-ignored scratch); if that
  happens, recover from `git log`.

## When to Stop and Ask for Help

**STOP executing immediately when:**
- Hit a blocker (missing dependency, test fails, instruction unclear)
- Plan has critical gaps preventing starting
- You don't understand an instruction
- Verification fails repeatedly

**Ask for clarification rather than guessing.** When the blocker is "I don't
understand this code well enough," a read-only research sub-agent is a valid
move — but you still write the fix.

**When the blocker is a bug, a test failure, or unexpected behavior, invoke
superpowers:systematic-debugging** — find the root cause before you fix anything.
A patch that doesn't understand the cause usually just creates the next blocker.
Don't change the same line three different ways hoping one sticks; that's the
signal to stop and root-cause it.

## When to Revisit Earlier Steps

**Return to Review (Step 1) when:**
- Partner updates the plan based on your feedback
- Fundamental approach needs rethinking

**Don't force through blockers** - stop and ask.

## Red Flags

**Never:**
- Start implementation on main/master branch without explicit user consent
- Skip the task review gate, or accept a report missing either verdict (spec compliance AND task quality are both required)
- Let your own self-review replace the fresh read-only review (both are needed)
- Move to the next task while the review has open Critical/Important issues
- Skip review loops (reviewer found issues = you fix = review again)
- Dispatch a reviewer without a diff file — generate it first (`review-package BASE HEAD`) and name the printed path in the prompt
- Tell a reviewer what not to flag, or pre-rate a finding's severity in the dispatch prompt ("treat it as Minor at most")
- Re-dispatch — or redo — a task the progress ledger already marks complete; check the ledger (and `git log`) after any compaction or resume
- Delegate writing to a sub-agent; sub-agents are read-only review and research passes only
- Run a throwaway experiment (temp commit, `git checkout <sha>`, scratch branch) without isolating it — a detached HEAD or stray checkout can strand your real commits. Verify your branch/HEAD after any such detour, and prefer a separate worktree for spikes
- Call a no-oracle change (CI/workflow, release, infra) "done" on a green lint/type-check — those check syntax, not behavior; it isn't proven until a real run (or a planned dry-run) passes

## Remember
- Review the plan critically first, including the pre-flight conflict scan
- Follow plan steps exactly; don't skip verifications
- Self-review every task, then gate it behind a fresh read-only review
- Fix review findings yourself per receiving-code-review; re-review until clean
- Track progress in the ledger, not only in todos
- Reference skills when the plan says to
- Stop when blocked, don't guess

## Integration

**Required workflow skills:**
- **superpowers:using-git-worktrees** - Ensures isolated workspace (creates one or verifies existing)
- **superpowers:writing-plans** - Creates the plan this skill executes
- **superpowers:requesting-code-review** - Code reviewer template for the final whole-branch review (incl. the optional Codex cross-model adversarial pass)
- **superpowers:receiving-code-review** - How to act on review findings (verify, don't perform agreement)
- **superpowers:systematic-debugging** - Root-cause a bug / test-failure blocker before fixing it
- **superpowers:finishing-a-development-branch** - Complete development after all tasks
