# Check patterns

Read at step 3, when the real entry point or the real seam isn't obvious.

## Entry points by system type

The real entry point is the one a user or caller would actually use. Reaching in
one layer below it is the most common way a check passes while the system is
broken.

| System | Real entry point | Reaching in too far |
| --- | --- | --- |
| HTTP service | An HTTP request to a running process | Calling the handler function directly |
| CLI | Executing the built binary or `python -m pkg` | Importing `main()` and calling it |
| Web UI | Driving a real browser through the flow | Rendering the component in a test harness |
| Queue consumer | Publishing to the real broker | Invoking the message handler with a literal payload |
| Library | Importing the built/installed package | Importing from the source tree with a patched path |
| Scheduled job | Triggering the scheduler's own invocation path | Calling the job body |
| Migration | Running the migration tool against a real schema | Reading the migration file and reasoning about it |

Where the built artifact matters — a bundle, a container, a wheel — verify
against the built artifact. A check that passes in the source tree and fails once
packaged is a check that tested the wrong thing.

## Real seam vs fake seam

A seam is real when a defect on the far side of it would make the check fail.
That is the whole test — apply it per boundary.

| Real | Fake |
| --- | --- |
| A running process reached over its actual transport | The function, imported and called |
| A database, container or file-backed, with the real schema | An in-memory fake with hand-written behavior |
| Real serialization — encode, transmit, decode | Passing the object through untouched |
| Real config loading from the real source | Constructing the config object inline |
| Real auth, even with a test identity | Auth bypassed by a flag the check sets |
| A recorded fixture captured from the real dependency | A mock whose responses were written from the docs |
| The clock, or an injected clock the production path also uses | Time patched globally in the test only |

Stubs are legitimate — `slice-work` allows a slice to stub a seam. They are not
legitimate **silently**. Name every stub in the report and say which ticket
removes it. A stub the report doesn't mention becomes a permanent one.

For a third party you genuinely cannot call, assert against a contract pinned to
its documented shape and record the risk — not a mock that agrees with itself
forever.

## Negative controls

One per check. Cheap ones, in rough order of preference:

| Check shape | Control |
| --- | --- |
| Feature now works | Stash or revert the change; confirm the check goes red; restore |
| A rule rejects bad input | Feed input that should pass — confirm it isn't rejected too |
| A value is written | Read the store before the run; confirm the value was genuinely absent |
| An endpoint returns data | Request a record that does not exist; confirm it isn't the same response |
| Config is honored | Change the config; confirm the behavior changes with it |
| An error path | Remove the trigger; confirm the error stops appearing |
| Nothing regressed | Introduce the regression deliberately; confirm the suite catches it |

Restore the tree after a destructive control, and say in the report that you did.

## Tells that a check proved nothing

Treat any of these as a failed control until shown otherwise:

- It passes with the implementation deleted, commented out, or reverted.
- It passes against a process started before the change was built.
- It asserts on a field that is absent in both the pass and the fail case —
  `assert "error" not in response` is true of an empty response.
- The assertion is on a value the check itself supplied.
- It passes with the network unplugged, and shouldn't.
- The suite reports `0 tests ran`, all-skipped, or a filter that matched nothing.
- A build step was skipped, so the artifact under test predates the change.
- The only evidence is an exit code, for a command that exits 0 on partial
  failure.

## Environment reality

State which of these was true, because a check's meaning depends on it: the
process was rebuilt after the change; the data store was real and its state
known; config came from where it comes from in production; the check ran from a
clean state or a stated one.

If a check needs a seeded state, seed it explicitly in the check. A check that
passes only against a database someone left in the right condition will fail on
the next machine and no one will know why.
