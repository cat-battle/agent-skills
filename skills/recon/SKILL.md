---
name: recon
description: Answer one specific question about a codebase with the smallest search that settles it, and report findings as cited, confidence-labeled evidence. Orients to the tree's shape first — monorepo, polyrepo, submodules, vendored, sparse — so "not found" is never confused with "not in this checkout". Bounded and read-only; it is not a codebase survey and does not build general understanding. Use when a named question is plausibly answerable from the code, when another skill needs grounded facts before proceeding, or when the user says "find out how X works", "does this repo already do Y", "where does Z happen".
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

1. **Orient.** Establish what kind of tree this is before searching it — single
   repo, monorepo, submodules, vendored, sparse, or one node of a polyrepo. The
   checks are in `references/topology.md` and cost a command or two. Skip this
   only when a previous recon this session already established the shape.
   **Do not assume the answer is in this checkout.** Deciding that up front is
   what separates "not found" from "not here", and those are different answers.
2. **Sharpen the question.** State it so that evidence could settle it either
   way. "How does auth work?" is a survey; "which middleware rejects an expired
   token, and what status does it return?" is a recon. If you can't sharpen it,
   the problem is the question — say so rather than searching hopefully.
3. **Predict the answer's shape.** Before searching, say what would count as
   finding it: a function, a config key, a route, a test, an absence. This is
   what makes it possible to know you're done.
4. **Pick entry points.** Start from names in the question, then the module that
   owns them, then its callers. Scope to the owning package, not the whole tree.
   See `references/search-patterns.md` for tactics.
5. **Search within pertinence** (table below). Read the narrowest thing that
   settles the question — a function body, not the file; a file, not the package.
6. **Spend the budget, then stop.** Roughly a handful of targeted searches. If it
   isn't converging, stop and escalate — an unconverged recon is a finding, not a
   failure. If the trail leaves the checkout, stop immediately; searching harder
   locally cannot find code that isn't there.
7. **Report.** Use the output contract below. Cite everything.

## Pertinence

You are looking for the answer to *the question*, not for general understanding.

| Pertinent | Not pertinent |
| --- | --- |
| The symbol or module named in the question | Unrelated packages |
| The **package that owns it** | Every package in the workspace |
| Its direct callers and importers | The whole dependency graph |
| Neighbors along **declared** dependency edges | Neighbors by proximity on disk |
| Config, manifests, lockfiles bearing on it | Every config in the repo |
| Tests covering the behavior in question | The full test suite |
| The entry point for the affected path | Entry points generally |
| The contract artifact at a service boundary | Every consumer of that service |
| Dependency source, when the question is about the dependency | Dependency source as background |
| Git history when the question is "why" | Git history as browsing |

When tempted past this line, check whether a *new* question has appeared. If so,
it is a separate recon with its own budget — not an extension of this one.
Scope creep in recon looks exactly like diligence, which is what makes it
dangerous.

## Output contract

Report findings, not narration. Each finding carries three things:

- **The claim** — one sentence.
- **The citation** — `path/to/file.py:42`, qualified by package or repo when the
  tree has more than one. Never a claim without one.
- **The confidence** — one of:
  - `confirmed` — read directly. The code says this.
  - `inferred` — deduced from surrounding evidence, not stated outright. Say what
    the inference rests on.
  - `absent` — searched and did not find it. Name where you looked, so the caller
    can tell a real absence from a shallow search.
  - `external` — it exists, but not in this checkout: another repo, an
    uninitialized submodule, outside the sparse cone. Name where it lives if
    determinable, and what access would settle it.
- **Third-party marker**, when the citation points into a dependency rather than
  the project's own code: name the package and version. Library behavior is a
  fact about a pinned version, not a decision the project made.

Then a verdict: **answered**, **partially answered** (with what's missing), **not
answerable from this checkout** (with which repo or artifact would answer it), or
**not answerable from code** (with what to ask the user instead).

`absent`, `external`, and both "not answerable" verdicts are first-class results.
A recon reporting "couldn't find it, looked in these four places" is more useful
than one that keeps digging until it finds something adjacent and presents it as
the answer. And `absent` vs `external` is the distinction that matters most to a
caller: one says the behavior doesn't exist, the other says you're looking in the
wrong repo. Never collapse them.

Callers that produce documents should carry findings through as
`[code: path/to/file.py:42]`, preserving the confidence label — an `inferred`
finding promoted to fact in a downstream document is a bug with a long fuse. An
`external` finding is usually also a dependency, and belongs wherever the caller
tracks those, not only in its findings list.

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

- `references/topology.md` — detecting the tree's shape, scoping rules per shape
  (monorepo, polyrepo, submodules, vendored, generated, sparse), and service
  boundaries. Read at step 1.
- `references/search-patterns.md` — entry points by project type, and search
  tactics. Read at step 4 when the entry point isn't obvious.
