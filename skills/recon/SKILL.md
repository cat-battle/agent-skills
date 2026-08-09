---
name: recon
description: Answer one specific question about a codebase with the smallest search that settles it, and report findings as cited, confidence-labeled evidence. Bounded and read-only — it is not a codebase survey and does not build general understanding. Use when a named question is plausibly answerable from the repo, when another skill needs grounded facts before proceeding, or when the user says "find out how X works", "does this repo already do Y", "where does Z happen".
---

# Recon

Answer **one named question** from the repo, with the smallest search that
settles it, and report evidence a caller can trust.

Recon is not exploration. Exploration is unbounded by nature — there is always
another file. A recon has a question, a budget, and a stopping condition, and it
ends in a verdict even when the verdict is "the repo doesn't say".

## Read-only contract

**Recon reads. It never writes, edits, runs, or installs anything.** No files
modified, no dependencies added, no code executed to see what happens. If a
question can only be settled by running something, that is out of scope — say so
and hand it back. This keeps recon safe to invoke reflexively from any skill or
phase, including ones under a no-implementation hard stop.

## When to use

- Another skill needs a fact before it can proceed, and the fact is in the code.
- The user asks a locatable question — how something works, whether something
  exists, where a behavior lives, what calls what.
- Do NOT use to "get familiar with the codebase". That has no stopping condition
  and will spend an unbounded budget.
- Do NOT use for questions about intent, priority, or desirability. See *What the
  code cannot tell you*.

## Process

1. **Sharpen the question.** State it so that evidence could settle it either
   way. "How does auth work?" is a survey; "which middleware rejects an expired
   token, and what status does it return?" is a recon. If you can't sharpen it,
   the problem is the question — say so rather than searching hopefully.
2. **Predict the answer's shape.** Before searching, say what would count as
   finding it: a function, a config key, a route, a test, an absence. This is
   what makes it possible to know you're done.
3. **Pick entry points.** Start from names in the question, then the module that
   owns them, then its callers. See `references/search-patterns.md` for tactics.
4. **Search within pertinence** (table below). Read the narrowest thing that
   settles the question — a function body, not the file; a file, not the package.
5. **Spend the budget, then stop.** Roughly a handful of targeted searches. If it
   isn't converging, stop and escalate — an unconverged recon is a finding, not a
   failure.
6. **Report.** Use the output contract below. Cite everything.

## Pertinence

You are looking for the answer to *the question*, not for general understanding.

| Pertinent | Not pertinent |
| --- | --- |
| The symbol or module named in the question | Unrelated packages |
| Its direct callers and importers | The whole dependency graph |
| Config, manifests, lockfiles bearing on it | Every config in the repo |
| Tests covering the behavior in question | The full test suite |
| The entry point for the affected path | Entry points generally |
| Git history when the question is "why" | Git history as browsing |

When tempted past this line, check whether a *new* question has appeared. If so,
it is a separate recon with its own budget — not an extension of this one.
Scope creep in recon looks exactly like diligence, which is what makes it
dangerous.

## Output contract

Report findings, not narration. Each finding carries three things:

- **The claim** — one sentence.
- **The citation** — `path/to/file.py:42`. Never a claim without one.
- **The confidence** — one of:
  - `confirmed` — read directly. The code says this.
  - `inferred` — deduced from surrounding evidence, not stated outright. Say what
    the inference rests on.
  - `absent` — searched and did not find it. Name where you looked, so the caller
    can tell a real absence from a shallow search.

Then a verdict on the question: **answered**, **partially answered** (with what's
missing), or **not answerable from the repo** (with what to ask the user instead).

`absent` and `not answerable` are first-class results. A recon that reports
"couldn't find it, looked in these four places" is more useful than one that
keeps digging until it finds something adjacent and presents it as the answer.

Callers that produce documents should carry findings through as
`[code: path/to/file.py:42]`, preserving the confidence label — an `inferred`
finding promoted to fact in a downstream document is a bug with a long fuse.

## What the code cannot tell you

Never answerable by recon; escalate to the user:

- Intent, priority, and why now.
- Whether current behavior is *intended* or merely *current*.
- What is deliberately out of scope.
- Audience, risk tolerance, and how much this matters.

That second one is the trap. The code tells you what happens; it never tells you
whether anyone wanted it to. Reading a behavior is not confirming a requirement,
and a recon that blurs the two launders an accident into a spec.

## References

- `references/search-patterns.md` — entry points by project type, and search
  tactics. Read at step 3 when the entry point isn't obvious.
