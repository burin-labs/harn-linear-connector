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
