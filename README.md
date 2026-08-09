# agent-skills

Various agent-skills that I have developed for my own needs.

## Layout

```
skills/
  _template/           # copy this to start a new skill
  recon/               # answer one question from the codebase, read-only
  task-interview/      # interview the user to closure, then hand off
  write-prd/           # gathered material -> implementation-ready PRD
```

Each skill is a directory containing a `SKILL.md` with YAML frontmatter
(`name`, `description`) followed by instructions for the agent. See
[`skills/_template/README.md`](skills/_template/README.md) for the full
anatomy and conventions.

## How skills compose

Skills invoke each other **by name**, via the Skill tool — never by relative
path:

```
task-interview ──step 2──> recon        (per question, read-only)
               ──step 7──> write-prd    (once the frontier closes)
```

A name lookup works regardless of how either skill was installed; a path like
`../write-prd/references/prd-template.md` breaks as soon as one is symlinked
without the other. So: **each skill directory is self-contained on disk, and
composition happens at runtime.**

Callers degrade rather than fail when a companion is missing —
`task-interview` searches directly if `recon` is absent, and hands back raw
material if `write-prd` is. Each caller says so at the point of invocation.

Skills that pass data define a contract at the boundary and state it in their own
SKILL.md, so neither side has to read the other. `recon` emits findings as
`claim + path:line + confidence`; `task-interview` carries them through;
`write-prd` enforces that an `inferred` finding is never written as fact.

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
for s in recon task-interview write-prd; do
  ln -s "$PWD/skills/$s" ~/.claude/skills/$s
done
```

Install skills that call each other together; see *How skills compose* for how
they degrade if you don't.

Verify with `/help` or by asking Claude to list available skills.

## Adding a skill

```bash
cp -r skills/_template skills/my-skill
```

Then edit `SKILL.md` — the `name` must match the directory, and the
`description` is what the agent reads when deciding whether to load it.
