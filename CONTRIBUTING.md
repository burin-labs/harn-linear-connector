# Contributing to harn-linear-connector

This repository does not accept external contributions.

## Why

`harn-linear-connector` is a generated and fleet-managed connector package. It carries the
Linear side of the Harn connector contract and almost nothing else. Its
runtime pin, CI workflow, dependency configuration, and the shared agent
contract in `AGENTS.md` are all projected here by
[`burin-labs/harn-bump-fleet`](https://github.com/burin-labs/harn-bump-fleet)
and are overwritten on the next sync. A pull request against any of them is
reverted by automation rather than reviewed, so opening one wastes your time.

The connector contract itself, the vocabulary it speaks, and the decision to
add or change a connector all live in
[`burin-labs/harn`](https://github.com/burin-labs/harn). That is where the
work happens.

## Where to go instead

- Found a bug in how this connector handles Linear, or want a connector that
  does not exist yet? File it on
  [`burin-labs/harn`](https://github.com/burin-labs/harn/issues) and apply
  the `area/connectors` label.
- Include the provider, the payload or API call that misbehaves, and what you
  expected instead. Redact tokens, signatures, and account identifiers before
  you paste anything.

Security reports do not belong in a public issue. Follow the org security
policy at
[`burin-labs/.github`](https://github.com/burin-labs/.github/blob/main/SECURITY.md).

## If you have write access

Maintainers and agents working in this repository follow one convention.

### Pull request titles

Use `[Area] Sentence case summary`. The area is the part of the repository
you touched, in square brackets. The summary starts with a capital letter,
stays in sentence case, and does not end with a period.

```
[connector] Verify the webhook signature before parsing the payload
[ci] Pin the release workflow to the fleet-managed action
[docs] Explain the outbound dispatch allowlist
```

### Scope

Keep one concern per pull request. Do not edit anything between the
`<!-- BEGIN HARN SHARED AGENT CONTRACT -->` and
`<!-- END HARN SHARED AGENT CONTRACT -->` markers in `AGENTS.md`, and do not
hand-edit `.github/workflows/ci.yml` or `.github/dependabot.yml`. Change the
fleet template instead.
