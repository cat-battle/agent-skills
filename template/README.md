# Skill template

Copy this directory to `skills/<your-skill-name>/` and edit `SKILL.md`.

## Anatomy of a skill

```
skills/<skill-name>/
  SKILL.md          # required — frontmatter + instructions
  README.md         # optional — notes for humans, not loaded by the agent
  references/       # optional — long material read on demand
  scripts/          # optional — executable helpers the skill can invoke
  assets/           # optional — templates, examples, fixtures
```

## Rules of thumb

- **`SKILL.md` is loaded in full when the skill fires.** Keep it short —
  a page or two. Anything longer belongs in `references/` with a line in
  SKILL.md saying when to read it.
- **The `description` is the only thing the agent sees before loading.**
  It is a routing decision, not a summary. Write it to answer "should I
  open this right now?"
- **Use `name` in kebab-case, matching the directory name.**
- **Write instructions for a model, not a human.** Imperative, concrete,
  ordered. Avoid background and rationale unless it changes behavior.
- **Optional frontmatter:** `allowed-tools: Read, Grep, Bash` restricts the
  skill to a tool subset. Omit it to allow everything.
