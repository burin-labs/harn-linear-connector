# CLAUDE.md — harn-linear-connector

## Quick repo conventions

- File extension: `.harn`. Use `snake_case` for filenames.
- Repo directories use `kebab-case`.
- Entry point: `src/lib.harn`.
- Tests live under `tests/`. Recorded webhook fixtures live under
  `tests/fixtures/webhooks/`.

## How to test

Install the pinned Harn CLI from crates.io and run the local gate:

```sh
cargo install harn-cli --version "$(cat .harn-version)" --locked
harn check src
harn lint src
harn fmt --check src tests
for test in tests/*.harn; do
  harn run "$test" || exit 1
done
```

## Reference Rust impl

The existing 1165-LOC Rust connector at
`/Users/ksinder/projects/harn/crates/harn-vm/src/connectors/linear/mod.rs`
is the **behavior spec**. Linear's exact header names and signature
encoding live there — read first before re-deriving from Linear's docs.

## Upstream conventions

For general Harn coding conventions and project layout, defer to
[`/Users/ksinder/projects/harn/CLAUDE.md`](/Users/ksinder/projects/harn/CLAUDE.md).

## Don't

- Don't add a typed Linear SDK to this repo. If you want one, propose
  `linear-sdk-harn` (GraphQL codegen) as a separate repo.
- Don't hand-edit `LICENSE-*` or `.gitignore`.
