# Interview dimensions

Probe each one. Skip any the repo already answers, or that clearly doesn't
apply. Ordered roughly by how expensive it is to get wrong.

Each dimension seeds at least one frontier question and feeds a named PRD
section. A dimension is not "done" when it has been asked — it is done when its
answer is Answered, Assumed, or Deferred, has been expanded per SKILL.md step 4,
and its PRD section could be written.

## The scoreboard

Closure test condition 3 checks these. **Required** sections must be writable or
marked *N/A* with a reason.

| § | Section | |
| --- | --- | --- |
| 1 | Problem | required |
| 2 | Goal + non-goals | required |
| 3 | Users and scenarios | required |
| 4 | Requirements (functional / non-functional) | required |
| 5 | Acceptance criteria | required |
| 6 | Dependencies | required |
| 7 | Constraints | |
| 8 | Findings from the code | |
| 9 | Risks and open questions | required |
| 10 | Assumptions | required |
| 11 | Rollout and phasing | |

> **Derived.** The source of truth is `prd-template.md` in the `write-prd` skill,
> which owns what each section must contain. This table exists so the interview
> can run its closure test without loading that skill. If they disagree,
> `write-prd` wins — and fix this table.

<!-- SCAFFOLD: each dimension below is a heading + the questions it should
     generate. Fill in as the skill gets used and gaps show up. -->

## 1. Problem → §1
What is wrong today, for whom, and what does it cost them? Why now?
Ask this before any solution talk — the answer is worthless once anchored.

## 2. Outcome → §2
What does the world look like when this is done? Who notices?

## 3. Scope boundaries → §2 non-goals, §4 "won't"
What is explicitly NOT part of this? What existing code must stay untouched?
What's the plausible adjacent thing you are deliberately not doing?

## 4. Users and scenarios → §3
Who touches this, and in what concrete situation? What are they doing right
before? What does "it worked" look like from where they sit?

## 5. Behavior → §4 functional
What must it do — inputs, outputs, states, and what happens on the unhappy
paths. This is the dimension the follow-up ladders expand hardest.

## 6. Qualities → §4 non-functional
Speed, scale, security, accessibility, compatibility, operability — only where
there's a real bar. Push back on invented numbers.

## 7. Success criteria → §5
How is it verified — a test, a manual check, a screenshot, a metric?

## 8. Dependencies → §6
Technical, data, external, and sequencing. See the Dependencies section of
SKILL.md — this dimension is worked as its own pass, not a single question.

## 9. Constraints → §7
Language/framework/version pins, style conventions, dependencies that may
not be added, performance or compatibility requirements.

## 10. Prior art → §8
Has this been attempted? Is there a similar thing in the repo to imitate?

## 11. Audience → §7, §11
Is this throwaway, a prototype, or production code others will maintain?

## 12. Risk and reversibility → §9
Does this touch data, auth, deploys, or anything hard to undo? How would we
know it went wrong, and how do we back it out?

## 13. Phasing and definition of done → §11
Does this land in one piece or in slices? What ships first, behind what flag?
And for *this session*: ship it, leave a PR, or just the PRD?

---

Dimensions 8, 9, and 10 are the ones usually worth a `recon` invocation before
asking the user. Dimensions 1, 2, 3, 11, and 12 are never in the code — intent,
priority, audience, and risk tolerance only come from the user.
