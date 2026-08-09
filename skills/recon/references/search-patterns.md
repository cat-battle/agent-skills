# Search patterns

Tactics for step 4, when the entry point isn't obvious from the question.
Assumes step 1 already established the tree's shape — see `topology.md`. In a
multi-package tree, apply everything below **within the owning package** before
widening.

<!-- SCAFFOLD: seeded with the common cases. Grow this as real recons turn up
     patterns that saved a search. -->

## Order of attack

1. **Exact name** from the question — symbol, route, env var, error string.
   Error strings and log messages are the highest-yield search terms in an
   unfamiliar repo: they are unique, and they sit exactly where the behavior is.
2. **Definition, then uses.** Find where it's defined, then who calls it. The
   call sites usually answer "when does this happen?" faster than the body does.
3. **Tests.** A test named for the behavior is a specification of it, often
   clearer than the implementation.
4. **Config and manifests** — for questions about versions, flags, limits, and
   whether a capability is even enabled.
5. **Git history** — only when the question is "why is it like this?"
   `git log -S<string>` finds the commit that introduced a line;
   `git log -p <file>` reads its evolution.

## Entry points by project type

| Type | Start at |
| --- | --- |
| CLI | argument parser setup, `main`, console-script entries in the manifest |
| HTTP service | route table or router registration, then middleware chain |
| Library | the public export surface — `__init__.py`, `index.ts`, `lib.rs` |
| Frontend | router config, then the component named in the question |
| Data pipeline | task/DAG definitions, then the transform in question |
| Anything unfamiliar | the manifest's scripts/entry-points, then CI config |

CI config is underrated: it names the real build, test, and lint commands, which
tells you what the project actually considers correct.

## Signals a search is going wrong

- You've opened files you can't connect back to the question.
- You're reading whole files rather than specific functions.
- The result set keeps growing instead of narrowing.
- You've started reconstructing the architecture rather than answering.
- You've drifted into packages with no declared edge to the one you started in.
- Every trail ends at an import, a client, or a URL that leaves the tree.

The last one is not a failed search — it is the answer. Report `external` and
stop; no amount of local searching finds code that isn't checked out.

Any of the others means: stop, restate the question, and either re-aim at a
narrower target or escalate.

## Absence is a claim

To report `absent` honestly, the search has to have been capable of finding the
thing. Three conditions:

1. **You searched the right tree.** In a monorepo, the owning package and its
   declared dependents. If the code could live in a repo you don't have, the
   finding is `external`, not `absent`.
2. **You tried at least two vocabularies** — the domain word and the likely code
   word (`retry`/`backoff`, `auth`/`session`/`token`, `cache`/`memo`).
3. **Nothing hid the files.** Sparse checkout, uninitialized submodules, and
   ignore rules all make present code invisible to a search. Check before
   claiming absence.

Then name where you looked in the finding, so the caller can judge the search
rather than take it on faith.
