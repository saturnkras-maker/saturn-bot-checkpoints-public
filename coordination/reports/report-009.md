---
report_id: report-009
tz_id: tz-009
status: waiting_for_prod_smoke_go_signal
phase: iter2-block3-schedulers-privacy-sender
created_at: 2026-05-05T00:20:00+07:00
base_sha: 21702f3
needs_human: yes
human_decision_needed: approve_prod_smoke
---

# Report-009 — TZ-009 / Iter-2 Block 3

<!-- markdownlint-disable MD013 -->

## Summary

TZ-009 Block 3 is implemented up to the required human gate.

Done:

- Published `coordination/tz/tz-009-iter2-block3-schedulers-privacy-sender.md`.
- Added closed pilot recipients config for Bitrix users `1` and `143` only.
- Added scheduler windows: morning `08:30` and evening `18:30`, `Asia/Barnaul`.
- Added privacy whitelist enforcement: exact `[1, 143]`; expansion raises `RuntimeError`.
- Added Bitrix IM outbox model and Alembic migration `0024_pilot_delivery_outbox`.
- Added sender path with dry-run guard before live `im.message.add`.
- Added e2e/dry-run artifacts under `artifacts/iter2-block3/`.

Not done intentionally:

- Live prod smoke was not executed because TZ-009 requires explicit Vladimir go-signal.

## PRs

- PR #40 — `Accept audit-only risk for closed pilot [checkpoint]`, merged.
- PR #41 — `Publish tz-009 block3 schedulers privacy sender [checkpoint]`, merged.
- PR #42 — `Implement tz-009 block3 scheduler privacy sender [checkpoint]`, merged.
- PR #43 — this report PR.

## Verification

Commands executed:

```bash
.venv/bin/ruff check src/services/pilot_delivery.py src/services/pilot_scheduler.py tests/services/test_pilot_delivery.py tests/services/test_pilot_scheduler.py src/db/p2_models.py
.venv/bin/python -m pytest tests/services/test_pilot_delivery.py tests/services/test_pilot_scheduler.py tests/services/test_p3_signal_runner.py tests/services/test_workday_tracker.py -q
.venv/bin/python -m pytest -q
```

Results:

- Targeted pilot suite: `29 passed in 0.10s`.
- Full suite: `880 passed, 22 warnings in 131.28s (0:02:11)`.
- DoD threshold `>= 880 passed`: met exactly.

Warnings are pre-existing smoke mark/deprecation warnings; no new functional regression found.

## Iter-2 Phase DoD summary

What entered Iter-2:

- Block 1 foundation/scope contour.
- Block 2 detectors/workday tracker accepted with risk for closed pilot.
- Block 3 scheduler/privacy/sender/outbox/e2e dry-run.

What remains for Iter-3:

- `timeman` primary source for real workday boundaries.
- `workday_sessions` persistent migration/model.
- Expansion beyond closed pilot only after explicit Iter-3 go-signal.
- Real manager rollout rules/permission matrix.

Open questions:

- Approve or reject live prod smoke for Bitrix users `1` and `143`.

## Prod smoke gate

Prepared live smoke:

- recipients: `1` and `143`;
- timezone: `Asia/Barnaul`;
- channel: Bitrix IM only;
- message prefix: `[PILOT-SMOKE iter2-block3]`.

Required decision from Vladimir:

- `go`: execute live prod smoke now;
- `no-go`: keep Iter-2 at dry-run and move to monitoring mode without live smoke.
