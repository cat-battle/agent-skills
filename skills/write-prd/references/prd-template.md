# PRD template

The target artifact. Sections marked **required** must be present or explicitly
marked *N/A* with a reason; the rest are included when they apply.

This is an *implementation-ready* PRD — written to be handed to whoever (or
whatever) builds the thing. It says what must be true when the work is done and
why, not how to build it. Design and task breakdown come after approval.

Label every non-obvious statement with its source: `[user]`, `[assumed]`,
`[code: path/to/file.py:42]`, or `[prototype]` for something established by a
throwaway experiment.

The PRD is the *only* file this phase writes into the project. No implementation
accompanies it — see the hard stop in SKILL.md.

<!-- SCAFFOLD: section set is a starting point. Prune or extend as real PRDs
     show which sections earn their keep. -->

---

## Header

Title, date, status (`draft` / `approved`), and a one-line summary someone can
read in isolation and understand.

## 1. Problem — required

What is wrong today, for whom, and what it costs them. Written in the user's
terms, before any solution language. If this section can't be written without
naming the proposed solution, the interview hasn't found the problem yet.

## 2. Goal — required

One sentence: the state of the world when this is done. Followed by
**non-goals** — the plausible adjacent things this deliberately does not do.
Non-goals are the highest-value section of most PRDs; they are what stops scope
from drifting silently.

## 3. Users and scenarios — required

Who touches this, and the concrete situations they're in. One or two scenarios
beats a persona table. For each: what they're doing right before, what they
want, what "worked" looks like to them.

## 4. Requirements — required

The core. Split into:

- **Functional** — what the system must do. Number them (`FR-1`, `FR-2`) so
  later discussion and acceptance criteria can reference them. Each one
  testable, each one traceable to a scenario or a stated constraint.
- **Non-functional** — performance, scale, security, accessibility,
  compatibility, operability. Only the ones with a real bar; a made-up latency
  target is worse than none.

Mark each as **must** / **should** / **won't (this round)**. "Won't" items are
recorded, not dropped — they're the evidence that a tradeoff was made knowingly.

## 5. Acceptance criteria — required

How completion is verified, tied back to requirement IDs. Prefer things that can
actually be run: a test, a command, a reproducible manual check. This section is
the closure test for the *work*, the way the frontier's closure test is for the
*interview*.

## 6. Dependencies — required

Four classes — technical, data, external, sequencing. For each: what it is,
whether it exists today, who owns it, and the fallback if it's unavailable. An
unowned dependency is a risk; move it to §9.

## 7. Constraints

Stack and version pins, conventions to follow, things that may not be touched,
deadlines, dependencies that may not be added.

## 8. Findings from the code

What recon established, with paths and confidence labels, plus anything a
throwaway prototype proved. Kept separate from user statements so a misreading is
visible and correctable rather than laundered into a requirement. Findings are
evidence for requirements, never requirements themselves — and an `inferred`
finding is evidence that has not been confirmed.

## 9. Risks and open questions — required

Unresolved items with an owner and what each blocks, plus known risks and how
they'd be mitigated or detected. Deferred frontier items land here by name — this
is where the interview admits what it could not close, which is what keeps the
rest of the document trustworthy.

## 10. Assumptions — required

Defaults chosen without asking, stated plainly so they are cheap to overturn.
If an assumption turning out wrong would invalidate a requirement, say which one.

## 11. Rollout and phasing

Milestones or slices if the work doesn't land in one piece. What ships first,
what's behind a flag, what the migration or backfill looks like, how it's undone
if it goes badly.

---

## Mapping from the interview

Applies when `task-interview` handed off; harmless to ignore when running
standalone.

| Interview material | Lands in |
| --- | --- |
| Outcome / why now | §1 Problem, §2 Goal |
| Scope boundaries | §2 Non-goals, §4 "won't" |
| Users, actors from follow-up ladders | §3 Users and scenarios |
| Answers about behavior, states, data shapes | §4 Functional requirements |
| Answers about speed, scale, safety | §4 Non-functional |
| Success criteria | §5 Acceptance criteria |
| Dependency pass | §6 Dependencies |
| Stack, conventions, don't-touch | §7 Constraints |
| Targeted exploration | §8 Findings from the code |
| Deferred frontier items | §9 Open questions |
| Assumed frontier items | §10 Assumptions |
| Session definition of done, sequencing | §11 Rollout and phasing |

Any frontier item that maps nowhere was probably not worth asking — note it, and
prune that question from `dimensions.md` next time.
