# CLAUDE.md - harn-linear-connector

Pure-Harn connector package for Linear webhooks and GraphQL outbound calls.

Shared Harn connector authoring rules live in the canonical guide:

- https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md

Keep this file limited to provider-specific notes and local hazards. Add shared connector guidance
to the Harn guide first.

## Provider Notes

- Webhook verification uses `linear-signature` with the Linear webhook secret and enforces the
  provider replay window.
- Delivery ids come from `linear-delivery`; use them as stable dedupe material when available.
- Linear has no OpenAPI surface. Keep outbound behavior on Harn `std/graphql` helpers unless a
  separate generated GraphQL SDK package is created.
- OAuth access tokens use `Authorization: Bearer`; personal API keys use `Authorization: <api_key>`.
