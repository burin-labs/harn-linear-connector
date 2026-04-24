# harn-linear-connector

Pure-Harn Linear connector for the Harn orchestrator. Verifies inbound
webhook signatures, enforces Linear's 60-second replay window, normalizes
Linear event payloads to the canonical `TriggerEvent` shape, and dispatches
outbound GraphQL queries.

> **Status: pre-alpha** — actively developed in tandem with
> [burin-labs/harn](https://github.com/burin-labs/harn). See the
> [Pure-Harn Connectors Pivot epic #350](https://github.com/burin-labs/harn/issues/350).

This is an **inbound + outbound** connector implementing Harn Connector
Contract v1. It requires a Harn version that includes
[harn#464](https://github.com/burin-labs/harn/issues/464) and
[harn#468](https://github.com/burin-labs/harn/issues/468).

Linear has no OpenAPI spec — its public API is GraphQL. A typed
`linear-sdk-harn` would be a separate, GraphQL-codegen-based effort and is
not bundled here.

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
harn check src/lib.harn
harn lint src/lib.harn
harn fmt --check src/lib.harn
harn connector check .
for test in tests/*.harn; do harn run "$test"; done
```


## License

Dual-licensed under MIT and Apache-2.0.

- [LICENSE-MIT](./LICENSE-MIT)
- [LICENSE-APACHE](./LICENSE-APACHE)
