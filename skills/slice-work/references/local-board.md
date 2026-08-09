# Local board export

Read when exporting to a working board on disk. Opt-in, like any export.

The generated plan (`.claude/kanban/<slug>.md`) and the working board
(`.claude/kanban/<slug>/`) are **different artifacts with different lifetimes**.
The plan is generated, reviewed in a diff, and does not change as work proceeds.
The board is mutable state: cards move. Exporting is the point where mutability
begins — after it, the board is the user's, not this skill's.

## Layout

```
.claude/kanban/
  <slug>.md              # the plan. generated, not edited by hand
  <slug>/
    INDEX.md             # generated view of the board. never hand-edited
    K-001.md
    K-002.md
    S-001.md
```

## Ticket file

Frontmatter carries the machine-readable fields; the body carries the prose.

```markdown
---
id: K-002
status: ready           # backlog | ready | doing | review | done
advances: [FR-2]
size: fits              # fits | unverified | indivisible
blocked_by: []          # ticket or spike ids
unstubs: []             # stubs this ticket removes, by ticket id
---

# Reject widgets over the size limit

**Slice:** the size rule only; other validation rules are K-005

**Demo:** POST a widget above the limit → 422 with the field named

**Seams:** HTTP → handler, real. Limit read from real config.

**Rests on:** the 10 MB limit is `[assumed]` in PRD §10 — confirm before pulling
```

Spikes use the same shape with an `S-` id, plus `expires:` and a `question:`
line in the body. A spike delivers a recommendation, not code.

IDs are zero-padded to three digits so files sort in plan order, and are stable
once written. Reordering the board never renumbers a ticket.

## INDEX.md

A generated view — columns, with one line per ticket. Regenerate it from the
ticket files whenever asked; never hand-edit it, and never treat it as the source
of a ticket's status. The frontmatter is authoritative.

```markdown
# Board — <title>

Plan: `../<slug>.md` · Regenerated: <date>

## Ready
- **K-001** POST /widget stores one and returns its id · FR-1, FR-3
- **K-002** Reject widgets over the size limit · FR-2 · rests on an assumption

## Doing
_(nothing)_

## Review
## Done
## Backlog
- **K-003** …
```

## Re-export and drift

Once work starts, the board diverges from the plan. That is expected. Re-export
reconciles rather than overwrites:

| Situation | Action |
| --- | --- |
| Ticket in plan, no file yet | Create it, `status: backlog` |
| File exists, still `backlog` or `ready`, plan text changed | Update the file |
| File exists, status is `doing`/`review`/`done` | **Leave it alone.** Report the difference; do not edit work in flight |
| File exists, ticket gone from the plan | **Never delete.** Report it as dropped from the plan and let the user decide |
| `INDEX.md` | Always regenerate |

The two "never" rows are the point of this table. A re-export that silently
rewrote a card someone is working on, or deleted one, would destroy state the
plan never owned.

Report every skipped and dropped ticket after a re-export. A reconciliation whose
conflicts are visible is useful; one that resolves them quietly is not.

## Before writing anything

1. Say how many files will be created or updated, and where.
2. If the directory already exists, run the reconciliation above and show what
   will be skipped **before** touching it.

This writes to the user's project rather than a shared system, so it is lower
stakes than the GitHub export — but it is still a write, and still opt-in.
