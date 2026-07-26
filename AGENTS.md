# AGENTS.md

Pure-Harn connector package for Linear webhooks and GraphQL outbound calls.

Shared connector authoring rules live in the Harn guide:

- [Connector authoring guide](https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md)

Put shared connector guidance in the Harn guide and keep only
provider-specific notes and local hazards here.

`CLAUDE.md` points here. Edit `AGENTS.md` only.

## Provider notes

- Webhook verification uses `linear-signature` with the Linear webhook secret and enforces the
  provider replay window.
- Delivery IDs come from `linear-delivery`; use them as stable dedupe material when available.
- Default secret IDs are `linear/webhook-secret` for webhook verification and `linear/api-token` for
  outbound API-token auth.
- Linear has no OpenAPI surface. Keep outbound behavior on Harn `std/graphql` helpers unless a
  separate generated GraphQL SDK package is created.
- OAuth access tokens use `Authorization: Bearer`; personal API keys use `Authorization: <api_key>`.
- `search` uses `searchIssues(query:)` first and falls back to `searchIssues(term:)` when a Linear
  GraphQL schema still expects `term`.

<!-- BEGIN HARN SHARED AGENT CONTRACT: managed by harn-bump-fleet -->

## Ecosystem working agreement

- Pursue the ambitious product outcome; make the seams boring with small typed
  interfaces, explicit invariants, and deterministic projections.
- Give each behavior one semantic owner. Generate or parity-test other surfaces
  instead of maintaining competing implementations.
- Work autonomously inside approved scope. Pause for destructive, production,
  high-spend, ambiguous, or authority-expanding actions—not routine reversible work.
- Treat stop, wait, stand down, and pivot as control events for long-lived work.
- Match evidence to the claim: exercise the canonical user path, state the
  falsifier, verify liveness and recovery, and record residual blind spots.
- "Ship" means landed on main with required deploy and post-merge checks complete.

<!-- END HARN SHARED AGENT CONTRACT -->
