# ADR-0001: Attention metadata is client-local

## Context

Vikunja's server model has no per-user attention surface. The fork must persist pin, snooze-until, and last-attention timestamps per task without changing the server or waiting on upstream schema.

## Decision

Store attention metadata in a client-local Zustand slice persisted through the app's existing IndexedDB offline database. Lane membership is computed from live task state + this metadata; nothing is stored server-side in v1.

## Consequences

- Attention state is per-device; a second browser does not see pins/snoozes. Accepted for a single-operator use case; cross-device sync is a documented future lever (candidate: abuse Vikunja subscriptions or a reserved label, both reversible).
- Reinstall/clearing site data loses pins/snoozes. Mitigation: export/import in settings is on the phase list, not v1.
- Fail-loud rule: undated or unresolvable tasks fall into the explicit `rest` lane, never a hidden bucket.