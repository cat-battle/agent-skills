# agent-skills

Various agent-skills that I have developed for my own needs.

## Layout

```
skills/
  _template/           # copy this to start a new skill
  task-interview/      # interview the user to closure, then hand off
  write-prd/           # gathered material -> implementation-ready PRD
```

Each skill is a directory containing a `SKILL.md` with YAML frontmatter
(`name`, `description`) followed by instructions for the agent. See
[`skills/_template/README.md`](skills/_template/README.md) for the full
anatomy and conventions.

## How skills compose

Skills invoke each other **by name**, via the Skill tool — never by relative
path. `task-interview` calls `write-prd` once its question frontier closes.

A name lookup works regardless of how either skill was installed; a path like
`../write-prd/references/prd-template.md` breaks as soon as one is symlinked
without the other. So: **each skill directory is self-contained on disk, and
composition happens at runtime.**

Where two skills genuinely need the same knowledge, one owns it and the other
keeps a deliberately minimal derived copy, labeled as derived with a pointer to
the source of truth. (Example: the PRD section list, owned by
`write-prd/references/prd-template.md`, mirrored as a bare checklist in
`task-interview/references/dimensions.md` so the interview can run its closure
test without loading the other skill.)

## Installing

Skills are discovered from `~/.claude/skills/` (personal) or
`<repo>/.claude/skills/` (project). Symlink rather than copy, so edits here
take effect immediately:

```bash
ln -s "$PWD/skills/task-interview" ~/.claude/skills/task-interview
ln -s "$PWD/skills/write-prd"      ~/.claude/skills/write-prd
```

Install skills that call each other together — `task-interview` degrades to
"interview, then hand you the material" if `write-prd` isn't present.

Verify with `/help` or by asking Claude to list available skills.

## Adding a skill

```bash
cp -r skills/_template skills/my-skill
```

Then edit `SKILL.md` — the `name` must match the directory, and the
`description` is what the agent reads when deciding whether to load it.
