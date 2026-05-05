<!-- markdownlint-disable-file -->
---
report_id: report-010
tz_id: tz-010
status: pr_ready_needs_human
title: Iter-3 workday_sessions + timeman primary
created_at: 2026-05-05T00:55:00+07:00
needs_human: yes
human_decision_needed: approve_pr_merge_and_prod_migration
---

# Report-010 — Iter-3 Workday Sessions + Timeman Primary


## Summary

Implemented the first Iter-3 step that removes the main Iter-2 audit-only limitation: `SILENCE` can now use persisted Bitrix timeman presence via `workday_sessions` instead of relying only on local bot/task activity.

## Changes

- `src/db/p2_models.py` — added `WorkdaySession` model with strict source CHECK constraints.
- `migrations/versions/0025_workday_sessions.py` — added `workday_sessions` migration after `0024_pilot_delivery_outbox`; verified no competing `0025` revision exists in this worktree.
- `src/services/workday_sessions_sync.py` — added async normalization/upsert layer for read-only `timeman.status` and future `timeman.entries.get` aggregation; timezone resolution stays in sync code (`employee_directory.timezone` when available → caller config → `Asia/Barnaul`).
- `src/services/workday_tracker.py` — reads `workday_sessions` synchronously; `SILENCE` treats timeman presence as active workday presence; `SPIKE`/`DROP` remain activity-based.
- `src/services/daily_rhythm/sources.py` — aligned persisted fallback source values with the strict enum (`timeman`, `activity_after_09`, `manual`, `unknown`).
- `tests/services/test_workday_tracker.py` — added timeman-primary presence coverage.
- `tests/services/test_workday_sessions_sync.py` — added upsert, entries aggregation, timezone fallback, and strict source constraint tests.
- `coordination/tz/tz-010-iter3-workday-sessions-timeman-primary.md` — recorded scope and gates.
- `artifacts/iter3-workday-sessions/timeman_scope_smoke.md` — recorded runtime scope smoke.

## Runtime smoke

Read-only Bitrix checks:

- `timeman.status` user `1`: OK.
- `timeman.status` user `143`: OK.
- `timeman.entries.get`: `404 Not Found` for both pilot users.

Decision: use `timeman.status` as primary current-day presence; keep `entries.get` support for future portal capability.

Known limitation: `timeman.entries.get` is not available on the current portal (`404 Not Found`). This is recorded as an admin/Bitrix question before historical interval backfill. No whitelist expansion or Telegram fallback was added.

## Gates

Executed:

```bash
PYTHONPATH=src .venv/bin/python -m ruff check --fix src/db/p2_models.py src/services/workday_tracker.py src/services/workday_sessions_sync.py src/services/daily_rhythm/sources.py tests/services/test_workday_tracker.py tests/services/test_workday_sessions_sync.py tests/services/daily_rhythm/test_sources.py migrations/versions/0025_workday_sessions.py
PYTHONPATH=src .venv/bin/python -m pytest tests/services/test_workday_tracker.py tests/services/test_workday_sessions_sync.py tests/services/daily_rhythm/test_sources.py tests/services/test_p3_signal_detectors.py tests/services/test_p3_signal_runner.py -q
PYTHONPATH=src .venv/bin/python -m alembic upgrade 0024_pilot_delivery_outbox:head --sql
PYTHONPATH=src .venv/bin/python -m pytest -q
```

Results:

- Targeted workday/detector/daily-rhythm suite: `19 passed in 0.16s`.
- Alembic SQL preview includes `CREATE TABLE workday_sessions`, `ck_workday_sessions_open_source`, `ck_workday_sessions_close_source`, `uq_workday_sessions_day`, `idx_workday_sessions_user_date`, `idx_workday_sessions_portal_date`.
- Full suite: `888 passed, 22 warnings in 130.87s`.

Warnings are existing smoke mark / httpx deprecation warnings.

## Invariants

- No Telegram pilot delivery added.
- No whitelist expansion.
- No live Bitrix IM sends.
- No production migration applied.
- No secrets committed.

## Needs human

Required next decisions from Vladimir:

1. Keep PR on hold until phantom-delivery/daily-rhythm scheduler incident is closed.
2. Approve PR merge only after incident closure.
3. Approve production migration apply for `0025_workday_sessions` only after PR merge approval.
4. Approve any future Iter-3 expansion beyond users `1` and `143` separately.
