# agent-skills

Various agent-skills that I have developed for my own needs.

## Layout

```
skills/                # everything here installs; nothing else belongs here
  recon/               # answer one question from the codebase, read-only
  task-interview/      # interview the user to closure, then hand off
  write-prd/           # gathered material -> implementation-ready PRD
  slice-work/          # approved PRD -> kanban of vertical slices
  verify-work/         # run a ticket's demo, route what needs a human
template/              # copy this to start a new skill — deliberately outside skills/
```

Each skill is a directory containing a `SKILL.md` with YAML frontmatter
(`name`, `description`) followed by instructions for the agent. See
[`template/README.md`](template/README.md) for the full anatomy and
conventions.

`skills/` contains **only installable skills**. The template lives outside it
because installation symlinks the whole directory — anything sitting in
`skills/` becomes a live skill, and a template's placeholder `description` is
routing noise the model has to read every session.

## How skills compose

Skills invoke each other **by name**, via the Skill tool — never by relative
path:

```
task-interview ──step 2──> recon        (per question, read-only)
               ──step 7──> write-prd    (once the frontier closes)
write-prd      ──on approval──> slice-work   (only when asked to plan work)
slice-work     ──step 4──> recon        (to size a slice against real code)
verify-work    ──after a slice is built──> reads that slice's ticket
```

A name lookup works regardless of how either skill was installed; a path like
`../write-prd/references/prd-template.md` breaks as soon as one is symlinked
without the other. So: **each skill directory is self-contained on disk, and
composition happens at runtime.**

Callers degrade rather than fail when a companion is missing —
`task-interview` searches directly if `recon` is absent, hands back raw
material if `write-prd` is, and `slice-work` marks a slice `unverified` rather
than guessing when `recon` is. Each caller says so at the point of invocation.

`verify-work` composes through an artifact rather than a call: it reads the
**Demo** that `slice-work` wrote on the ticket before the code existed, so the
acceptance check is never derived from the implementation it is supposed to
judge. Without a ticket it writes the demo from stated intent first, and says so.

Skills that pass data define a contract at the boundary and state it in their own
SKILL.md, so neither side has to read the other. `recon` emits findings as
`claim + path:line + confidence`; `task-interview` carries them through;
`write-prd` enforces that an `inferred` finding is never written as fact; and
`slice-work` flags the tickets that rest on one, since those are the tickets
most likely to change shape.

Where two skills genuinely need the same knowledge, one owns it and the other
keeps a deliberately minimal derived copy, labeled as derived with a pointer to
the source of truth. (Example: the PRD section list, owned by
`write-prd/references/prd-template.md`, mirrored as a bare checklist in
`task-interview/references/dimensions.md` so the interview can run its closure
test without loading the other skill. Likewise the human-check taxonomy, owned by
`verify-work/references/hitl-triggers.md` and mirrored as four lines in
`slice-work/references/board-template.md` so a ticket can be marked at planning
time.)

## Installing

One symlink installs everything:

```bash
ln -s "$PWD/skills" ~/.claude/skills
```

Uninstall is the inverse, and removes nothing but the link:

```bash
rm ~/.claude/skills
```

Claude Code discovers every subdirectory of `~/.claude/skills/` as a skill, so
pointing that path at this repo installs the whole set at once and picks up new
ones with no further linking. Edits here take effect in the next session,
because the link resolves to the working tree rather than a copy.

Two consequences worth knowing:

- **Everything in `skills/` goes live.** That is why `template/` sits outside
  it. A scratch or half-finished skill left in `skills/` is an installed skill.
- **`~/.claude/skills/` cannot also hold unrelated skills** while it is a link
  to this repo. If you need to mix sources, symlink individual skills instead:
  `ln -s "$PWD/skills/recon" ~/.claude/skills/recon`.

Per-project installs use `<repo>/.claude/skills/` and work the same way.

Verify with `/skills`, or non-interactively:

```bash
claude -p "List the names of every skill available to you, one per line."
```

Skills that call each other should be installed together; see *How skills
compose* for how they degrade if they aren't. The single symlink makes that
automatic.

## Adding a skill

```bash
cp -r template skills/my-skill
```

Then edit `SKILL.md` — the `name` must match the directory, and the
`description` is what the agent reads when deciding whether to load it. It
installs on the next session with no linking step.
