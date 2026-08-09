---
name: task-interview
description: Interview the user about a task before any work starts, following every clarification to completion and surfacing dependencies, then hand off to the write-prd skill to produce the document. Writes no implementation code. Use when the user says "interview me", "ask me questions first", "let's spec this out", "get on the same page", or when a request is large or ambiguous enough that guessing wrong would waste real work.
---

# Task Interview

Turn a vague request into an agreed understanding, complete enough that a PRD can
be written from it without guessing. The interview is not a fixed questionnaire —
it is a **frontier of open questions that is worked until it is empty**.

The PRD is what the frontier is worked *toward*: every question exists to fill a
section of it, and the section list in `references/dimensions.md` is the
scoreboard. The document itself is written by the `write-prd` skill at step 7.

## Hard stop: this phase does not write implementation code

**This phase produces questions, answers, and a PRD. Nothing executable.**

This holds even when the implementation seems obvious, even when the change is
small, even when the user's answers have effectively dictated the code. Being
able to write the code is the *sign the interview worked* — it is not permission
to start. A PRD delivered alongside unrequested code has destroyed the thing it
was for: the user's chance to disagree before effort is spent.

If the user asks you to start building, that is a new instruction and you follow
it — but they must actually ask. Silence is not consent, an approved PRD is not
consent, and enthusiasm about the plan is not consent.

The single exception is a throwaway prototype, under the conditions in
*Prototypes* below. It is narrow on purpose.

## When to use

- The user explicitly asks to be interviewed, questioned, or to spec something out.
- The request is multi-step or open-ended, and different readings would produce
  materially different work.
- Do NOT use for small, well-specified changes. One clarifying question inline
  beats a whole interview, and a one-file fix does not need a PRD.
- If the user already has the material and just wants it written up, skip to
  `write-prd`.

## Process

1. **Seed the frontier.** Read `references/dimensions.md` and turn each relevant
   dimension into an open question. Track them explicitly — an unwritten question
   gets dropped. Tag each with the PRD section it feeds, so gaps show up as empty
   sections rather than as questions you forgot to ask.
2. **Recon, only if pertinent.** For frontier questions plausibly answerable from
   the repo, invoke the `recon` skill — one question per invocation, named
   explicitly. Close what the repo can close before asking the user; never ask
   what the code already says. Carry findings forward with their citations and
   confidence labels intact. A finding marked `inferred` is not settled — it
   stays on the frontier as a question to confirm with the user.
   If `recon` isn't available, search directly but keep it question-scoped: name
   the question first, a handful of searches, then escalate to the user.
3. **Ask in rounds.** Use `AskUserQuestion` with the highest-leverage questions
   first — the ones whose answers change what the *other* questions should be.
   Offer concrete options with a recommended default so the user can click rather
   than compose. Cap each round at what fits the tool; the frontier holds the rest.
4. **Expand every answer.** The core loop, and it is mandatory:
   - After each answer, ask what that answer just made unknown, and push those
     follow-ups onto the frontier.
   - An answer naming a system, actor, format, or state you can't yet describe is
     an unexpanded branch. Follow it. The triggers and rungs are in
     `references/question-bank.md`.
   - Repeat from step 2 until the closure test passes. Do not exit the loop
     because a round "felt like enough".
5. **Dependency pass.** Walk all four classes below. Dependencies routinely open
   new branches — if they do, return to step 4.
6. **Close the remainder.** Every surviving frontier item ends in one of three
   states, none of them silent:
   - **Answered** — carried into its tagged PRD section.
   - **Assumed** — obvious default for a low-stakes unknown; carried into PRD §10.
   - **Deferred** — can't be settled now; carried into PRD §9 with who resolves it
     and what it blocks.
7. **Hand off.** Invoke the `write-prd` skill and pass it the handoff below. Do
   not write the PRD yourself — that skill owns the format.
8. **Stop.** The phase ends at the PRD. Say what the next step would be; do not
   take it.

### Closure test

The frontier is complete when all three hold:

1. **Every open question is Answered, Assumed, or Deferred** — none silent.
2. **One more pass over the answers surfaces no new branch.**
3. **Every required PRD section could be written from what you have** — no
   placeholders, no "TBD" standing in for a question you could have asked. A
   section that genuinely doesn't apply is marked *N/A* with a reason, which is a
   decision; leaving it blank is not.

Stop expanding a branch when the answer would no longer change what gets built,
who it affects, or how it is verified. That is the depth limit — not a question
count, and not a round count. A detail that is merely *interesting* is out of
scope; a detail that changes an interface, a dependency, or an acceptance
criterion is not.

### Dependencies

Probe all four classes; each maps to a row in PRD §6.

- **Technical** — modules, packages, services, schemas, migrations, or APIs this
  touches or is touched by. Which callers break? What has to ship first?
- **Data** — inputs required, their shape and source, and what must exist before
  the work can run at all.
- **External** — other people's decisions, credentials, access, third-party
  services, or approvals. Anything you can't unblock yourself.
- **Sequencing** — what must land before this, what waits on it, what can proceed
  in parallel.

For each: what it is, whether it exists today, who owns it, and what happens to
the task if it is unavailable.

### Prototypes

The one exception to the hard stop, and it is rare. Admissible only when a
frontier question is **empirical** — unanswerable by asking or reading, because
nobody knows until something is run. "Does this library actually support X?"
qualifies. "How should we structure this?" does not; that is design, and design
happens after approval.

All of the following must hold:

- **Declared first** — name the frontier question it answers before building. If
  you can't name it, this isn't a prototype, it's implementation with a euphemism.
- **Smallest thing that answers it** — it ends when the question closes, not when
  the feature works.
- **Outside the project tree** — scratchpad only. Nothing committed, no project
  file modified. If it must run in the repo to mean anything, get explicit
  permission and revert afterward.
- **Discarded, result recorded** — what survives is a `[prototype]` finding for
  PRD §8, not the code. Never offer it as a starting point for the real work;
  that is how an exception becomes a habit.
- **Not a foothold** — closing the question returns you to the interview.

## Handoff to write-prd

Pass, organized by target section:

- Every answer, attributed `[user]`.
- Assumptions made, attributed `[assumed]`.
- Recon findings with paths, attributed `[code: path:line]`.
- Prototype results, attributed `[prototype]`.
- Deferred items with owner and what they block.

Attribution is not decoration — `write-prd` enforces traceability and cannot
reconstruct a source you dropped.

## References

- `references/dimensions.md` — what to probe, and the PRD section each feeds.
  Read at step 1.
- `references/question-bank.md` — phrasings by task type, plus the follow-up
  ladders that drive step 4. Read when drafting and again when expanding.

Companion skills, invoked by name: `recon` at step 2, `write-prd` at step 7.
