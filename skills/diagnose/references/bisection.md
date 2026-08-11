# Bisection

Read at step 3, to pick the axis to halve. The goal of every experiment is to
**cut the remaining space roughly in half**, not to check the most suspicious
thing first. Suspicion is a poor sort key; it is why debugging sessions wander.

## Pick the axis

| The symptom | Halve along | First move |
| --- | --- | --- |
| Used to work, now doesn't | History | Find a known-good revision, `git bisect` |
| Never worked | The system's layers | Find the boundary where the value goes bad |
| Works here, fails there | The environment | Diff the two environments, not the code |
| Works for some inputs | The input | Minimize a failing input |
| Works alone, fails together | Ordering and shared state | Bisect the set of neighbors |
| Works, then stops after a while | Time and accumulation | Watch a resource over the run |

More than one can apply. Take the cheapest first — an environment diff is minutes,
a history bisect can be an hour of builds.

## History bisect

`git bisect start <bad> <good>`, then `git bisect run <cmd>` where the command
exits 0 for good and non-zero for bad. Automate it even for ten revisions; a
manual bisect is where mislabeled steps happen.

Conditions for the result to mean anything:

- **The tree is clean.** Uncommitted changes ride along at every step and turn the
  bisect into noise.
- **The build is redone at each step.** Otherwise you are testing one artifact
  against many source trees. This is the single most common way a bisect lies.
- **Dependencies move with the code.** Restore the lockfile per revision; a
  dependency resolved fresh at each step means you are bisecting the registry.
- **The check is the symptom**, not a proxy for it, and exits distinctly. Reserve
  exit code 125 for "cannot test this revision" so bisect skips rather than
  misreads it.

The first bad commit is where the symptom *became visible*. A commit that merely
exposed a latent defect is a real and common result — when the culprit looks
innocent, that is usually what happened, so keep going into why it exposed it.

Without usable history — squashed, shallow, or the symptom predates the repo —
fall back to layer bisect.

## Layer bisect

For a never-worked bug: the input is fine at one end and wrong at the other.
Find the boundary where it changes.

1. List the boundaries the value crosses, in order: parse, validate, transform,
   serialize, transmit, deserialize, store, render.
2. Observe the value at the **midpoint** boundary. Right or wrong?
3. Discard the half that is behaving. Repeat.

Observe *at* boundaries, not inside functions — a boundary has a defined correct
value, so the answer is unambiguous. Check the type and encoding, not only the
value: a string `"0"`, a number `0`, and a null all print about the same and
behave very differently.

## Environment bisect

"Works locally, fails in CI" is not a code question until proven otherwise.
Diff the environments before reading any source: runtime and dependency versions,
env vars, config file resolution order, working directory, filesystem case
sensitivity, locale and timezone, available memory and CPU count, network egress,
clock, user and permissions, and whether the container image is the same.

Then bisect the diff: move half the differences from the working environment to
the broken one, or the reverse. Reproducing the failure locally by adopting one
environment property *is* the diagnosis for this class.

## Input bisect

Halve the failing input, keeping it well-formed, until removing anything makes the
symptom disappear. What remains is the minimal reproducer, and it is usually
small enough to point straight at the mechanism.

Watch for the input that is bad in a way you cannot see: trailing whitespace, a
BOM, CRLF, non-ASCII that renders identically, a surprising length that crosses a
buffer or column limit. When the minimized input looks fine, compare bytes rather
than characters.

## One variable at a time

The rule that makes all of the above work. If an experiment changes the code *and*
restarts the process *and* reseeds the store, its result attaches to nothing.

Between two runs of an experiment, state what differed. If you cannot state it in
one clause, the experiment is not an experiment.

## Intermittent failures

Do not treat a flaky symptom as unreproducible. Treat the **rate** as the
measurement.

1. Establish a baseline: N runs, F failures. Record both. Twenty runs is a
   reasonable floor for something that fails "sometimes".
2. Run every experiment the same number of times. A change that takes 8/20 to
   0/20 is evidence; 8/20 to 0/1 is nothing.
3. Push the rate up before hunting: raise concurrency, add load, slow a
   dependency, shrink a timeout, randomize ordering, run on a busier machine. A
   bug that fails 90% of the time is a bug you can bisect.
4. Beware the timing instrument. Logging adds delay and often makes a race
   disappear — see `observation.md`. If the symptom vanishes the moment you
   instrument it, that is itself a strong finding: it points at timing.

For order-dependent failures in a suite, bisect the set of tests that run before
the failing one, not the failing test itself.

## Signals the bisection is going wrong

- Two things changed between runs and you cannot say which mattered.
- The space stopped halving; you are now checking one idea at a time.
- You are re-testing something already ruled out, because you no longer trust the
  earlier result. Trust it or redo it deliberately — do not drift.
- Every revision looks bad, including ones known to be good. The harness is
  wrong, not the code.
- The reproduction stopped working partway through, and you kept going.

Any of these: stop, restate what is known, and report **narrowed** with the
hypotheses actually killed.
