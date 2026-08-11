# Failure signatures

Read at step 4 to **aim** the hypotheses. Every entry here is a lead, never a
conclusion — a signature narrows where to look, and the toggle still has to prove
it. Reporting a mechanism from this table without testing it is the exact failure
the skill is built to prevent.

<!-- SCAFFOLD: seeded with the common cases. Grow this as real diagnoses turn up
     a signature that saved an experiment. -->

## By symptom shape

| Symptom | Mechanism families to test first |
| --- | --- |
| Works locally, fails in CI | Version drift; missing env var; config resolution order; working directory; filesystem case sensitivity; test parallelism; locale or timezone; no network egress; clean-checkout state you had locally |
| Fails only on the second call | State leaked between calls; a cache populated on the first; a reused connection or cursor; non-idempotent setup; a consumed one-shot value — iterator, stream, token |
| Fails only in production | Data shape absent from fixtures; scale; real concurrency; different config or secrets; permissions; a network boundary that does not exist locally |
| Intermittent, no pattern | Race on shared state; ordering assumption; timeout near the real latency; resource exhaustion — pool, fd, memory; an external dependency's own flakiness; unseeded randomness |
| Works, then degrades over time | Leak — memory, connections, file handles; unbounded cache or queue; expiring credential or session; clock drift; a growing table hitting a scan |
| Off by hours, or wrong day at the boundary | Timezone applied twice or not at all; DST; naive vs aware datetime; serialization losing the offset; the server's clock |
| Only some records or users | Data-dependent: nulls, encoding, unusual length, a field absent on older rows; permissions differing by role; a partitioned or sharded store |
| Passes alone, fails in the suite | Shared fixture mutated; global or module-level state; ordering dependence; a leaked patch or monkeypatch; parallel workers sharing a resource |
| Started failing with no code change | A floating dependency version; an upstream API change; an expired certificate, token, or credential; data crossing a threshold; a rotated key; disk filling |
| Error text does not match the source | Stale artifact — not rebuilt; wrong version deployed; a shadowed module or duplicate install; the wrong process is answering. **Confirm this before anything else** |
| Change has no effect at all | Editing a file that is not the one loaded; a build not rerun; a cached bundle or bytecode; a running process not restarted; the request reaching a different instance |
| Silent wrong result, no error | A swallowed exception; a default substituted on failure; a type coercion; truncation; a filter matching nothing and returning empty rather than raising |
| Hangs | Deadlock; a wait with no timeout; an unbounded retry; a full queue with no consumer; a connection pool exhausted; DNS |

## By system type

**HTTP service.** Match the layer to the status: 4xx usually means the request
never reached your handler — routing, middleware, auth, body parsing, CORS,
proxy. 5xx means it did. A response that differs between direct calls and calls
through the proxy points at the proxy: buffering, header rewriting, timeouts,
size limits.

**CLI.** Argument parsing and precedence; the working directory; whether the
installed entry point resolves to the source you are editing; stdin/stdout
buffering when piped; exit codes that stay 0 on partial failure; a TTY-dependent
code path that changes under redirection.

**Queue consumer.** Redelivery and at-least-once semantics; poison messages;
visibility timeout shorter than the handler; ordering assumed but not guaranteed;
a dead-letter queue quietly absorbing the evidence; schema drift between producer
and consumer.

**Browser / UI.** State surviving across navigations; a stale render from a cached
bundle; a race between fetch and mount; the event handler bound to the wrong
element; a difference between the dev server and the built artifact — this last
one accounts for a surprising share of "works in dev".

**Data pipeline.** Partial or duplicated runs; schema evolution; nulls introduced
by a join; timezone at the partition boundary; a step reading yesterday's output;
silent type coercion during load.

**Native / systems.** Uninitialized memory; lifetime and use-after-free; alignment
and endianness; buffer sizes; signal handling; a wrong shared library resolved at
load time.

## Priors worth applying

- **Recent change first.** Most breakage is recent. `git log` over the touched
  paths since the last known-good point costs one command.
- **Suspect your own code before the dependency**, and the dependency before the
  language or kernel. Invert only with evidence — but do invert it when the
  version moved, since dependency drift is the top cause of "started failing with
  no code change".
- **The boring cause is the common one.** Stale build, wrong environment, wrong
  process, typo in a config key. Rule those out early precisely because they feel
  too dull to check.
- **Two symptoms appearing at once usually share one cause.** Look for the thing
  upstream of both before diagnosing them separately.
- **"It can't be that" is a hypothesis.** It has a killing observation, so it is
  cheap to test — and it is disproportionately often where the bug is, because the
  assumption is why nobody looked.
