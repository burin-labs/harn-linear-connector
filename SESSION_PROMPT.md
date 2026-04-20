# SESSION_PROMPT.md — harn-linear-connector v0

You are picking up the v0 build of `harn-linear-connector`, a pure-Harn
Linear connector. This file is your self-contained bootstrap.

## Pivot context (60 seconds)

Harn is moving per-provider connectors **out** of its Rust monorepo and
into external pure-Harn libraries under `burin-labs/`. This repo is one
of the four flagship per-provider connectors. The existing Rust impl at
`/Users/ksinder/projects/harn/crates/harn-vm/src/connectors/linear/mod.rs`
(1165 LOC) is the behavior spec — port its semantics into pure Harn.

Tracking ticket: [Pure-Harn Connectors Pivot epic
#350](https://github.com/burin-labs/harn/issues/350).

## What this repo specifically delivers

A pure-Harn module that implements the Harn Connector interface and is
loadable as the `linear` provider:

- `pub fn provider_id() -> string` returning `"linear"`.
- `pub fn kinds() -> list` returning `["webhook"]`.
- `pub fn payload_schema() -> dict` returning the canonical normalized
  event schema.
- Lifecycle: `pub fn init(ctx)`, `pub fn activate(bindings)`,
  `pub fn shutdown()`.
- `pub fn normalize_inbound(raw) -> dict` — verifies HMAC against the
  raw body using `hmac_sha256` + `constant_time_eq`, enforces Linear's
  60-second replay window, normalizes the Linear event payload.
- `pub fn call(method, args)` — outbound dispatch. v0 surface is
  effectively `"graphql"` with `{ query, variables }`, plus a couple
  of convenience wrappers around common GraphQL mutations.

## Linear-specific details to honor

- **Header / signature format**: the exact header names (e.g.
  `linear-signature` and `linear-delivery-timestamp`) and encoding
  (hex hash of the raw body) live in
  `/Users/ksinder/projects/harn/crates/harn-vm/src/connectors/linear/mod.rs`.
  Read those constants first; don't re-derive from Linear's public docs
  when a working implementation exists.
- **60-second replay window**: Linear specifies that a webhook is valid
  only within 60 seconds of its delivery timestamp. Reject deliveries
  outside this window. This is *narrower* than Slack's 5-minute window
  — easy to mis-port if you copy from `harn-slack-connector`.
- **Resource events**: Linear emits events keyed by resource type
  (`Issue`, `Comment`, `Project`, `IssueLabel`, `Cycle`,
  `ProjectUpdate`, `Reaction`, `User`, etc.) with an `action` field
  (`create`, `update`, `remove`).

## What's blocked

- **[harn#346 (Connector interface contract)](https://github.com/burin-labs/harn/issues/346)** —
  the formal interface isn't accepted yet. Match the function shapes
  listed above; expect tweaks.
- **[harn#345 (Package management v0)](https://github.com/burin-labs/harn/issues/345)** —
  needed for distribution. **Do not cut a v0.1.0 release tag until
  #345 lands.**
- **[harn#347 (Bytes value type + raw inbound body access)](https://github.com/burin-labs/harn/issues/347)** —
  Linear sends UTF-8 JSON, so `raw.body_text` works for HMAC
  verification in v0. Revisit when #347 lands.

## What's unblocked

- `hmac_sha256(key, message) -> hex_string` — Harn builtin, just shipped.
- `constant_time_eq(a, b) -> bool` — **use this for signature
  comparison**.
- HTTP, JSON, dict, list, regex, datetime stdlib.

## v0 milestones (build in order)

### M1 — Connector interface skeleton

- Stub all interface functions in `src/lib.harn`.
- `provider_id()` returns `"linear"`. `kinds()` returns `["webhook"]`.
  `payload_schema()` returns the canonical schema dict.
- `init`, `activate`, `shutdown` manage module state in a top-level dict.
- Acceptance: `harn check src/lib.harn` exits 0; smoke test imports the
  module and calls each interface function without error.

### M2 — HMAC verification + replay-window enforcement

- Port HMAC verification from
  `/Users/ksinder/projects/harn/crates/harn-vm/src/connectors/linear/mod.rs`.
  Use `hmac_sha256(secret, raw.body_text)` and `constant_time_eq`
  against the signature header value.
- Enforce the 60-second replay window using
  `linear-delivery-timestamp` (or whatever the Rust impl reads).
  Reject if `abs(now - delivery_ts) > 60s`.
- Reject (`Err`) on missing signature, missing timestamp, mismatched
  signature, expired timestamp.
- Acceptance: `tests/sig_smoke.harn` covers:
  - Valid signature + fresh timestamp → Ok.
  - Tampered body → Err.
  - Valid signature but timestamp 70s in the past → Err.
  - Missing signature header → Err.

### M3 — Event normalization

- Normalize the verified payload per resource. v0 covers `Issue`,
  `Comment`, `Project`. Each produces:
  ```harn
  {
    event_type: "Issue.create" | "Issue.update" | "Issue.remove" | "Comment.create" | ...,
    resource_id: "...",
    actor: { id, name, email? },
    occurred_at: "...",
    organization_id: "...",
    raw: <original payload>,
    // resource-specific normalized fields
  }
  ```
- Acceptance: `tests/normalize_events.harn` exercises 3+ recorded
  fixtures across the supported resources.

### M4 — Outbound GraphQL dispatch

- `call("graphql", { query, variables })` posts to
  `https://api.linear.app/graphql` with `Authorization: <api_key>`
  (note: Linear personal API keys are passed *without* a `Bearer`
  prefix; OAuth tokens *are* prefixed — port the exact behavior from
  the Rust impl).
- Surface GraphQL `errors[]` as `Err` with a structured shape
  (`{ message, path, extensions }` per error).
- Acceptance: `tests/graphql_smoke.harn` exercises a mocked happy-path
  query and a mocked errors-array response, asserting both shapes.

## Recommended workflow

1. **Use a worktree per milestone:**
   ```sh
   cd /Users/ksinder/projects/harn-linear-connector
   git worktree add ../harn-linear-connector-wt-m1 -b m1-skeleton
   ```
2. **Read the Rust impl first** for header names, replay window
   constants, and auth token format. These are the most error-prone
   bits of the port.
3. **Test the replay-window edges.** ±60s, ±59s, ±61s, future
   timestamps, invalid timestamp formats — all should have explicit
   tests.
4. **Pin webhook fixtures from a sandbox workspace**, never from a
   production workspace.

## Reference materials

- Harn quickref: `/Users/ksinder/projects/harn/docs/llm/harn-quickref.md`.
- Harn language spec: `/Users/ksinder/projects/harn/spec/HARN_SPEC.md`.
- Existing Rust impl (the spec for behavior):
  `/Users/ksinder/projects/harn/crates/harn-vm/src/connectors/linear/mod.rs`.
- HMAC builtins conformance fixture:
  `/Users/ksinder/projects/harn/conformance/tests/stdlib/hmac_sha256.harn`.
- Linear webhook docs:
  <https://developers.linear.app/docs/graphql/webhooks>.
- Linear GraphQL API:
  <https://developers.linear.app/docs/graphql/working-with-the-graphql-api>.

## Testing expectations

- Negative-path tests for HMAC verification are mandatory.
- Replay-window edge tests are mandatory (the 60s constant is the
  most likely thing to drift in a port).
- Use `constant_time_eq` for *all* signature comparisons.
- Mock all live HTTP for v0 tests.
- Run before committing:
  ```sh
  cd /Users/ksinder/projects/harn
  cargo run --quiet --bin harn -- check /Users/ksinder/projects/harn-linear-connector/src/lib.harn
  cargo run --quiet --bin harn -- lint  /Users/ksinder/projects/harn-linear-connector/src/lib.harn
  cargo run --quiet --bin harn -- fmt --check /Users/ksinder/projects/harn-linear-connector/src/lib.harn
  for t in /Users/ksinder/projects/harn-linear-connector/tests/*.harn; do
    cargo run --quiet --bin harn -- run "$t" || exit 1
  done
  ```

## Definition of done for v0

- [ ] All interface functions implemented and `harn check` clean.
- [ ] HMAC verification uses `constant_time_eq`, with negative-path
      tests proving rejection of tampered payloads.
- [ ] 60-second replay window enforced with explicit edge tests.
- [ ] `normalize_inbound` covers `Issue`, `Comment`, `Project` events.
- [ ] `call("graphql", ...)` returns Ok / Err correctly per response
      shape.
- [ ] **No v0.1.0 tag cut until [harn#345](https://github.com/burin-labs/harn/issues/345)
      and [harn#346](https://github.com/burin-labs/harn/issues/346)
      both land.**

## Future work (not v0)

- `linear-sdk-harn` — typed GraphQL SDK via codegen from Linear's
  introspection schema. Out of scope here. File when there's a real
  consumer.
