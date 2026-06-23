# Changelog

This file documents the customizations carried by this fork (`pfurini/superpowers`,
branch `personal`) on top of upstream `obra/superpowers`. It does **not** track upstream
releases — see the upstream repository for those.

Fork base: `896224c` (merge-base with `upstream/main`).

All changes so far are skill-content only (`skills/`); no code, hooks, or configuration
are modified. Together they form one coherent theme: hardening the
`brainstorming → writing-plans → using-git-worktrees` pipeline with a spec / glossary /
ADR discipline and two worktree-timing fixes.

## Unreleased (fork-local)

### Added

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
