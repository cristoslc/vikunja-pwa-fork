# AGENTS.md — vikunja-pwa-fork

This is a fork of [Bassey240/vikunja-pwa](https://github.com/Bassey240/vikunja-pwa) (AGPL-3.0), see [PURPOSE.md](PURPOSE.md).

## Fork status and remotes

- `origin` = `cristoslc/vikunja-pwa` (the fork; push here by default)
- `upstream` = `Bassey240/vikunja-pwa`
- Fork naming: directory is `vikunja-pwa-fork`, per operator fork convention. Do NOT seed collaboration surfaces (CONTRIBUTING, issue/PR templates) — those belong to upstream and must be read, not rewritten. Contributions upward go through `upstream` only with explicit operator authorization.
- Keep upstream's AGPL-3.0 `LICENSE`, README attribution, and `CHANGELOG.md` format; fork-made entries go in the fork's `CHANGELOG.md` Unreleased section.

## Upstream sync

- Rebase onto `upstream/main` before opening a PR upward. Vikunja API-compat work upstream matters: `vikunja-better-ui` targets Vikunja 2.7.0 REST, this PWA pins its own API assumptions in `src/utils/` and `src/store/`.
- Never force-push `upstream`-shared history.

## Test command

`npm run test` (runs `test:unit` node:test suites then `test:smoke` API smoke). Playwright UI smoke runs via `npm run test:smoke:ui`; CI gate is `npm run ci` (lint + build + smoke). Before every PR: run `npm run test:unit` at minimum.

## Layout notes

- Source: `src/` (React 19 + Zustand, routes in `src/router.tsx`, app shell in `src/components/AppShell.tsx`, detail panel is an inspector/sheet not a route).
- The Attention work lives per [docs/plans/attention-view.md](docs/plans/attention-view.md); decision records in `docs/adr/`.
- `docs/` otherwise is upstream's; do not restructure it.

## Worktree discipline

All non-main work happens in `.worktrees/<branch>`, project root stays on `main`.