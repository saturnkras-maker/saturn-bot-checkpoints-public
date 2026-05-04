---
tz_id: tz-010
phase: iter3-workday-sessions-timeman-primary
status: implemented_local_pr_ready
created_at: 2026-05-05T00:40:00+07:00
needs_human: yes
human_gate: merge_pr_and_apply_production_migration
---

# TZ-010 — Iter-3 Workday Sessions + Timeman Primary

<!-- markdownlint-disable MD013 -->

## Goal

Close the Iter-2 audit-only debt by introducing `workday_sessions` persistence and making Bitrix `timeman` a primary workday presence source for detector calibration.

## Scope

- Add `workday_sessions` model and Alembic migration.
- Enforce strict source enum at DB level: `timeman`, `activity_after_09`, `manual`, `unknown`.
- Keep live Bitrix REST calls outside `WorkdayTracker`.
- Add async sync layer that normalizes read-only `timeman.status` into `workday_sessions`.
- Store timezone per session; resolve it in sync code via employee directory when available, caller config next, then portal default `Asia/Barnaul`.
- Update `WorkdayTracker` so `SILENCE` uses timeman presence and does not rely only on `BotMessage`/`TaskEvent`.
- Keep `SPIKE`/`DROP` activity-based to avoid distorting baselines with mere presence.
- Preserve old detector behavior when the new table is absent in legacy/local test fixtures.

## Out of scope

- Applying production migration.
- Expanding pilot whitelist beyond `1` and `143`.
- Live manager rollout.
- Replacing Bitrix IM delivery channel.
- Iter-3 manager expansion rules.

## Runtime finding

The current runtime webhook answers `timeman.status` for users `1` and `143`.
`timeman.entries.get` returns `404 Not Found` on the portal, so the implemented safe primary source for now is `timeman.status` for the current day.

Known limitation: historical multi-interval backfill depends on resolving why `timeman.entries.get` is unavailable on this portal. Do not expand whitelist or add Telegram/alternate write paths for this TZ.

## Human gates

- PR stays on hold until phantom-delivery/daily-rhythm scheduler incident is closed.
- Merge PR only after Vladimir approval.
- Apply production migration only after Vladimir approval.
- No live delivery changes in this TZ.
