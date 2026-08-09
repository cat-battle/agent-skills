# Question bank

Reusable phrasings, grouped by task type. Pull from here when drafting the
question set; adapt the wording to the user's own vocabulary.

<!-- SCAFFOLD: seeded with a few examples per type. Grow this file as real
     interviews reveal questions that earned their keep. -->

## Feature

- Who is the user of this feature, and what are they doing right before they hit it?
- What's the smallest version that would still be worth shipping?
- Where should this live — new module, or extend an existing one?

## Bug

- What did you expect, and what happened instead?
- Do you have a reproduction, or should I find one first?
- Is a workaround acceptable, or do you want the root cause fixed?

## Refactor

- What problem is the current shape causing? (If none, why now?)
- Is behavior allowed to change at all, or is this strictly structural?
- What's the blast radius you're comfortable with — one file, one package, repo-wide?

## Migration

- What has to keep working during the transition?
- Big-bang or incremental with both paths live?
- Who else's code breaks if I change this interface?

## Research

- What decision will this research inform?
- What would change your mind?
- Do you want a recommendation, or the options laid out neutrally?

---

# Follow-up ladders

These drive step 4 of SKILL.md. Each trigger is a shape of answer that has left
something unexpanded; the rungs are what to push onto the frontier. Climb until
the closure test passes — the next rung would no longer change what gets built,
who it affects, or how it's verified.

## Trigger: the answer names a system you can't describe
("it pulls from the billing service", "it goes through the queue")

1. Does it exist today, or is it also being built?
2. What's the interface — call, event, table, file?
3. Who owns it, and can it change for us?
4. What happens to this task if it's down, slow, or unavailable?

## Trigger: the answer names an actor
("ops runs it", "the customer uploads it")

1. How do they trigger it, and from where?
2. What can they do wrong, and what should happen when they do?
3. Do they need to be told the outcome, and how?

## Trigger: the answer names data
("it reads the config", "we store the results")

1. What shape, and where does it come from?
2. Does it exist already, or must it be created or migrated?
3. What's the behavior when it's missing, empty, or malformed?
4. Is any of it sensitive — does that constrain logging or storage?

## Trigger: the answer names a state or mode
("only when it's in draft", "unless the flag is on")

1. What are the other states, and does the behavior differ in each?
2. Who sets it, and when does it change?
3. What happens on the transition, not just in the state?

## Trigger: the answer is a comparative
("faster", "cleaner", "more reliable")

1. Than what, measured how?
2. What's the number or observation that means we're done?

## Trigger: the answer defers to something else
("same as the existing one", "however X does it")

1. Which one specifically — can I read it?
2. Is that the pattern you want kept, or just the current state?
3. If they diverge later, which is the source of truth?

## Trigger: the answer excludes something
("not worrying about auth for now", "ignore the legacy path")

1. Out of scope permanently, or just not this session?
2. Should the code leave room for it, or is that premature?
3. Does anything break if it stays unhandled?
