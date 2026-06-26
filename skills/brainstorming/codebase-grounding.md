# Codebase Grounding (read-only mapping pass)

A design or a plan is only as correct as its picture of the code it builds on.
When a load-bearing claim rests on *how existing code actually works* — a path a
task will touch, a pattern to mirror, an interface to consume, a boundary
assumption to spike — ground it by **reading the code**, not by recalling it.
This is the read-only pass that `brainstorming`'s "verified current state" and
`writing-plans`' "de-risk first" / "mandatory reading" steps invoke.

It is read-only by construction, so it is one of the passes you may delegate to
a sub-agent (it returns a file:line-cited artifact; you build the design or plan
on the artifact). For a small area, do it inline; for a broad subsystem,
delegate it.

## The hard rules

- **Every claim cites `file:line`, verified by reading — never from memory.** The
  function you remember was three commits ago. An uncited claim about existing
  behavior is a guess.
- **grep finds *where*; reading confirms *what*.** A `cache.get` match may also
  delete-on-miss or wrap a remote call. The match tells you a name exists, not
  what the code does.
- **Don't assume layout.** Run `ls` / read the config; don't assume `src/`. Paths
  hallucinate in monorepos and polyglot trees.

## The pass

1. **Scope it.** Two sentences: `I am mapping <X> in order to <Y>.` and `I am not
   mapping <Z>.` Time-box it (~30 min for a feature, ~90 for a subsystem). The
   out-of-scope sentence is what stops the map from sprawling.
2. **List entry points** — every place execution can *enter* the area (routes,
   CLI commands, queue consumers, jobs, event listeners, exported functions),
   each as `<file:line> — <what triggers it>`. More than ~10 entry points means
   the scope is too wide — return to step 1.
3. **Trace & read.** Read each entry point's body — line by line, no skimming —
   noting every outward call with `file:line`. Follow one level deep; decide if a
   second is truly needed (most maps don't). Record **Surprises** (code that
   doesn't match its name; defensive code hinting at a past bug) and **Open
   questions** as you go.
4. **Write the artifact** — present tense ("This module routes…", not "I
   explored…"), ≤ ~300 lines, five sections: *Scope / Entry points / Call graph
   (file:line) / Surprises / Open questions*. Grow it during the trace; don't
   reconstruct it from memory at the end. A non-empty Open-questions section is
   required — empty means you stopped recording uncertainty.

**The test:** if what you produced is not a file you could hand a teammate, you
have not mapped the codebase — you have read some code. The design's "verified
current state" and the plan's file paths, Mirror snippets, and spike answers all
draw from this artifact.

## Delegating it

To delegate, dispatch a read-only sub-agent: give it the scope sentences and the
area to map, and have it return the artifact above. It uses Grep + Read only and
must not edit anything. The main agent consumes the artifact — the design and the
plan are still authored on the main agent.
