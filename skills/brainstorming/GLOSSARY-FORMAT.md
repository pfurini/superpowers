# GLOSSARY.md Format

`GLOSSARY.md` is the project's canonical terminology — one agreed name per concept,
with the alternatives you've decided *not* to use. It is a glossary and nothing else:
no implementation detail beyond the optional canonical `code` identifier (below), no spec
content, no scratch notes. Code, docs, specs, and ADRs all spell concepts the way the
glossary spells them.

## Structure

```md
# {Project} Glossary

{One or two sentences on what this project is, so a term's meaning has a frame.}

**Order**:
A customer's request to buy, from submission until it ships or is cancelled.
_Avoid_: purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: client, buyer, account
```

Group terms under `## {Cluster}` subheadings when natural clusters emerge; a flat list
is fine when they don't.

## Rules

- **Be opinionated.** When several words exist for one concept, pick the best and list
  the rest under `_Avoid_`. The glossary's job is to *kill synonyms*, not catalogue
  them — the value is the *choice*.
- **Define what it IS, not what it does.** One or two sentences. If you're writing
  behavior, you're writing a spec, not a glossary entry.
- **Only project-specific terms.** General programming vocabulary (timeout, retry,
  cache) doesn't belong even if the project leans on it. Ask: is this a concept unique
  to this domain, or a general one? Only the former.
- **No implementation detail in definitions.** Don't write table schemas, file paths, or
  internals into a definition — the glossary outlives any particular implementation. (The
  one optional `code` identifier below is the exception: it's synonym-killing on the code
  axis, not implementation detail.)
- **Append, don't clobber.** When a session introduces a genuinely new shared term, add
  one line; never rewrite existing entries to fit the change you're working on.

## Code identifier and default-language label (optional)

Two optional fields cover the common case where a project's user-facing language isn't
English, or a concept's code symbol differs from its name. The bold heading stays the
canonical concept name — what docs, specs, and ADRs use:

- **`code`** — the single canonical identifier the codebase uses (table, type, column,
  enum, function), when it differs from the heading. This is `_Avoid_` applied to the
  code axis: one canonical name, no drift. Omit it when the heading already is the code
  name.
- **`label (<lang>)`** — the user-facing string in the project's default language.

```md
**Withdrawal waiver**:
A buyer's per-purchase waiver of the 14-day right of withdrawal.
code: `recesso_waiver`
label (it): Recesso
_Avoid_: cancellation, refund opt-out
```

**Stop at `code` plus one default label.** The glossary fixes the *concept* and its
canonical code name; it is not a translation catalogue. Full localization — every string
in every supported language — belongs to the project's i18n layer, keyed by the `code`
identifier. Don't mirror all translations here: a second copy drifts from the catalogue
the moment one is wired in, and adding a locale would then mean editing every entry
instead of extending the catalogue.

## Location

One `GLOSSARY.md` at the repo root — the single canonical vocabulary for the whole
project. Create it **lazily**: when neither exists, write the root `GLOSSARY.md` the
first time a term is worth pinning down.

## Already have a glossary somewhere else?

Many repos already carry an informal glossary inside a README, an `AGENTS.md` /
`CLAUDE.md`, or a wiki. Don't create a second, competing one. Instead, offer to
**consolidate**: move the terms into `GLOSSARY.md` and leave a one-line cross-reference
where they used to live (e.g. "Canonical glossary: `GLOSSARY.md`"). One home, linked
from everywhere — never the same term defined in two places that can drift apart.
