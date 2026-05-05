---
report_for: tz-008-iter2-block-2
status: accepted_with_risk
created_at: 2026-05-04T12:45:00Z
worktree: worktrees/bitrix-bot-mvp-report-008-push
needs_human: no
---

# Report 008 — Iter-2 Block 2: workday tracker and live detectors

<!-- markdownlint-disable MD013 -->

## Summary

Iter-2 Block 2 принят Владимиром как `accepted_with_risk` для закрытого пилота
(Владимир + Сергей Иванович): добавлен read-only `workday tracker` поверх локальных
audit-таблиц и подключены live detector-ы `STAGNATION`, `SPIKE`, `DROP`, `SILENCE`
в P3 signal контур. Для случаев, где dev-портальный `timeman` недоступен, gate покрыт
синтетическим fixture на `bot_messages` + `task_events` без live Bitrix/Telegram
отправок.

Block 3 по TZ-009 уже выполнялся 2026-05-04T23:59+07 и обнаружил authorship bug
(`author_id=1` вместо `bot_id=2350`). Iter-2 closure заблокирован на TZ-011 sender
authorship fix + visual ack. Safe-mode rollout приостановлен; `audit-only` принят
только как Block 2 detector calibration artifact, не как разрешение на новый Block 3 start.

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
- `audit-only` принят с риском только для закрытого пилота. Это не является разрешением
  на prod smoke или live manager-facing send без отдельного go-signal Владимира.

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

Verdict: `audit-only` accepted_with_risk for closed-pilot safe mode only
(Vladimir + Sergey Ivanovich). It remains insufficient as final workday-boundary truth
for wider rollout or live `SILENCE`/`DROP` manager-facing delivery without a separate
go-signal and follow-up migration.

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

`TZ-008 / Block 2` is accepted_with_risk for detector scaffolding, synthetic/audit
calibration only. Владимир accepted the `audit-only` alternative for Block 2 closed-pilot
scope (Владимир + Сергей Иванович), but this does not authorize a new Block 3 start.

Block 3 по TZ-009 already executed 2026-05-04T23:59+07 with sender authorship bug
(`author_id=1` instead of `bot_id=2350`). Iter-2 closure is blocked on TZ-011 sender
authorship fix and visual ack. Safe-mode rollout remains paused until TZ-011 is closed.

`timeman` primary + `workday_sessions` persistence is moved to Iter-3 follow-up /
Block 2.5 debt, with gates rerun before wider rollout or live `SILENCE`/`DROP` delivery.

## Addendum — 2026-05-05 reviewer corrections

1. ADR collision fixed: `docs/adr/ADR-039-audit-only-workday-tracker-closed-pilot.md`
   was renamed to `docs/adr/ADR-040-audit-only-workday-tracker-closed-pilot.md`.
   Canonical ADR-039 remains `ADR-039-phantom-delivery-and-go-signal-gates.md` in the
   phantom/go-signal branch.
2. TZ-011 status: received and executed in `fix/tz-011-sender-authorship`; technical
   blocker resolved by preserving numeric `imbot.message.add` private target ids. Live
   smoke `1761448` read back in bot dialog with `author_id=2350`; strict visual UI ack
   remains the human gate if required for DoD.
3. Block 3 safe-mode wording corrected: no new Block 3 start/rollout before TZ-011 +
   visual ack.
4. Scheduler/audit: committed artifacts in this worktree contain only dry-run/placeholder
   digest rows; Владимир separately confirmed morning IM was empty. No second live
   phantom was found in this worktree evidence.
