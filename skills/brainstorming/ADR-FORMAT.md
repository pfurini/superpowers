# ADR Format

Architecture Decision Records live in `docs/adr/`, named `ADR-NNNN-slug.md` with
sequential numbering: `ADR-0001-slug.md`, `ADR-0002-slug.md`, … (the folder is a
default — follow the project's existing convention if it already keeps ADRs elsewhere.)

> **Why the `ADR-NNNN` id is load-bearing:** teams cite decisions inline from code
> (`// ADR-0006: soft-orphan, not cascade`), from commit messages, and from other
> ADRs. The id is the stable anchor those references resolve against, so the
> frontmatter `id` and `status` are **required**, not optional.

## Template

```md
---
id: ADR-NNNN
title: {the decision, one line}
status: accepted        # proposed | accepted | deprecated | superseded-by: ADR-MMMM
date: YYYY-MM-DD
---

# {Short title of the decision}

{1-3 sentences: the context, what was decided, and why.}
```

That's it. An ADR can be a single paragraph. The value is recording *that* a decision
was made and *why* — not filling out sections. Everything under the title is free-form
prose.

## Required frontmatter

- **`id`** — `ADR-NNNN`, matching the number in the filename. The anchor inline code
  comments cite, so it must be stable and unique.
- **`title`** — the decision in one line. Mirrors the `# ` heading; lets a reader (or a
  table of contents) see what was decided without opening the file.
- **`status`** — one of `proposed | accepted | deprecated | superseded-by: ADR-MMMM`.
  A reader who finds `// ADR-0011` in the code needs the doc to tell them whether it
  still holds; without a status, a stale reference is indistinguishable from a live one.
- **`date`** — the day the decision was made (`YYYY-MM-DD`). Decisions are read in
  sequence later; the date is how a reader orders them and judges what was known at the
  time. Use the real current date — don't guess it.

When a decision is replaced, set the old ADR's status to `superseded-by: ADR-MMMM`
rather than deleting the file — deleting leaves the inline code references dangling.

## Optional sections

Only when they add genuine value; most ADRs need none.

- **Context** — the forces in play, when the one-line summary under the title isn't enough.
- **Considered options** — when the rejected alternatives are worth remembering. Don't
  manufacture strawmen; if there was one obvious choice, leave it out.
- **Consequences** — when non-obvious downstream effects need calling out.

## Numbering

Scan `docs/adr/` for the highest existing `ADR-NNNN` and increment by one. The `id`
frontmatter must match the number in the filename.

## When to record an ADR

All three must be true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful.
2. **Surprising without context** — a future reader will look at the code and wonder
   "why on earth did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked
   one for specific reasons.

Easy to reverse? Skip it — you'll just reverse it. Not surprising? Nobody will wonder
why. No real alternative? Nothing to record beyond "we did the obvious thing."

### What qualifies

- **Architectural shape.** "Monorepo." "Event-sourced write model, projected read model."
- **Integration patterns between areas.** "Ordering and Billing talk via domain events,
  not synchronous HTTP."
- **Technology choices that carry lock-in.** Database, message bus, auth provider,
  deployment target — the ones that would take a quarter to swap, not every library.
- **Boundary and scope decisions.** "Customer data is owned by the Customer area;
  others reference it by id only." The explicit no's are as valuable as the yes's.
- **Deliberate deviations from the obvious path.** "Manual SQL instead of an ORM
  because X." Anything a reasonable reader would assume the opposite of — it stops the
  next engineer from "fixing" something that was deliberate.
- **Constraints not visible in the code.** "No AWS, for compliance." "Sub-200ms
  responses, per a partner API contract."
- **Rejected alternatives when the rejection is non-obvious.** Picked REST over GraphQL
  for subtle reasons? Record it, or someone re-proposes GraphQL in six months.

## Dedup before writing

ADRs are project-level and durable — never write one for a decision the project has
already recorded. Before creating any ADR, read the existing titles and bodies in
`docs/adr/` and match them against the decisions the session surfaced. Write new ADRs
only for the genuine gaps. If a surfaced decision *revises* an existing ADR, supersede
it (new ADR + `superseded-by` on the old) — don't silently duplicate.

## Example

```md
---
id: ADR-0007
title: Purchase and access are separate concerns
status: accepted
date: 2026-02-14
---

# Purchase and access are separate concerns

A completed Purchase records that money changed hands; Access records what a user may
read. They are separate tables linked by id, not one "entitlement" row. Chosen so a
refund can revoke access without erasing the financial record, and so a comp/grant can
hand out access with no purchase — the cost is two writes on the happy path.
```
