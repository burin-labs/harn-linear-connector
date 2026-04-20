# CLAUDE.md — harn-linear-connector

**Read [SESSION_PROMPT.md](./SESSION_PROMPT.md) first.** It contains the
pivot context, the connector interface contract, the 60-second replay
window detail, and the v0 milestones.

## Quick repo conventions

- File extension: `.harn`. Use `snake_case` for filenames.
- Repo directories use `kebab-case`.
- Entry point: `src/lib.harn`.
- Tests live under `tests/`. Recorded webhook fixtures live under
  `tests/fixtures/webhooks/`.

## How to test

Until `harn add` ships
([harn#345](https://github.com/burin-labs/harn/issues/345)):

```sh
cd /Users/ksinder/projects/harn
cargo run --quiet --bin harn -- run /Users/ksinder/projects/harn-linear-connector/tests/normalize_smoke.harn
cargo run --quiet --bin harn -- check /Users/ksinder/projects/harn-linear-connector/src/lib.harn
cargo run --quiet --bin harn -- lint  /Users/ksinder/projects/harn-linear-connector/src/lib.harn
cargo run --quiet --bin harn -- fmt --check /Users/ksinder/projects/harn-linear-connector/src/lib.harn
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
