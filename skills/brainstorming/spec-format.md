# Spec / Design Document Format

The document this skill produces is part **spec** (what we're building and why — the
part a reviewer signs off on) and part **design** (how it's shaped — the part that
guides implementation). The spec backbone below is the mandatory core; the design
sections layer on top, each scaled to its complexity — a sentence when it's obvious, a
few paragraphs when it's nuanced. A truly simple change might be half a page total.
Don't pad sections to look thorough.

## The spec backbone

**One-line summary.** Title, then a single sentence: `This proposes <X> so that <Y>.`
X is concrete (a behavior, an artifact); Y is the outcome. If you can't write it in one
try, the idea isn't clear enough yet — go back to questions.

**Goals.** 3-7 bullets, each a concrete observable outcome — something you could write a
test for, even if you won't.

**Non-Goals.** 3-5 bullets. *This is the more important list.* Each non-goal is
something a reasonable reader might assume is in scope but isn't. Goals expand on their
own in conversation; non-goals only get pinned down when you write them, and unwritten
ones get built anyway (doubling the work) or cut at the end (disappointing someone).

**Constraints.** Every external requirement the implementation must respect, grouped:

- **Compatibility** — APIs, schemas, protocols that must not break
- **Performance** — latency budgets, throughput floors, payload sizes
- **Security / Compliance** — auth, data residency, audit, PII handling
- **Operational** — supported runtimes, environments, infra dependencies

Each one line, concrete, falsifiable. "Must be performant" is not a constraint;
"p95 < 200ms at 1k RPS" is. State `None` explicitly for a group rather than omitting it.
Can't answer one? Mark it `OPEN` and move it to Open Questions — don't guess.

**Acceptance Criteria.** Numbered, pass/fail, no subjective words. Each is
`Given <state>, when <action>, then <expected>` or `<observable> is <measurement>`. At
least one per goal. "Orders older than 30 days return 410 for all roles" — yes. "The
feature works correctly" / "edge cases handled" — no.

**Phrase the invariant, not a test shape.** "The link's href is derived from the course
slug" (an invariant — one assertion on the slug proves it) beats "asserted for two
different courses" (a test shape — it implies a parametrized fixture nobody needs). A
criterion says what is *true* when done; it never prescribes how many tests prove it.
Mapping each criterion to its cheapest verification is the plan's job, not the spec's.

**Open Questions.** Each names the question, *who likely knows*, and *the impact of
getting it wrong*. If the spec has none, you're not looking hard enough — write down the
assumption you're least sure of as a question; that's your weakest link.

## Design sections (add what the work needs)

Layer these on when they carry weight — skip the ones that don't apply:

- **Verified current state** — what exists today, cited by `path:line`, with a date if
  it could drift. Verify; don't describe from memory.
- **Architecture & components** — the units, each with one clear purpose, a defined
  interface, and named dependencies. Prefer smaller, well-bounded units.
- **Data model / API shapes** — actual SQL, actual interfaces, actual request/response
  shapes, close enough that the implementer makes no design decisions.
- **Data flow & error handling** — the happy path, and what happens when each step fails.
- **File reference table** — paths from repo root, line numbers for specific logic.
- **Testing approach** — what to cover at unit / integration / e2e.
- **Rollback** — for anything touching data, infra, or shared state: how to undo it.
  Even "revert the PR" is worth stating.

## Where decisions go (don't write the same thing twice)

This document, the glossary, and the ADRs divide the labor. Each decision lands in one
of them, not all three:

| Artifact | Holds | Lives in |
|---|---|---|
| **This spec / design** | what & why, scope, non-goals, acceptance criteria, the shape | `docs/superpowers/specs/` (per task) |
| **ADR** | the load-bearing architectural *subset* that meets the 3 criteria — durable, code-citable | `docs/adr/` (project-level) |
| **GLOSSARY** | canonical terminology only, no behavior | `GLOSSARY.md` (project-level) |

The spec *references* an ADR ("the write model is event-sourced — see ADR-0004") rather
than re-arguing it. The decision's rationale lives in the ADR; the spec builds on it.

## Red flags

- Non-Goals empty or under two items — you haven't bounded the scope.
- A goal maps to no acceptance criterion — it's too abstract to ship.
- Acceptance criteria containing "should", "robust", "performant", "user-friendly" —
  none are testable; replace with measurements.
- The *requirements* run past a couple of pages — the rest is design detail; separate it.
- Written entirely in passive voice — you're hiding who owns the decision.
