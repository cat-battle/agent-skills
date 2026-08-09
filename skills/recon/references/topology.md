# Codebase topology

Read at step 1. What you are searching determines how you search it, and — more
importantly — determines whether the answer is here at all.

The default assumption, "the code is in this checkout," is wrong often enough
that orienting first is cheaper than discovering it mid-search.

## Detecting the shape

Cheap checks, in order. Stop when the shape is clear.

| Look for | Indicates |
| --- | --- |
| `pnpm-workspace.yaml`, `workspaces` in `package.json`, `go.work`, Cargo `[workspace]`, `pyproject.toml` with multiple packages, `nx.json`, `turbo.json`, `BUILD`/`WORKSPACE` | **Monorepo** |
| `.gitmodules` | **Submodules** — separate repos, possibly uninitialized |
| `vendor/`, `third_party/`, checked-in `node_modules` | **Vendored** dependencies |
| Generated markers: `*_pb2.py`, `*.g.dart`, `codegen/`, "DO NOT EDIT" headers | **Generated** code |
| Dependency manifest naming internal packages you can't find locally | **Polyrepo** — the rest lives elsewhere |
| `openapi.yaml`, `*.proto`, `schema.graphql`, an SDK/client package | **Service boundary** |
| `git sparse-checkout list` returns paths | **Sparse checkout** — files exist upstream but not on disk |

Two commands worth running when unsure: `git config --get remote.origin.url`
(which repo is this?) and a look at the dependency manifest (what does it claim
to depend on that isn't here?).

## Scoping rules by shape

### Monorepo

The unit of pertinence is **the owning package**, not the repo. A search across
every package is the survey that recon exists to prevent.

1. Locate the package that owns the symbol in question.
2. Search inside it first.
3. Widen only along **declared dependency edges** — packages that depend on it,
   or that it depends on. The manifest tells you which; guessing does not.
4. Shared/common/util packages are the exception worth widening into early,
   since behavior often lives there rather than in the caller.

Report the package in findings, not just the path — `packages/billing/src/x.ts:42`
means something different in a monorepo than a bare path does.

### Polyrepo

The answer may be **architecturally absent** — it exists, in a repo you don't
have. This is not the same as "not found," and conflating them is the failure
mode this section exists to prevent.

When the trail leaves the checkout:

1. Stop searching. Nothing here will settle it.
2. Establish what the local code *believes* about the other side — the client,
   the interface, the config pointing at it. That belief is a finding, marked
   `inferred`, because a consumer's model of a service is not the service.
3. Report `external`, naming the repo or service if determinable, and say what
   would settle it: access to that repo, its API docs, or the user.

### Submodules

Check initialization before concluding anything: an uninitialized submodule is an
empty directory, which searches exactly like an absence. `git submodule status` —
a leading `-` means not initialized. If uninitialized, report `external`, not
`absent`. Never initialize one yourself; that is a write.

### Vendored and installed dependencies

Reading a dependency's source to answer "what does this library actually do" is
legitimate and often the fastest path. Two rules:

- **Mark it as third-party** in the finding, with the version. Library behavior
  is a fact about a pinned version, not about the project.
- **Never let it become a requirement.** How a dependency behaves is a constraint
  to design around, not something the project decided.

### Generated code

Read the **generator input** — schema, proto, template, spec — not the output.
The output answers "what does it do today"; the input answers "what is it
supposed to do", and only the input is editable. If a question can only be
answered from generated output, say so, since that usually indicates the
generator or its input is the real subject.

### Sparse checkout

Paths outside the sparse cone are absent from disk but present upstream. Confirm
with `git sparse-checkout list` before reporting `absent`, and report `external`
instead if the path is merely not materialized.

## Service boundaries

When a question crosses from one service to another, **the contract artifact is
authoritative** — the OpenAPI spec, the `.proto`, the GraphQL schema, the
published types.

The consumer's client code is not authoritative. It tells you what one caller
believes the other side does, which may be stale, partial, or wrong. Findings
read from a client are `inferred` about the service and `confirmed` only about
the client.

If the contract artifact is in the repo, prefer it over both.

## When the tree isn't enough

Escalate to the user rather than guessing, and say precisely what you need:

- "Which repo owns X?" — when a dependency edge leaves the checkout.
- "Is there a spec for this interface?" — when only client code is available.
- "Should I treat the vendored copy as current?" — when vendored and declared
  versions disagree.

These questions are cheap for the user and unanswerable by search. Asking one is
a better outcome than a confident finding built on the wrong tree.

## Feeding dependencies downstream

An `external` finding is usually also a **dependency** — another team's repo,
service, or schema that this work relies on. Callers producing a PRD should carry
it into §6 Dependencies, not just §8 Findings, since it needs an owner and a
fallback rather than only a citation.
