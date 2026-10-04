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
- Match evidence to the claim. Use the smallest owning product-path check;
  add a falsifier for contested, load-bearing, or potentially vacuous claims.
  Record relevant controls, recovery, and blind spots without repeating proof.
- Evidence follows source and artifact identity, not the branch name. Reuse
  verified branch or merge-candidate evidence after landing when relevant code,
  build inputs, and dependencies are unchanged. Repeat affected checks only
  for a relevant change, observed failure, deployment, or packaging difference.
- "Ship" means integrated on owning main with terminal integration checks and
  applicable release or deployment checks complete. Confirm the landed change
  and merge result; do not rebuild or recapture screenshots solely for main.
- Ship a ready PR by adding the `ship` label when the repo has a Smart Ship
  caller (`.github/workflows/smart-ship.yml`); otherwise land through the merge
  queue with `gh pr merge --squash --auto`. Never `gh pr merge --admin`.
  Incidents use the org override labels `bypass-ci`, `bypass-merge-queue`, or
  `force-merge`.

<!-- END HARN SHARED AGENT CONTRACT -->

## Pull request titles

Use `[Area] Sentence case summary`, for example
`[Connector] Verify the webhook signature before parsing the payload`. The
summary starts with a capital letter and does not end with a period. See
[CONTRIBUTING.md](CONTRIBUTING.md) for scope rules and the files this
repository does not accept hand edits to.
