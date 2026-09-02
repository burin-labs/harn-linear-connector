## What changed

<!--
Three to five sentences. Say what you changed and why, in plain language.

Example:
Inbound webhooks with a valid signature but an unknown event type were
returning a parse error instead of an ignore result. The normalizer now
returns an explicit ignore envelope for event types outside the supported
set, so the orchestrator stops treating a routine unknown event as a
delivery failure. A fixture covers one supported and one unsupported event
type. No change to the outbound dispatch surface.
-->

## Evidence

<!-- The command you ran and what it reported. `harn test` output, a fixture
that fails before the change and passes after, or a link to the CI run. -->

## Checklist

- [ ] The title follows `[Area] Sentence case summary`.
- [ ] No edits between the `HARN SHARED AGENT CONTRACT` markers in `AGENTS.md`.
- [ ] No hand edits to `.github/workflows/ci.yml` or `.github/dependabot.yml`.
- [ ] Secrets, tokens, and signatures are redacted from every example.
