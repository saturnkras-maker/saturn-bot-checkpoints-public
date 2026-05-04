---
report_for: tz-008-iter2-block-2
status: success
created_at: 2026-05-04T12:45:00Z
worktree: worktrees/bitrix-bot-mvp-report-008-push
accepted_with_risk: audit_only_for_closed_pilot
---

# Report 008 — Iter-2 Block 2: workday tracker and live detectors

<!-- markdownlint-disable MD013 -->

## Summary

Iter-2 Block 2 выполнен: добавлен read-only `workday tracker` поверх локальных audit-таблиц и подключены live detector-ы `STAGNATION`, `SPIKE`, `DROP`, `SILENCE` в P3 signal контур. Для случаев, где dev-портальный `timeman` недоступен, gate покрыт синтетическим fixture на `bot_messages` + `task_events` без live Bitrix/Telegram отправок.

## Что изменено

- `src/services/workday_tracker.py` — новый агрегатор рабочего дня:
  - считает дневную активность по `BotMessage` и `TaskEvent`;
  - строит персональный baseline за N рабочих дней;
  - выделяет silent users за последние N рабочих дней.
- `src/services/p3_signal_detectors.py` — добавлены live detector-ы:
  - `STAGNATION` — существующий task-history detector оставлен live;
  - `SPIKE` — всплеск текущей активности относительно baseline;
  - `DROP` — падение текущей активности относительно baseline;
  - `SILENCE` — отсутствие активности у активного сотрудника за рабочие дни.
- `src/services/p3_signal_runner.py` — загрузка параметров workday detector-ов из `config/detectors.yaml`.
- `config/detectors.yaml` — каноническая конфигурация Block 2:
  - 4 live detector-а: `STAGNATION`, `SPIKE`, `DROP`, `SILENCE`;
  - 9 scaffold detector-ов: `OVERDUE`, `OVERLOAD`, `BOT_INACTIVE`, `MORNING_PLAN_SKIP`, `EVENING_REPORT_SKIP`, `IGNORED_LEADER_PUSH`, `EXCESSIVE_POSTPONE`, `FREQUENT_DELEGATION`, `TASK_PING_PONG`.
- `config/norms.yaml` — `SPIKE`, `DROP`, `SILENCE` добавлены в result-factor mapping.
- `tests/services/test_workday_tracker.py` — synthetic fixture и regression tests для tracker + 3 новых live detector-ов.
- `artifacts/tz-008-iter2-block2/workday_tracker_fixture.md` — описание fixture и ограничения по timeman.

## Проверки

- `make bootstrap` — PASS, локальная `.venv` собрана.
- `PYTHONPATH=src .venv/bin/python -m pytest tests/services/test_workday_tracker.py tests/services/test_p3_signal_detectors.py tests/services/test_p3_signal_runner.py -q` — PASS: `8 passed in 0.10s`.
- `PYTHONPATH=src .venv/bin/python -m ruff check src/services/workday_tracker.py src/services/p3_signal_detectors.py src/services/p3_signal_runner.py tests/services/test_workday_tracker.py` — PASS: `All checks passed!`.
- `git diff --check` — PASS.

## Ограничения и blockers

- Live Bitrix/Telegram send не выполнялся — по правилам проекта без явного подтверждения Владимира.
- `timeman` не использован как live source: по ранее зафиксированному smoke webhook 62 метод `timeman.status` возвращал `insufficient_scope`. Поэтому detector gate сделан на локальном synthetic fixture и существующих audit-таблицах.
- Blocker-ов на завершение Block 2 нет.

## Autonomy / scope

- Изменения ограничены P3 detector/workday tracker контуром, конфигом, тестом, артефактом и финальным отчётом.
- Секреты, env-файлы, production DSN, live IDs и внешние сообщения не затрагивались.

## Addendum — coordinator review after Block 2

Requested review items:

1. `timeman` / workday tracker source truth.
2. Full suite result.
3. Alembic migration / `workday_sessions` status.

### 1. Workday tracker source truth

Current Block 2 implementation is `audit-only`, not `timeman primary`.

The tracker uses local audit tables:

- `BotMessage` (`user_id`, `sent_at`)
- `TaskEvent` (`actor_user_id`, `occurred_at`)
- `EmployeeDirectory` (`bitrix_user_id`, `is_active`)

This is now documented in `artifacts/iter2-block2/workday_tracker_design.md`.

Verdict: the current tracker is acceptable only as an interim detector calibration path.
It is not sufficient as final workday-boundary truth for live `SILENCE` and `DROP`
delivery decisions.

### 2. Full suite

Full suite was run after the addendum check:

```text
PYTHONPATH=src .venv/bin/python -m pytest -q
851 passed, 22 warnings in 30.26s
```

Verdict: no regression was observed in the available full suite, but the original TZ-008
numerical DoD of `>= 870 passed` is not met by this checkout. The current suite total is
851, not 870+.

### 3. Alembic migration / workday_sessions

No Alembic migration for `workday_sessions` exists in this implementation.

Observed migration head remains:

- `migrations/versions/0023_voice_n3_task_proposals.py`

Verdict: `workday_sessions` persistence was not created. The current audit-only path uses
existing local tables and does not satisfy the original storage requirement from TZ-008.

### Addendum conclusion

`TZ-008 / Block 2` should be treated as partially accepted for detector scaffolding and
synthetic/audit calibration only.

Before `Block 3` sends real manager-facing Bitrix IM notifications based on `SILENCE` or
`DROP`, one of two explicit decisions is required:

1. Preferred: implement `timeman` primary + `workday_sessions` persistence and rerun gates.
2. Alternative: accept `audit-only` as an explicit ADR-level architecture decision, with
   the known risk that workday boundaries may be inferred from incomplete audit activity.

needs_human: yes for Block 3 go/no-go if this addendum is not accepted explicitly.
