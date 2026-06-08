# Grilling Toward Alignment

The clarifying-questions phase is an interview, not a survey. The goal is not a filled
form — it's a shared mental model precise enough that the design has no load-bearing
"probably" left in it. This file is the how.

## The three disciplines

**1. Map the decision tree first — silently.** Before asking anything, list every
decision the design depends on, in dependency order. Don't show this list; it's
scaffolding. The point is to ask the *root* questions first — the ones whose answers
prune or reshape everything downstream — instead of asking in the order things occur to
you. Asking a leaf question before its parent forces the user to reason about a
combinatoric space instead of a single fork.

**2. Recommend an answer to every question.** Each question ships with your pick and a
one-line reason. The user's job is to confirm, override, or redirect — not to generate
answers from scratch. You have more context than you think; a recommendation-free
question pushes work back onto the user that you could have done.

```text
**Q:** Between per-row soft-delete and a separate archive table, which fits — given
you need the rows queryable for 30 days, then gone?

**Recommendation:** soft-delete with a `deleted_at` plus a nightly purge, because a
30-day window is short enough that a second table isn't worth the join.

Alternatives: archive table | hard-delete + audit log.
```

**3. Check the code before you ask.** If the repo already answers a question — a
convention, an existing pattern, a prior decision — find it and cite it instead of
asking. "This needs a new column on `orders`; or is a separate table the pattern here?"
beats "does this touch the database?". Prefer the project's semantic / code-graph tools
if it has them; otherwise an explore subagent, or grep/read. Reading real code in front
of the user is the moment they trust you're grounded in *their* system, not a generic
checklist.

## Walk the tree

Ask one question. Wait. Absorb the answer: if it kills branches, cross them off; if it
opens new ones, add them. Don't move to the next question until the current answer is
integrated. If an answer is ambiguous, ask **one** clarifying follow-up — not three.
Continue in dependency order to the natural end of the tree, not a round number of
questions.

## Sharpen the language as you go

Terminology drift is a design bug that ships. While you interview:

- **Challenge against the glossary.** If the user uses a term that conflicts with the
  project's `GLOSSARY.md`, call it out: "Your glossary defines 'cancellation' as X, but
  you seem to mean Y — which is it?"
- **Sharpen fuzzy or overloaded terms.** Propose a precise canonical name. "You're
  saying 'account' — do you mean the Customer or the login User? Those are different
  things." When a term resolves, lock its canonical definition in your notes right
  then — don't lose it. (The `GLOSSARY.md` file write itself is deferred until the
  worktree exists; see the brainstorming checklist — interview, then workspace, then writes.)
- **Stress-test relationships with concrete scenarios.** Invent specific cases that
  probe the edges: "A customer cancels one of three items after one has shipped — what
  exists now?" Forces a precision the abstract question doesn't.
- **Cross-reference code.** When the user states how something works, check whether the
  code agrees, and surface contradictions: "Your code cancels whole Orders, but you just
  described partial cancellation — which is right?"

## When to stop

Stop the interview when:

- Every remaining open decision is either deferred by explicit choice ("decide at
  execution time") or out of scope, **and**
- Nothing left would materially change the design, **or**
- The user says stop.

Stop **before** you reach implementation detail that's genuinely decided at coding time.
Grilling past the useful horizon is its own anti-pattern.

## Anti-patterns

- **Parallel questions.** "What's the schema? And the API? And the auth model?" — ask
  one; the answer to the first often reshapes the rest.
- **Yes/no railroading.** "Should we use X?" instead of "Between X and Y, which fits —
  given constraint Z?"
- **Recommendation-free questions.** Makes the user generate from scratch.
- **Asking what the code already answers.** Check first.
- **Grilling past the horizon.** If the next question is a coding-time detail, stop.

## Done when

- [ ] Every decision with cross-cutting impact has an answer.
- [ ] No "probably", "I think", or "we'll figure it out" remains on a load-bearing decision.
- [ ] The user has confirmed the *shape* of the design, not just each isolated answer.
- [ ] Terms that needed pinning down are in `GLOSSARY.md`; decisions worth recording are
      flagged for ADRs.
