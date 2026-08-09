# Slicing patterns

Read at step 4, when a requirement resists splitting.

<!-- SCAFFOLD: seeded with the axes that reliably work. Grow this as real
     backlogs turn up a split that was hard to see. -->

## Split axes

Try them in this order. The first two cover most requirements.

| Axis | Split by | First slice |
| --- | --- | --- |
| **Path** | Routes through the flow | The happy path. Each error and edge path is its own slice. |
| **Rule** | Business rules within a behavior | The simplest rule. Variants, exemptions, and overrides follow. |
| **Data** | Input variation | One type, format, or source. More formats are later slices. |
| **Operation** | CRUD verbs | Create, then read. Update and delete are separate slices, not "finishing" the first. |
| **Interface** | Who calls it | One caller — the API, or one client. A second client is a second slice. |
| **Quality** | Works → works well | Correct first. Fast, concurrent, and at-scale are their own slices with their own bars. |

Note what is missing: **by layer**. Model / service / controller / UI is not a
split, it is a schedule for deferring integration.

## Worked example

> **FR-4 (must):** Users can export their report as CSV or PDF, with the export
> emailed to them when it exceeds 30 seconds.

Horizontal, the default and wrong:

1. Add export service interface
2. Implement CSV writer
3. Implement PDF writer
4. Add job queue integration
5. Add email templates
6. Wire up endpoint and test end to end

Nothing is demonstrable until 6. The integration risk — the queue, the email
provider — is discovered last, when it is most expensive.

Vertical, split by data then path then quality:

1. **K-001** Export one report as CSV, synchronously, downloaded in the browser.
   Real endpoint, real writer, real download. *(walking skeleton)*
2. **K-002** Export as PDF, synchronously. Same path, second format.
3. **K-003** Exports over the threshold queue and email a link. Real queue, real
   provider, one hardcoded email body.
4. **K-004** The email body is templated and branded.
5. **K-005** Exports over 30 s in production traffic hold the SLA — the quality bar
   with its own measurement.

The queue and the email provider — the two seams most likely to surprise — are
crossed in the third ticket rather than the last one, and every ticket before
them already ships something a user can do.

## Tells that a ticket went horizontal

- The title is a noun phrase: "Validation layer", "Data model", "API surface".
- Acceptance is "unit tests pass".
- The demo needs a mock, a stub, or a test harness the user would never touch.
- It cannot be described as a change in what someone can do or see.
- A "wire it together" or "end-to-end tests" ticket exists anywhere on the board.
- Nothing is demonstrable until several tickets land together.

The last two are board-level tells and matter most: they mean the whole slicing
pass failed, not one ticket.

## Splits that are not slices

- **Refactor first.** A preparatory refactor with no behavior change is not a
  slice. Fold it into the first ticket that needs it, or make it an explicit
  enabling ticket with a stated reason for being separate.
- **Setup.** "Create the repo", "add CI" — this is the walking skeleton's job.
  It earns a separate ticket only when it delivers nothing traversable, which
  should make you re-examine the skeleton.
- **Spikes.** Knowledge, not deliverable. Time-boxed, prefixed `S-`.

## When a requirement genuinely will not split

Atomic cutovers, format migrations, protocol handshakes. Some behavior has no
smaller demonstrable unit.

Do not manufacture a fake slice. Give the smallest honest ticket, mark it
`indivisible`, and record the reason on the ticket. Then look for the two things
that usually *are* still separable around it: the **verification** (a check that
the migration is correct, shippable ahead of the migration) and the **reversal**
(the rollback path). Both are demonstrable on their own, and both are the parts
that matter when the big ticket goes wrong.
