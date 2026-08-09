---
name: verify-work
description: Verify that a built change actually does what its ticket claimed — run the ticket's own demo through the real entry point, prove the check could have failed, and separate what the agent can settle from what only a human can judge. Produces evidence and a verdict; does not repair failures. Use when a slice is built and needs checking, before a kanban card leaves "doing", or when the user says "test this", "verify it", "does this work", "QA this", "did we break anything".
---

# Verify Work

Establish whether a change does what it was supposed to do, using evidence
someone else could re-run — and name precisely what a human still has to judge.

## The demo is the test

`slice-work` writes every ticket's **Demo** before the code exists: an observable
check through the system's real entry point. That is not documentation. It is the
acceptance test, written while nobody knew how it would be implemented.

So verification does not begin by reading the implementation and deciding what to
check. **It begins by reading the ticket.** A check derived from the code will
agree with the code — including where the code is wrong.

When the demo and the behavior disagree, the demo wins until a human says
otherwise. Rewriting a demo to match what was built is how a board turns green
while the product stays broken.

## Hard stop: verify, don't repair

**This skill reports. It does not fix.** A failing check is a finding — the
finding is the deliverable. Do not patch the code, relax an assertion, widen a
tolerance, retry until green, or edit the ticket's demo. Do not mark anything done
because the fix "is obvious"; if it is obvious, say what it is and let the user
decide whether you apply it.

Verifying and repairing in one motion destroys the signal, because the record no
longer shows what was broken.

## When to use

- A ticket from `.claude/kanban/` has been built and is leaving `doing`.
- The user asks whether a change works, or wants regression coverage checked
  around something just built.
- Standalone, on a change with no ticket — write the check from the stated intent
  **before** reading the diff. See step 1.
- Do NOT use to write a test suite from scratch, or to raise coverage. That is
  implementation work with its own ticket.
- Do NOT use to decide whether the behavior is *desirable*. That is intent, and
  intent is human — see `references/hitl-triggers.md`.

## Process

1. **Fix the claim.** Name exactly what is being verified: a ticket ID, or one
   sentence of intent. Pull the ticket's **Demo**, **Slice**, **Seams**,
   **Advances**, and **Rests on**. With no ticket, write the demo now, from
   intent, before opening the diff — and show it to the user as the thing you are
   about to check.
2. **Classify the checks.** Split the demo into what the agent can settle and what
   only a human can, using `references/hitl-triggers.md`. Do this **before running
   anything**, so the human's queue is known at the start rather than discovered
   at the end.
3. **Confirm the seam is real.** The ticket said which boundaries it crosses for
   real. Verify the check actually crosses them — real process, real transport,
   real store — and name anything stubbed. `references/check-patterns.md` has the
   real-vs-fake tells per system type.
4. **Run it through the real entry point.** The command, request, click path, or
   published message named in the demo. Record the invocation and the actual
   output verbatim.
5. **Run the negative control** (below). A green that could not have been red
   proves nothing.
6. **Check the slice boundary.** The ticket said what it deliberately left out.
   Behavior beyond that line is a finding too — unplanned scope is unverified
   scope, even when it works.
7. **Check the blast radius.** Run the existing checks that already cover the
   seams this change touched. Not the whole suite as a ritual — the ones whose
   subject the diff moved.
8. **Package the human's part.** One batched request: what to look at, how to
   reach it, what would count as wrong, and what stays blocked until they answer.
   Format in `references/hitl-triggers.md`.
9. **Report the verdict** (below), then update the board if one exists. Stop.

## Negative control

For each check that passed, answer: **what would have made this fail?**

Then demonstrate it, cheaply — stash the change and watch the check go red, feed
the input the rule is supposed to reject, point the client at the wrong record,
break the config the code claims to read. One control per check is enough.

The failures this catches are the ones that survive review: a test asserting a
mock, a request hitting a stale process, an assertion on a field that is absent
in both the pass and fail case, a happy path that never reached the code under
test. All of them look exactly like success.

If a control cannot be run, say so and downgrade the check to `weak` in the
report. A green with no control is a claim, not evidence.

## Evidence, not assertion

"Tests pass" is not a finding. Every claim in the report carries:

- **The invocation** — the exact command, request, or path, runnable as written.
- **The observed output** — quoted, trimmed to the part that settles it. Not
  paraphrased, not summarized as "returned successfully".
- **The control** — what was done to show the check could fail, and what happened.

Anything you did not observe yourself is not evidence. If a step was skipped,
report it skipped; a gap named is cheap, and a gap smoothed over gets trusted.

## What only a human can settle

An agent cannot verify how something looks, whether wording is right, whether the
result is what the requester meant, or anything whose consequences are
irreversible or outward-facing. Those are **routed, not guessed**.

The full taxonomy, the anti-pattern of over-routing, and the request format live
in `references/hitl-triggers.md`. Read it at step 2 — it is short, and the
classification changes what you run.

The trap: a passing check proves the system does what the ticket said. It never
proves the ticket said the right thing. That question always belongs to a person,
and no amount of green moves it.

## Verdicts

One per ticket, stated plainly:

- **verified** — every check passed with a control, and nothing is owed to a
  human.
- **verified pending human** — the agent-side checks passed; named human checks
  are outstanding. **This is not done.**
- **failed** — a check failed. Report the invocation, the expected result, the
  observed result, and nothing else. Diagnosis only if asked.
- **unverifiable here** — the check cannot run in this environment. Name what
  blocks it and what would unblock it. Distinct from failed, and never collapsed
  into it.

## Output

A report in the conversation, always. Additionally, when a local board exists at
`.claude/kanban/<slug>/`:

- Append the evidence to the ticket file under `## Verification`, dated.
- Move the card: `review` when a human is owed something, `done` only on
  **verified**, back to `doing` on **failed**. A card never reaches `done` with an
  open human check — that transition is the human's to make, after they answer.

Statuses are `backlog | ready | doing | review | done` — a derived copy; the
format is owned by `slice-work/references/local-board.md`.

Changing a card's status is the only write this skill makes. It does not edit the
plan, the demo, or the code.

## References

- `references/hitl-triggers.md` — the classes of check an agent cannot settle,
  the tells for a false HITL, and the request format. Read at step 2.
- `references/check-patterns.md` — driving the real entry point per system type,
  real-seam vs fake-seam tells, and negative-control recipes. Read at step 3.
