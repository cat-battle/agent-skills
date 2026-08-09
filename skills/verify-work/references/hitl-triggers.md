# Human-in-the-loop triggers

Read at step 2, before running anything. Classification changes what you run, so
it is worth the minute it costs.

A human check is expensive: it blocks the board and spends someone's attention.
Route what genuinely needs it and nothing else. **Over-routing is the more common
failure** — a queue of checks that were all really agent-runnable trains people to
rubber-stamp, and the one check that mattered gets stamped with the rest.

## The seven classes

| Class | The test | What the human is given |
| --- | --- | --- |
| **Perceptual** | Correctness is "does it look/sound/feel right" — layout, spacing, motion, contrast, audio. | The screenshot or recording, plus the path to reproduce. |
| **Judgment** | A person's reaction is the acceptance criterion — is this copy clear, is this error message useful, is this flow confusing. | The exact strings or the flow, in context, not in isolation. |
| **Intent** | "Is this what you meant?" The code can satisfy the ticket and still miss the ask. | The demo's result next to the original request. |
| **Consequence** | The action is irreversible or leaves the system — production data, real payments, outbound mail, third-party writes, deletion. | A prepared, reviewable action and an explicit authorization request. |
| **Access** | The check needs credentials, MFA, a physical device, a paid tier, or a sandbox the agent cannot reach. | The exact steps and the expected result, so they run it once. |
| **Oracle-less** | No known-correct answer exists to compare against — ranking quality, model output, a heuristic's "good enough". | A sample with the agent's own read, so they are correcting rather than starting cold. |
| **Environment** | Correctness depends on real hardware, real network conditions, real scale, or real data the agent has no copy of. | What to run where, and what number or behavior counts as a pass. |

Perceptual, Judgment, and Intent are the ones that look agent-verifiable and are
not. Consequence is the one where the agent must stop *before* acting, not report
afterward.

## False HITL

These look like human checks and are not. Run them.

| Looks like | Actually |
| --- | --- |
| "It's a UI, so a human has to look." | The **click path** is agent-runnable — drive a real browser and assert the state. Only the aesthetic judgment is human. Split them: run the path, route the look. |
| "I can't start the server / database." | You usually can. Start it, or say concretely what stopped you. "Couldn't run it" without a reason is an unverified check wearing a HITL badge. |
| "It touches the database, so it's risky." | A local or test database is not production. Consequence means *irreversible* or *outward-facing*, not merely stateful. |
| "The output is long, a human should review it." | Length is not judgment. Assert the part the ticket named. |
| "It needs an API key." | Check whether one is configured, or a sandbox exists, first. Access means genuinely unreachable. |
| "A human should confirm the tests are right." | If the demo was written before the code, it is already the human's check. Run it. |

Before routing anything, answer: *what specifically stops me from settling this?*
If the answer is not one of the seven classes, it is not a HITL — it is work.

## Shift the human check left

A ticket whose demo needs a human should say so **when the ticket is written**,
not when it is finished. Discovering it at verification time means the human is
surprised instead of queued, and the card stalls at the end of the slice where
delay is most expensive.

`slice-work` tickets carry a **Verified by** field for this. When verification
turns up a human check the ticket did not predict, report that as a planning miss
alongside the verdict — it is the signal that the next batch of tickets should
mark theirs.

## The request

One batched request per verification pass, not a trickle. A human interrupted
five times will answer the fifth one carelessly.

Ask for a **decision**, not a review. Each item gets four lines:

```
[K-002 · perceptual] Error banner on the oversize-upload path
  Reach it:  npm run dev → /upload → attach fixtures/12mb.png
  Judge:     does the banner read as the file's problem, not a system fault?
  Wrong if:  it reads as a crash, or the size limit isn't stated
  Blocks:    K-002 → done; K-005 is queued behind it
```

Rules:

- **Name what would be wrong**, not just what to look at. "Does this look OK?"
  gets a yes.
- **Reach it in one paste.** If they have to work out how to get there, they will
  approximate, and approximation is how a check gets rubber-stamped.
- **Say what it blocks.** It is what tells them whether to do it now.
- **Never ask a question you could have answered.** Check that first; the credit
  spends fast.
- For the **Consequence** class, the request is an authorization: show the exact
  action, its blast radius, and how to undo it — or state plainly that it cannot
  be undone.

## While the human check is open

The ticket is **not done**. It sits in `review`, and the report says
`verified pending human` with the outstanding items named.

Do not proceed as though the answer will be yes. If later work genuinely cannot
wait, say which tickets are proceeding on an unconfirmed check and what has to be
revisited if the answer comes back no — the same treatment `slice-work` gives an
open question, for the same reason.
