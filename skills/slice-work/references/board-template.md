# Board template

The target artifact. Read at step 1.

<!-- SCAFFOLD: field set is a starting point. Prune fields that never earn their
     line in a real board; add ones that a real slice kept needing. -->

## Ticket fields

| Field | Rule |
| --- | --- |
| **ID** | `K-001`, `K-002`… Zero-padded so exported files sort in plan order. Stable once written; never renumber to reorder. |
| **Title** | A change in behavior, in the present tense. `Reject widgets over the size limit`, not `Validation layer`. A noun phrase is a horizontal ticket wearing a title. |
| **Slice** | What this ticket deliberately leaves out. The narrowing is the design work — record it. |
| **Advances** | Requirement IDs, `(partial)` where partial. Many-to-many with tickets. |
| **Demo** | The observable check through the real entry point. Concrete enough to run: a command, a request, a click path, a message published. |
| **Seams** | Which boundaries this crosses for real. Stubs named explicitly, each with the ticket that removes it. |
| **Rests on** | Assumptions, `inferred` findings, or open questions this ticket depends on. Omit only when genuinely none. |
| **Size** | `fits` — passes the no-"and" test. `unverified` — depends on code not yet checked. `indivisible` — admitted big, with why. |

Optional, when they carry information: owner, and a risk note for tickets ordered
early *because* they are risky.

## Board file

`.claude/kanban/<slug>.md`

```markdown
# Kanban — <title>

Source: `.claude/prds/<slug>.md` (status: approved)
Generated: <date>

Order is by risk, not dependency. K-001 is the walking skeleton.

## Ready

### K-001 — POST /widget stores one and returns its id
- **Slice:** happy path only; one field; no validation, no auth, no list endpoint
- **Advances:** FR-1, FR-3 (partial)
- **Demo:** `curl -XPOST localhost:8080/widget -d '{"name":"a"}'` → 201 with an
  id; the row is readable in the database afterward
- **Seams:** HTTP → handler → repository → real Postgres. Nothing stubbed. Runs
  in CI against a container.
- **Rests on:** —
- **Size:** fits

### K-002 — Reject widgets over the size limit
- **Slice:** the size rule only; other validation rules are K-005
- **Advances:** FR-2
- **Demo:** POST a widget above the limit → 422 with the field named
- **Seams:** HTTP → handler, real. Limit read from real config.
- **Rests on:** the 10 MB limit is `[assumed]` in PRD §10 — confirm before K-002
  is pulled
- **Size:** fits

## Backlog

### K-003 — …

## In progress

_Empty. This skill does not move cards._

## Review

_Empty._

## Done

_Empty._
```

Spikes carry an `S-` prefix, a question, and an expiry:

```markdown
### S-001 — Can the existing job runner guarantee at-least-once delivery?
- **Question:** …
- **Expires:** 2 days. If unanswered, proceed with K-007 under the assumption it
  cannot, and record the fallback.
- **Blocks:** K-007, K-008
- **Delivers:** a recommendation and a citation, not code
```

## Exports

Two targets, both **opt-in on an explicit request** and never a finishing touch.
The plan above stays the source of truth for the *plan* after either one — it is
reviewable in a diff, and a tracker is not.

- **Local working board** — `.claude/kanban/<slug>/`, one file per ticket. The
  default for actually working the board. See `local-board.md` for the file
  format and the re-export drift rules.
- **GitHub Issues** — below. For when the team lives in the tracker.

## Export to GitHub Issues

It writes to a shared system, so it needs more care than the local export.

Before creating anything:

1. **Confirm the target repo** and show the exact list of tickets to be created.
   Ask once; do not infer the repo from the git remote alone.
2. **Check for existing issues** with the same titles. Creating a second copy of
   a backlog is expensive to undo by hand.

Mapping:

| Board | GitHub |
| --- | --- |
| Title | Issue title, prefixed with the ticket ID |
| Slice, Demo, Seams, Rests on | Issue body, under the same headings |
| Advances | A line of requirement IDs in the body, and a label per requirement if the repo already uses them |
| Column | Project board column, when a project is named; otherwise omit |
| Spike | Label `spike`, with the expiry in the body |

There is no reconciliation story for this target. Re-running it against a repo
that already has the issues creates duplicates, which is why the duplicate check
above is a precondition rather than a nicety.
