# Targeted exploration

Optional, and deliberately narrow. **This is not a codebase survey.** The goal is
to close specific frontier questions cheaply, not to build general understanding
of the repo.

## Protocol

1. **Name the question first.** Only search when a specific frontier question is
   answerable from the repo. If you can't state which one, don't search — that
   urge is curiosity, and curiosity is unbounded.
2. **Apply the pertinence rule.** You are looking for the answer to *that
   question*.

   | Pertinent | Not pertinent |
   | --- | --- |
   | The module named in the request | Unrelated packages |
   | Its direct callers and importers | The whole dependency graph |
   | Config, manifests, lockfiles | Every config in the repo |
   | Tests covering the touched behavior | The full test suite |
   | Entry points for the affected path | Entry points generally |

3. **Respect the budget.** A handful of targeted searches per question. If it
   isn't converging, it is a question for the user, not for grep — hand it back
   to the frontier and ask.
4. **Record it.** Findings become `[code: path/to/file.py:42]` entries for PRD §8,
   kept separate from what the user said so a misreading stays visible and
   correctable rather than laundered into a requirement.

## Which questions are worth recon

Usually answerable from the repo:

- **Prior art** — grep for the feature name, similar modules, past attempts.
- **Constraints** — manifests, lockfiles, linter/formatter config, CI config,
  CLAUDE.md.
- **Technical dependencies** — imports of the module in question, and its callers.
- **Existing verification** — tests already covering the behavior being changed.

Never in the code, always ask:

- Intent, priority, and why now.
- Audience and risk tolerance.
- What is deliberately out of scope.
- Whether current behavior is intended or merely current.

That last one is the trap: the code tells you what happens, never whether anyone
wanted it to. Reading a behavior is not the same as confirming a requirement.
