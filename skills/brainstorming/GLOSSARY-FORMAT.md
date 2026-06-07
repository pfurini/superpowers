# GLOSSARY.md Format

`GLOSSARY.md` is the project's canonical terminology — one agreed name per concept,
with the alternatives you've decided *not* to use. It is a glossary and nothing else:
no implementation detail, no spec content, no scratch notes. Code, docs, specs, and
ADRs all spell concepts the way the glossary spells them.

## Structure

```md
# {Project} Glossary

{One or two sentences on what this project is, so a term's meaning has a frame.}

## Terms

**Order**
A customer's request to buy, from submission until it ships or is cancelled.
_Avoid_: purchase, transaction

**Invoice**
A request for payment sent to a customer after delivery.
_Avoid_: bill, payment request

**Customer**
A person or organization that places orders.
_Avoid_: client, buyer, account
```

Group terms under `## {Cluster}` subheadings when natural clusters emerge; a flat
`## Terms` list is fine when they don't.

## Rules

- **Be opinionated.** When several words exist for one concept, pick the best and list
  the rest under `_Avoid_`. The value is the *choice*, not the catalogue.
- **Define what it IS, not what it does.** One or two sentences. If you're writing
  behavior, you're writing a spec, not a glossary entry.
- **Only project-specific terms.** General programming vocabulary (timeout, retry,
  cache) doesn't belong even if the project leans on it. Ask: is this a concept unique
  to this domain, or a general one? Only the former.
- **No implementation.** Table names, function names, file paths — none of it. The
  glossary outlives any particular implementation.

## Single vs. multi-area repos

**Single area (most projects):** one root `GLOSSARY.md`.

**Multiple areas with genuinely different languages** (a term means different things in
different subsystems): a root `GLOSSARY-MAP.md` indexes them and records how the areas
relate; each area keeps its own leaf `GLOSSARY.md`.

```md
# Glossary Map

## Areas

- [Ordering](./src/ordering/GLOSSARY.md) — receives and tracks customer orders
- [Billing](./src/billing/GLOSSARY.md) — generates invoices and processes payments

## Relationships

- **Ordering → Billing**: Ordering emits `OrderShipped`; Billing turns it into an Invoice.
- **Ordering ↔ Billing**: shared identifiers — `CustomerId`, `Money`.
```

Create the map **lazily** — only when a second area genuinely needs its own language.
Most projects never do. If only a root `GLOSSARY.md` exists, it's a single-area repo;
if neither exists, create the root `GLOSSARY.md` when the first term is worth pinning
down.

## Already have a glossary somewhere else?

Many repos already carry an informal glossary inside a README, an `AGENTS.md` /
`CLAUDE.md`, or a wiki. Don't create a second, competing one. Instead, offer to
**consolidate**: move the terms into `GLOSSARY.md` and leave a one-line cross-reference
where they used to live (e.g. "Canonical glossary: `GLOSSARY.md`"). One home, linked
from everywhere — never the same term defined in two places that can drift apart.
