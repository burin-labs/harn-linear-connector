# Repository settings proposal

This is a proposal, not a change. Repository and organization settings belong
to the founder. Nothing here has been applied.

## Context

`harn-linear-connector` is public so that it gets free hosted CI and so that
`harn add github.com/burin-labs/harn-linear-connector@<tag>` resolves without
credentials. It is not public to solicit contributions. Every commit on
`main` comes from the release bot or the founder, and the repository has no
external issue or pull request history.

The current settings advertise collaboration surfaces that nobody staffs. An
outside reader who files an issue here gets silence, which is worse than being
told plainly where to go.

## Proposed changes

### 1. Disable issues

Issues are enabled and unused. The connector contract and the decision to
change a connector live in `burin-labs/harn`, and `CONTRIBUTING.md` now
routes reports there with the `area/connectors` label.

Turning issues off here removes the dead end. It is reversible, and closed
issues are preserved and stay readable if the tracker is ever re-enabled.

Confirm before applying: this repository has no open issues.

### 2. Leave discussions off

Discussions are already disabled. No change requested. Recorded so a future
sweep does not enable them by reflex.

### 3. Restrict who can open a pull request against `main`

Outside contributions are not accepted, but a fork can still open a pull
request, which spends hosted CI minutes on a change that will be closed.

Restrict pull requests targeting `main` to organization members. In GitHub
terms this is a branch ruleset on `main` with a bypass list limited to the
org and the release bot, not a change to repository visibility. The package
must stay publicly readable so `harn add` keeps working.

### 4. Apply the label taxonomy

`.github/labels.yml` records the shared priority, status, and effort
categories plus a single `area/core`. The labels were created through the
API during this sweep. The file is the record of what should exist, so a
later drift check has something to compare against.

## Not proposed

- Archiving the repository. It is still built against and still receives
  runtime bumps.
- Making the repository private. That breaks `harn add` for every consumer.
- Removing the shared agent contract or the fleet-managed CI workflow.
