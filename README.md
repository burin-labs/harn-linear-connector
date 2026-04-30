# harn-linear-connector

Pure-Harn Linear connector for the Harn orchestrator. Verifies inbound
webhook signatures, enforces Linear's 60-second replay window, normalizes
Linear event payloads to the canonical `TriggerEvent` shape, and dispatches
outbound GraphQL queries.

> **Status: pre-alpha** — this repo is a first-party connector package for
> Harn `0.7.42` or newer. CI installs the pinned CLI version from
> `.harn-version`.

This is an **inbound + outbound** connector implementing Harn Connector
Contract v1. The canonical contract docs live in the Harn repo:

- [Connector authoring](https://github.com/burin-labs/harn/blob/main/docs/src/connectors/authoring.md)
- [Connector architecture](https://github.com/burin-labs/harn/blob/main/docs/src/connectors/architecture.md)

Linear has no OpenAPI spec — its public API is GraphQL. Outbound calls use
Harn's `std/graphql` operation/envelope helpers so query documents, errors,
rate-limit metadata, and cursor pagination are handled the same way as other
GraphQL-first connector packages. A larger fully generated `linear-sdk-harn`
can build on the same substrate later.

## Install

```sh
harn add github.com/burin-labs/harn-linear-connector@main
```

For local multi-repo development, a path dependency is still useful:

```toml
[dependencies]
harn-linear-connector = { path = "../harn-linear-connector" }
```

## Usage

```harn
import linear_connector from "harn-linear-connector/default"

trigger triage on linear {
  source = {
    kind: "webhook",
    signing_secret: env("LINEAR_WEBHOOK_SECRET"),
    api_key: env("LINEAR_API_KEY"),
    events: ["Issue"],
  }
  on event {
    if event.action == "create" && event.data.priority == 1 {
      linear_connector.call("graphql", {
        query: """
          mutation Comment($issueId: String!, $body: String!) {
            commentCreate(input: { issueId: $issueId, body: $body }) {
              success
            }
          }
        """,
        variables: { issueId: event.data.id, body: "Auto-triaged: urgent." },
      })
    }
  }
}
```

## Configuration

Inbound webhooks require the Linear webhook signing secret. The connector
checks `raw.signing_secret`, `raw.metadata.signing_secret`,
`binding.config.signing_secret`, `binding.config.secrets.signing_secret`, or
managed-ingress secret aliases in `raw.metadata.secret_ids`; otherwise it reads
`linear/signing-secret` through `secret_get`.

Outbound calls require one of:

- `access_token` or `access_token_secret` for OAuth tokens. Requests use
  `Authorization: Bearer <token>`.
- `api_key` or `api_key_secret` for personal API keys. Requests use
  `Authorization: <api_key>`.

Use OAuth for workspace/user integrations and personal API keys only for local
scripts or private automation. Minimum OAuth scopes depend on the methods used:

- `read` for `list_issues`, `search`, and read-only `graphql` queries.
- `write` or narrower write scopes for mutations.
- `comments:create` is enough for `create_comment`.
- `issues:create` is only needed if future recipes add issue creation.
- Avoid `admin`; this connector does not require it for the MVP methods.

## Inbound Events

Supported Linear webhook resources are:

- `Issue` -> `linear.issue.<action>`
- `Comment` / `IssueComment` -> `linear.comment.<action>`
- `IssueLabel` -> `linear.issue_label.<action>`
- `Project` / `ProjectUpdate` -> `linear.project.<action>`
- `Cycle` -> `linear.cycle.<action>`
- `Customer` -> `linear.customer.<action>`
- `CustomerRequest` -> `linear.customer_request.<action>`

Actions are `create`, `update`, and `remove`. Unknown resources are normalized
with their lower-case resource name; unknown actions are rejected. Dedupe keys
prefer `Linear-Delivery`, then `webhookId`, then `sha256(raw_body)`.

## Outbound Methods

`call("graphql", args)` is the escape hatch. It accepts `query`, optional
`variables`, optional `operation_name` / `operationName`, auth, and optional
`api_base_url`.

Typed MVP helpers are local to this package:

- `list_issues({ filter?, first?, after?, include_archived? })`
- `update_issue({ id, changes })`
- `create_comment({ issue_id, body })`
- `search({ query, first? })`

Connection-returning methods preserve Linear pagination data in `pageInfo`.
Callers should pass `pageInfo.endCursor` as `after` while
`pageInfo.hasNextPage` is true. `search` uses Linear's `searchIssues` field and
falls back from the newer `query` argument to the older `term` argument when
Linear returns a GraphQL validation error.

GraphQL responses always include `meta` on success. Typed helper return objects
also receive this `meta` field:

- `observed_complexity`
- `complexity_estimate` as a best-effort local hint
- `complexity_warning`
- `rate_limit.requests_*`
- `rate_limit.complexity_*`

GraphQL errors throw structured objects. Partial data is preserved on
`error.data`, GraphQL errors are preserved on `error.errors`, and rate-limit
responses are classified as `error.code == "rate_limited"` when Linear returns
HTTP 429 or a GraphQL error with `extensions.code == "RATELIMITED"`.

## Development

The connector exports `provider_id`, `kinds`, `payload_schema`, `init`,
`activate`, `shutdown`, `normalize_inbound`, and `call`. `normalize_inbound`
returns `NormalizeResult` v1:

- valid Linear webhooks return `{ type: "event", event: { kind, dedupe_key, payload } }`
- failed signature, missing timestamp, replay-window, and unsupported-action
  cases return `{ type: "reject", status, headers, body }`

Dedupe keys are deterministic. The connector prefers `Linear-Delivery`, then
`webhookId`, and finally `sha256(raw_body)`. Event kinds are
`linear.<resource>.<action>`, for example `linear.issue.update` and
`linear.comment.create`.

Run the local contract and fixture suite with:

```sh
harn --version
harn check src/lib.harn
harn lint src/lib.harn
harn fmt --check src/lib.harn
harn connector check .
for test in tests/*.harn; do harn run "$test"; done
```

CI installs the pinned Harn CLI from `.harn-version` via
`cargo install harn-cli --version "$(cat .harn-version)" --locked`, then runs
the same checks plus a clean package-install smoke. Local development should use
the installed CLI and this package's `harn.toml`; it should not depend on a
sibling `~/projects/harn` checkout.

## License

Dual-licensed under MIT and Apache-2.0.

- [LICENSE-MIT](./LICENSE-MIT)
- [LICENSE-APACHE](./LICENSE-APACHE)
