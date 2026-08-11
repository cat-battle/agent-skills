# Observation

Read at step 5, when the failure is not visible from outside the system.

## Cheapest evidence first

Work down this list; stop as soon as the value is visible. Each step costs more
and perturbs more than the one above it.

1. **What the system already emits.** Logs, existing traces, metrics, the error
   output in full — including the parts already scrolled past. Read the whole
   trace before adding anything.
2. **Turn up what it emits.** A verbosity flag, a debug log level, a framework's
   own tracing, `--verbose`, SQL echo. Costs nothing and is already wired to the
   interesting places.
3. **Post-mortem artifacts.** Core dump, heap snapshot, CI artifact bundle, the
   failing job's environment dump, the stored request payload.
4. **Temporary instruments.** Log lines at boundaries you chose. See hygiene
   below.
5. **A debugger.** Best when you need state at one point and can pause. Worst for
   anything timing-dependent.
6. **System-level capture.** `strace`/`dtruss`, packet capture, filesystem
   watches. For when the program's own account of itself is not to be trusted —
   which file did it actually open, which host did it actually reach.

## Read the trace properly

A stack trace names the crash site, which is rarely the cause. Three things in it
are worth more than the top frame:

- **The deepest frame in the project's own code.** Library frames above it are
  usually reporting the bad input they were handed.
- **The exception's cause chain** — the `caused by` / `__cause__` / `.cause`
  links. The original failure is often several wraps down, and re-raising code
  frequently discards the useful message.
- **What is missing.** A frame you expected and do not see means that path never
  ran, which reframes the question from "why did it do the wrong thing" to "why
  did it not get there".

Async, threaded, and callback code produce traces that stop at the scheduler
boundary. The frames you want are on the other side; capture the context at
submission, not at failure.

## Instrument at boundaries

Put the probe where the value has a defined correct shape: on entry and exit of
the suspect region, on both sides of a serialization step, before and after a
cache, at the point of a decision.

Record more than the value:

- The **type** and, when it matters, the length or byte encoding.
- **Provenance** — which branch produced it, which config key it came from.
- **Identity** — request or correlation ID, so lines from concurrent work can be
  separated. Interleaved logs from parallel requests read exactly like a single
  impossible sequence.

For a decision that went the wrong way, log the inputs to the condition, not the
outcome. "Took the else branch" restates the symptom; the operand values explain
it.

## The observer effect

Instruments change the system. Assume it, and check for it.

| Instrument | What it changes |
| --- | --- |
| Log lines in a hot path | Timing — races and deadlocks stop reproducing |
| Any I/O inside a lock | Lock duration, and therefore contention order |
| A debugger breakpoint | Scheduling wholesale; other threads run on |
| Buffered stdout | Apparent ordering, and output lost entirely on a crash |
| Serializing a lazy value to log it | Forces evaluation early, sometimes fixing it |
| Logging a mutable object | Prints its state at flush time, not at call time |
| Extra verbosity | Disk, rate limits, and occasionally a different code path |

Two rules follow. Copy or format a value at the moment of observation rather than
handing the live object to the logger. And when a symptom disappears as soon as
you instrument it, do not conclude it is fixed — record it as evidence of a timing
or ordering mechanism, then reach for a lower-perturbation instrument.

## When you cannot run it live

CI-only and production-only failures still allow evidence: the full job log at
maximum verbosity, artifacts and core dumps, the exact image digest and dependency
lockfile, the environment dump, and the input that triggered it. Pull those before
theorizing.

Prefer reproducing the *environment* over reproducing the *incident* — run the CI
container locally with the CI env vars. That converts an unobservable failure into
an observable one and is almost always faster than reasoning from logs.

Never add an instrument to production as a first move; it is a change to a live
system and needs the user's decision, not yours.

## Do not trust

- **Timestamps across hosts.** Clocks differ; ordering across machines needs a
  correlation ID, not a comparison of times.
- **Sampled traces.** The absent span may have been dropped, not skipped.
- **A log line's position** in a buffered stream, especially near a crash.
- **The error message's text** as a guide to the version running. Stale artifacts
  print old messages — if the message does not exist in the source you are
  reading, you are reading the wrong source. Confirm the artifact before anything
  else.
- **"Nothing in the logs."** Confirm the logger is configured, at the right level,
  and writing where you are looking. Silence is usually a broken instrument.

## Instrument hygiene

The tree ends as it started.

- Keep a running list of every file touched as an instrument, from the first one.
- Mark each addition distinctly — a fixed token like `DIAGNOSE-TEMP` — so nothing
  is found by memory at the end.
- Prefer instruments that cannot be left behind: a debugger, an env-var-gated
  existing log level, a scratch script outside the tree.
- Before reporting, `git diff` the tree and confirm it is empty of instruments.
  Grep the marker too — a diff is easy to skim past.
- State in the report what was installed and that it was removed.

If an instrument is genuinely worth keeping — a log line that should have existed
all along — say so in the report and leave it out of the tree anyway. It is a
change, and changes belong to a ticket.
