---
name: slice-work
description: Turn an approved PRD (or a comparable spec) into a kanban backlog of small vertical slices — each one demonstrable through the system's real entry point, each one carrying its own integration check, ordered by risk. Produces the board only; writes no implementation code. Use when the user says "break this PRD into tickets", "make a kanban", "build the backlog", "split this into tasks", or once write-prd has an approved document.
---

# Slice Work

Convert an approved spec into a backlog of **vertical slices**: tickets that each
change what the system observably does, and each prove it where the seams are
real.

## The mapping is not one-to-one

A PRD is organized by requirement. Requirements are a *specification* structure —
they describe capabilities, and capabilities decompose neatly into layers. That
is why the default conversion produces horizontal tickets: "add the model", "build
the API", "wire up the UI".

**One requirement usually spans several slices. One slice usually advances several
requirements.** Never emit `FR-1 → K-001`. If your ticket list is the requirement
list renumbered, you have transcribed the PRD, not sliced it.

## The vertical slice test

> A ticket is vertical if you can demonstrate it through the system's **real entry
> point** — the endpoint, the CLI invocation, the UI action, the consumed queue
> message.

If the only way to show it working is a unit test or a mock, it is horizontal.
Rewrite it or merge it into the slice it serves. This test is the whole skill;
everything below is machinery for passing it.

## Sizing test

**The demo needs no "and".** One reviewable change, one demonstrable behavior. If
describing what a ticket delivers requires a conjunction, it is two tickets.

Small is not the goal in itself — small is what makes the integration check cheap
enough to run every time.

## When to use

- After `write-prd` produces an approved PRD and the user wants work planned.
- Standalone, when the user has any spec, issue, or design doc and wants a
  backlog with real slices rather than a task list.
- Do NOT use to *decide* what to build. If the spec has holes, name them and hand
  back — see *When the spec resists*.
- Do NOT use for a single-ticket change. If the whole thing is one slice, say so;
  a board of one is overhead.

## Hard stop: tickets only

**This skill produces a board and nothing else.** No implementation, no
scaffolding, no starter branches, however clear the first ticket looks. Planning
the work is not permission to start it. It also does not silently repair the PRD:
if a requirement is untestable or contradictory, that is a finding to report, not
something to fix inside a ticket description.

## Process

1. **Read `references/board-template.md`.** It defines the ticket fields and the
   board file — the artifact spec.
2. **Inventory the spec.** Pull the numbered requirements and their must/should/
   won't marks, the acceptance criteria, the dependencies, the open questions, and
   the assumptions. "Won't (this round)" items are not tickets; they are the
   boundary you slice up to.
3. **Find the thinnest end-to-end path.** The one traversal of the system that
   touches every real seam with the least behavior. This becomes **K-001, the
   walking skeleton** — always first, always mandatory.
4. **Slice the rest along behavior, not layers.** Split by path, rule, data
   variation, interface, operation, or quality. `references/slicing-patterns.md`
   has the axes and a worked example; read it when a requirement resists.
5. **Write each ticket's demo before its description**, and mark who can check it.
   If you cannot state the observable check, the slice is not vertical yet, and no
   amount of description will fix that. If the check needs a person — how it
   looks, whether the wording is right, an irreversible action — say so in
   **Verified by** now, so the human is queued rather than surprised at the end of
   the slice.
6. **Declare every stub.** A slice may stub a seam. It may not stub one silently —
   name the stub and the ticket that removes it.
7. **Order by risk, not convenience.** Riskiest and least-understood first, since
   that is where the plan is most likely wrong and earliest is cheapest to learn.
   Dependency order only breaks ties.
8. **Run the closure test** (below). Then play the board back and stop.

## Shifting integration left

Four mechanisms. They work together; the first is load-bearing.

- **Every ticket's acceptance is an integration-level check.** Not "unit tests
  pass" — a check that crosses at least one real seam. If acceptance is written
  this way per ticket, integration happens per ticket by construction.
- **The walking skeleton is the first ticket.** Trivial behavior, every real seam
  crossed, deployed and running in CI. It carries almost no user value and ships
  anyway: after it, nothing is ever *integrated* again, only extended.
- **No integration ticket at the end.** If the board needs a "wire it together"
  or "end-to-end testing" ticket, the slices were horizontal. Treat the urge to
  write one as a failed slice, and re-slice.
- **Contract tests where you genuinely cannot integrate.** A third-party API you
  don't control gets a contract test pinned to its documented shape, plus a named
  risk — not a mock that agrees with itself forever.

The same argument applies to the human checks: a ticket needing a person's
judgment says so when it is written, not when it is finished. `verify-work` runs
these demos and routes what it cannot settle; a demo it has to reinterpret was
underspecified here.

## Open questions are not slices

An unresolved item from the PRD is one of two things, never a vague ticket:

- **A spike** — time-boxed, phrased as a question, delivering knowledge and a
  recommendation rather than working code. Give it a deadline and say what
  happens if it expires unanswered.
- **A blocker** — recorded on the ticket it blocks, with an owner.

A ticket whose estimate quietly absorbs an open question is how a plan gets
wrong without anyone noticing.

## Traceability

Carry the chain through. Each ticket names the requirement IDs it advances, and
marks partial advancement as partial. Where a ticket rests on an `assumed`
statement, an `inferred` code finding, or an open question, say so on the ticket —
those are the tickets most likely to change shape, and knowing which they are is
what makes the order defensible.

A finding that arrived `inferred` never becomes fact by being written into a
ticket.

## Closure test

Both directions, before playing the board back:

- **Every `must` requirement is advanced by at least one ticket.** A must with no
  ticket is either forgotten scope or a hidden non-goal — resolve which.
- **Every ticket advances at least one requirement**, or it is a spike, or it is
  scope creep. Name which.

Report any leftovers on either side rather than inventing a ticket to absorb them.

## When the spec resists

Some requirements do not slice thinly — a format migration, a cryptographic
handshake, an atomic cutover. Do not fake a slice to satisfy the rule. Say the
requirement is indivisible at this size, give the smallest honest ticket, and
record why. One admitted big ticket is safer than four fake small ones that can
only be demonstrated together.

If sizing a slice depends on what the code actually does today, invoke `recon`
with a named question. Without it, mark the slice's size `unverified` and move on;
never guess in silence.

## Output

`.claude/kanban/<slug>.md`, matching the PRD's slug where there is one. This is
the **plan**: generated, reviewable in a diff, and unchanged by work happening.
All tickets start in **Backlog** or **Ready** — this skill does not move cards.
`verify-work` is what moves them, once a slice is built and checked.

Exporting is always a **separate, explicit request**, never an unasked-for
finishing touch. Two targets:

- **Local working board** — `.claude/kanban/<slug>/`, one file per ticket with a
  `status` field. This is where cards actually move, and where the plan stops
  being authoritative. Re-exporting reconciles rather than overwrites: a ticket
  in flight is never edited and a dropped ticket is never deleted. Rules in
  `references/local-board.md`.
- **GitHub Issues** — writes to a shared system, so confirm the repo and the
  ticket list first, and check for duplicates before creating anything.

## References

- `references/board-template.md` — ticket fields, plan file layout, and the
  GitHub Issues export mapping. Read at step 1.
- `references/slicing-patterns.md` — split axes, a worked example, and the tells
  that a ticket went horizontal. Read at step 4 when a requirement resists.
- `references/local-board.md` — per-ticket file format and the re-export drift
  rules. Read when exporting a local working board.

PRD section numbers referenced here are owned by
`write-prd/references/prd-template.md`; this skill keeps no copy of the section
list, only the handful of fields it consumes.
