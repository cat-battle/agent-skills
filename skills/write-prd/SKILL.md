---
name: write-prd
description: Turn gathered material — interview answers, notes, a transcript, an issue thread, a rough spec — into an implementation-ready product requirements document with traceable, testable requirements. Produces the document only; writes no implementation code. Use when the user says "write a PRD", "write this up as a spec", "turn these notes into requirements", or when the task-interview skill has closed its question frontier.
---

# Write PRD

Convert gathered material into a PRD that someone could build from without
asking a follow-up question.

## When to use

- Invoked by `task-interview` once its frontier is closed.
- Standalone, when the user already has the material — notes, a transcript, an
  issue thread, a design doc — and wants it written up properly.
- Do NOT use to *gather* the material. If the input has holes, see *Missing
  material* below; if it's mostly holes, run `task-interview` instead.

## Hard stop: document only

**This skill produces a PRD and nothing else.** No implementation, no scaffolding,
no "starter" files, however obvious the code seems once the requirements are
clear. Writing the spec is not permission to build the thing it specifies — if
the user wants that, they will ask.

## Process

1. **Read `references/prd-template.md`.** It defines the sections, which are
   required, and what each must contain.
2. **Inventory the material.** Sort what you have by target section before
   writing anything. Gaps become visible now rather than mid-draft.
3. **Draft in template order.** Problem before Goal, requirements before
   acceptance criteria — the order is a dependency chain, not a formatting
   preference.
4. **Enforce the two rules** (below) on every line as you write.
5. **Number and cross-link.** `FR-1`, `FR-2`… so acceptance criteria, risks, and
   later discussion can reference specific requirements.
6. **Check completeness.** Every required section written, or marked *N/A* with a
   reason. No "TBD" standing in for something you could have asked.
7. **Play it back.** Show the PRD, ask for corrections once, then stop.

## The two rules

- **Traceability.** Every non-obvious statement is labeled with its source:
  `[user]`, `[assumed]`, `[code: path/to/file.py:42]`, or `[prototype]`. Nothing
  is unsourced, because an unsourced requirement is a guess wearing a suit.
  Findings arriving from the `recon` skill keep their confidence label — an
  `inferred` finding may not be stated as fact, and an `inferred` finding that a
  requirement depends on belongs in §9 as something to confirm.
- **Testability.** Every functional requirement is written so a reader can say
  whether a given implementation satisfies it. "Fast", "clean", and "intuitive"
  are not requirements — they are unfinished questions. Convert them into
  observations, or record them as open questions and say so.

## Missing material

Do not paper over a gap. For each one, choose and label:

- **Assume** — low-stakes and there's an obvious default. Goes in §10 Assumptions.
- **Ask** — it would change what gets built. Ask now; a handful of targeted
  questions is fine and is not a full interview.
- **Defer** — genuinely unresolvable today. Goes in §9 with an owner and what it
  blocks.

A PRD whose gaps are visible is useful. One whose gaps are smoothed over is worse
than no PRD, because it gets trusted.

## Output

`.claude/prds/<slug>.md`. Right-size it — a two-day feature's PRD is a page. The
template's sections are a checklist to consider, not a page count to hit.

## References

- `references/prd-template.md` — section-by-section spec for the document, and
  the mapping from interview material into it. This file is the **source of
  truth** for the section list; `task-interview` keeps a derived copy of the
  section names for its closure test.
