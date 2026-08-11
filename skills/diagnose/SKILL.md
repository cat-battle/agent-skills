---
name: diagnose
description: Find the cause of one named misbehavior by experiment — reproduce it first, kill hypotheses with predictions rather than collecting plausible ones, and prove causation by making the symptom appear and disappear on demand. Reports the causal chain, the fix location, and everything else the cause reaches; installs instruments but reverts them, and does not fix. Use when verify-work returns failed, or when the user says "why is this broken", "debug this", "find the root cause", "it works locally but not in CI".
---

# Diagnose

Find the cause of **one named misbehavior** by experiment, and prove it is the
cause before reporting it.

## The reproduction is ground truth

Diagnosis begins when the symptom is reproducible on demand, and not before. A
hypothesis formed without a reproduction cannot be tested, only argued for — and
the argument always sounds good, because it was built backwards from a suspicion.
If it cannot be reproduced, that is a verdict of its own, not permission to guess.

## Plausible is not causal

The characteristic failure here is not missing the suspicious line. It is finding
one too easily: code that *looks* wrong is abundant, and in a broken system every
oddity attracts the blame.

**A suspect becomes a cause only when you can toggle it** — move that one thing
and watch the symptom appear and disappear. Until then it is a lead, and the
report says so.

## Hard stop: diagnose, don't fix

**This skill explains. It does not repair.** Do not patch, refactor, upgrade a
dependency, or "clean up while I'm here". Two reasons, both load-bearing:

- Repairing mid-investigation destroys the record of what was broken.
- Change three things and watch the symptom vanish, and you have learned nothing
  about which one mattered — while spending the reproduction you needed to find
  out.

If the fix is one obvious line, say so, cite it, and let the user decide. Naming a
fix is in scope; applying it is not.

## Every change is an instrument

Unlike `recon`, this skill runs and writes: log lines, a breakpoint, a stubbed
clock, a stashed commit during bisect. That is legitimate; leaving it behind is
not. **The tree ends as it started** — track every instrument, remove them all
before reporting, and say what went in and that it came out. An instrument
mistaken for a fix is worse than the original bug.

## When to use

- `verify-work` returned **failed** and the cause is now the question.
- A test, build, or job fails and the reason is not obvious from the output.
- Behavior differs across environments — passes locally, fails in CI.
- Do NOT use to answer "how does this work?" on a healthy system. That is `recon`,
  which is read-only and cheaper.
- Do NOT use to decide whether the behavior is *wrong*. If the disagreement is
  about intent, see *When it isn't a bug*.

## Process

1. **Fix the symptom.** One sentence, observable, with the exact invocation and
   the actual output. "It's broken" is not a symptom; "`POST /orders` returns 500
   on the second call in a session" is. If the symptom cannot be stated this way,
   stop and get it — everything downstream inherits this sentence.
2. **Reproduce.** Deterministically if you can; if intermittent, record the hit
   rate over a stated number of runs, because that rate is the measuring
   instrument for every experiment that follows. No reproduction → go to the
   `not reproducible` verdict. Never proceed on a symptom you have only been told
   about.
3. **Regression or never-worked?** Ask before searching — the two have different
   searches, and one command usually settles it. A regression bisects *history*; a
   never-worked bisects the *system*. See `references/bisection.md`.
4. **List hypotheses, with predictions.** Each states what you would observe if it
   were true, and what observation would kill it. Three or four is plenty. A
   hypothesis with no killing observation is a hunch — discard it or sharpen it.
5. **Bisect, don't sample.** Each experiment halves the remaining space and moves
   **one variable**. Record prediction → observation → survived / killed. Killing
   a hypothesis is progress and gets reported as such.
6. **Toggle to prove cause** (below). No toggle, no cause.
7. **Separate the crash site from the cause** (below), and name where the fix
   belongs.
8. **Map the blast radius.** Find the other paths that reach the same cause. A
   real cause almost always has more victims than the one reported.
9. **Report, revert the instruments, stop.** Do not continue into the fix.

Spend a bounded budget, roughly a handful of experiments. If the space is not
narrowing, stop and report **narrowed** — a bounded region with three hypotheses
killed is a useful result, and it beats a confident wrong answer.

## The toggle

For the cause you are about to report, answer: **what makes the symptom come
back?**

Then show it, both directions:

- Restore the suspected cause → symptom returns.
- Remove or neutralize it → symptom goes.

One variable moves. Anything else that changed between the two runs — a rebuild, a
restarted process, a reseeded store — makes the toggle inconclusive, and it gets
reported inconclusive rather than quietly counted.

Where a toggle is genuinely impossible — production-only data, a third-party
outage, a race you can observe but not schedule — say so explicitly and downgrade
the finding to `suspected`. That label is the whole point: it tells the reader the
chain was reasoned, not demonstrated.

## The crash site is not the cause

The null dereference at line 42 is where the failure *surfaced*. Why the value was
null is the cause, and it usually lives in a different file, often behind a
boundary the symptom never mentions.

Report both, as a chain: **cause → mechanism → symptom**, each link cited. Then
name the fix location explicitly — it is normally the cause, occasionally a
missing guard at the boundary between them, and almost never the crash site
itself. A patch at the crash site converts a wrong answer into a silent one.

Every claim in the chain is either observed or reasoned. Mark which. A chain with
one unobserved link is still worth reporting, with that link flagged.

## Evidence, not narration

Each experiment in the report carries:

- **The invocation** — exact, runnable as written.
- **The prediction** — what you expected before running it.
- **The observation** — quoted output, trimmed to what settles it. Not "it failed
  as expected".
- **The conclusion** — which hypothesis this killed or survived.

The prediction is recorded **before** the observation, and never rewritten
afterward. A prediction edited to match what happened turns every experiment into
a confirmation, which is exactly the failure mode this skill exists to prevent.

Facts you needed from the code but did not test — who calls this, what the default
is — come from `recon`, and keep its `confirmed` / `inferred` labels. Without
`recon`, search directly and label findings the same way.

## Verdicts

- **diagnosed** — cause identified, toggle demonstrated both directions, blast
  radius named, fix location stated.
- **suspected** — the chain is reasoned and cited but the toggle could not be run.
  Name what blocked it. Not the same as diagnosed; never round up.
- **narrowed** — bounded to a region, not pinned. Say what was ruled out, what
  remains, and the next experiment. Legitimate on a spent budget.
- **not reproducible** — the symptom did not occur under the conditions tried.
  List them: environment, data, timing, concurrency, sequence. This is **not** a
  finding that the system is fine, and must never be reported as one.
- **works as designed** — the behavior is intentional and cited as such. The
  disagreement is about intent, not correctness. See below.

## When it isn't a bug

Some investigations end at a decision rather than a defect: the behavior is what
the code was built to do, and someone expected otherwise. Say so plainly and stop.
Do not relabel it a bug to justify the search, and do not change it — altering
deliberate behavior is a product decision. Hand it to the user, or to
`task-interview` if it needs specifying.

## Output

A report in the conversation, always. Additionally, when the symptom came from a
ticket in `.claude/kanban/<slug>/`:

- Append the chain and evidence to the ticket under `## Diagnosis`, dated.
- Leave the card where it is. Diagnosis does not advance a card — the work isn't
  done, it is now understood.

Status changes and code edits belong to other skills. This one writes findings.

## References

- `references/bisection.md` — history / layer / input / config bisect, choosing
  between them, and handling intermittent failures. Read at step 3.
- `references/failure-signatures.md` — symptom classes and the mechanism families
  that produce them, per system type. Read at step 4 to aim the hypotheses, never
  to skip testing them.
- `references/observation.md` — getting evidence out of a running system, the
  observer effect, and instrument hygiene. Read at step 5 when the failure is not
  visible from the outside.
