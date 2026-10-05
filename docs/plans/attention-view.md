---
title: "Attention View — Plan"
status: proposed
created: 2026-10-05
upstream: Bassey240/vikunja-pwa fork
---

# Attention View — Fork Plan

## Problem

The operator's bottleneck is attention allocation, not capture. Existing Vikunja surfaces answer "where did I put this task" (projects, filters, calendar); none answer "what deserves me now". Proven model: the focus-board BB plugin's attention lanes (Pinned / Needs you / Unread / Working / Idle/Recent) with drag-to-act, snooze, and sweep rituals. HouseOps history (Eisenhower bucket moves in the Vikunja era, then `backlog.md` sections Do Now / Important / Batch / Backlog / Incoming / Done) shows the same shape: triage is the loop, movement between lanes is the core gesture.

## Goal

An `/attention` route and a new per-project view kind ("attention board") that computes lanes from live task state, plus a small client-local attention-metadata slice (pin, snooze, last-attention timestamps). Drag onto a lane or lane action IS the state change.

## Design principles

1. **No server schema changes.** All attention metadata is client-local (Zustand slice persisted through the existing offline IndexedDB, `store/slices/attention.ts`). Pin/snooze are per-device facts; sync is a later, optional concern.
2. **Lanes are computed, not stored.** A pure function `computeLanes(tasks, metadata, now)` returns `{ pinned, needsYou, dueSoon, stalled, snoozed, rest }`. Fail loud on undated tasks: they belong in `rest`, never silently.
3. **Drag-to-act.** Dropping a card writes a Vikunja action (complete, due move, priority) or lane-local action (pin, snooze preset), reusing the existing dnd-kit engines and mutation toast/undo path (`slices/mutations.ts`).
4. **Right-click first-class.** Attention cards get a native context menu (the fork's differentiator; upstream PWA has none) using the existing positioned-popup `ContextMenu.tsx` upgraded with a `contextmenu`/long-press trigger.

## Lane rules (v1)

| Lane | Rule |
|---|---|
| Pinned | `attention.pinned = true` (order: manual, drag) |
| Needs you | Overdue (due < today) OR snooze expired today OR first-attention never given since created |
| Due soon | due within configured window (default 3 days) |
| Stalled | no attention event in N days (default 14) and not pinned/snoozed; needs `lastAttentionAt` |
| Snoozed | `attention.snoozeUntil > now` |
| Rest | everything else (collapsed) |

`Needs you` merges HouseOps patterns: escalation thresholds, pending-decision items surface by due date; v1 derives purely from due/snooze, no NLP.

## Architecture mapping

- **Slice**: `src/store/slices/attention.ts` — `{ pinned: {}, snoozes: {}, lastAttentionAt: {} }`, persisted via offline-db; mark-attention hook fires when a task opens in `DetailSheet` (inspector mode) or is drag-targeted.
- **Hook**: `src/hooks/useAttentionLanes.ts` — memoized `computeLanes`, unit-tested in `tests/unit/attention-lanes.test.ts`.
- **View**: `src/components/attention/AttentionScreen.tsx` registered in `src/router.tsx` as `/attention` + sidebar entry above Today; also registered as a per-project view kind in `ProjectTasksScreen` (5th kind after list/kanban/table/gantt).
- **Interactions**: drop targets per lane header (Pin here / Snooze presets today+1d/+7d / complete), `TaskMenu` extension for Attention actions, keyboard: arrows + `S` snooze, `P` pin, `C` complete, `X` select for bulk sweep.
- **Sweep**: bulk triage mode over one lane (multi-select via `bulk-tasks.ts` slice; commit/skip per card), reusing `BulkTaskEditor.tsx`.

## Phases (test-first, per test-driven-design)

1. **P0 metadata slice** — schema + persistence + mark-attention on open. Failing tests: slice round-trip, persistence.
2. **P1 lane computation** — `computeLanes` pure function, table-driven unit tests (edge: past-due + snoozed + pinned overlap ordering), `dueSoon` window config.
3. **P2 board view** — `/attention` route + view-kind registration + lane rendering + reason chips on cards (why it's in this lane). Playwright smoke: lanes render, counts match.
4. **P3 drag-to-act + context menu** — drop targets write real Vikunja mutations through existing mutation/undo paths; native right-click trigger.
5. **P4 sweep + keyboard** — bulk triage ritual, keymap, empty/disabled states (no due dates → honest `rest` only).
6. **P5 polish + upstream prep** — settings (snooze presets, stalled N, due-soon window), CHANGELOG, decide whether this is PR-able upstream or fork-only.

## Non-goals

No notifications (Vikunja reminders already exist), no calendar changes, no capture changes, no per-user server-side attention state in v1.

## Risks

- Upstream drift: fork cut from v0.6.0; rebase before each phase.
- `Needs you` without due dates is weak in v1; label-based overrides are the identified v2 lever (a `pinned`/`attention` label as manual force-lane, mirroring HouseOps operator rulings).