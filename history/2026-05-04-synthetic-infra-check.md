# Пакет: synthetic-infra-check

**Закрыт:** 2026-05-04T09:42:26+07:00
**Ветка:** infra/github-checkpoints
**Коммит:** eb4f82edeb90 feat(p3): add n3 voice boundary package

## Что вошло
- `artifacts/task-p3-m1-discovery-scaffold/final-report.md`
- `migrations/versions/0023_voice_n3_task_proposals.py`
- `src/absences/__init__.py`
- `src/absences/manager.py`
- `src/db/p2_models.py`
- `src/departments/__init__.py`
- `src/departments/resolver.py`
- `src/holidays/__init__.py`
- `src/holidays/calendar.py`
- `src/onboarding/__init__.py`
- `src/onboarding/tracker.py`
- `src/voice/anti_empty_guard.py`
- `src/voice/dialog_engine.py`
- `src/voice/limits_guard.py`
- `src/voice/receiver.py`
- `src/voice/structurer.py`
- `src/voice/task_creator.py`
- `src/voice/transcriber.py`
- `tests/services/test_p3_n3_boundaries.py`
- `tests/voice/test_n3_voice_pipeline.py`

## Тесты
- targeted: нет данных
- regression: нет данных
- full: нет данных

## Блокеры
- нужен GitHub-логин Владимира для collaborator

## Следующий шаг
проверить публикацию latest.md/latest.json в публичном репозитории

## Фрагмент final-report

# TASK-P3-M1 / N+1 — final report

Status: code checkpoint complete for discovery + v1 scaffold + P3 signal batch runner + N+1 tail package.
Branch: `feat/p3-m1-discovery-and-v1-scaffold`

## 1. Scope delivered

- Discovery/scaffold layer preserved from checkpoint:
  - `discovery/bitrix_api_report.md`
  - `discovery/llm_capabilities_report.md`
  - v1 config scaffolds under `config/`
  - signal contract: `artifacts/task-p3-m1-discovery-scaffold/signal-contract-v0.md`
- Schema/repository layer preserved from checkpoint:
  - migration `0020_create_feedback_voice_scaffold.py`
  - migration `0021_create_signal_karma_ops_layer.py`
  - SQL preview `artifacts/task-p3-m1-discovery-scaffold/alembic-0021-upgrade-head.sql`
  - `src/repositories/p3_ops_repo.py`
- N+1 schema addition:
  - migration `0022_create_daily_result_factor.py`
  - ORM model `DailyResultFactor`
  - final schema decision: daily aggregate is stored in separate table `daily_result_factor`, not embedded into karma aggregates.
- Signal detector batch:
  - `src/services/p3_signal_detectors.py`
  - `tests/services/test_p3_signal_detectors.py`
- Runner/repository flow:
  - `src/services/p3_signal_runner.py`
  - `tests/services/test_p3_signal_runner.py`
- N+1 package tests:
  - `tests/services/test_p3_n1_package.py`

## 2. Signal coverage

### Operational signals — done

- `OVERDUE` — latest task snapshot, active status, deadline threshold.
- `STAGNATION` — normalized task events, workday counting, `pilot_start_date` clean-start.
- `OVERLOAD` — active task count by responsible employee, warn/critical thresholds.
- `BOT_INACTIVE` — employee directory + last bot message age.
- `MORNING_PLAN_SKIP` — employee directory + missing/stale morning plan bot message.
- `EVENING_REPORT_SKIP` — employee directory + missing/stale evening report bot message.
- `IGNORED_LEADER_PUSH` — stale push against known task snapshot without later task event.

### Management / cyclic signals — done

- `EXCESSIVE_POSTPONE` — forward deadline moves in rolling window.
- `FREQUENT_DELEGATION` — reassignment events by actor in rolling week window.
- `TASK_PING_PONG` — unique responsible count in rolling window.
- `PUSH_OVERUSE` — pushes by sender in rolling week window.
- `REPEATED_PUSH_SAME_TASK` — repeated pushes to same employee/task in rolling window.
- `RED_ZONE_STREAK` — reads `daily_result_factor`, counts red-zone streak, threshold from `norms.yaml`.

## 3. N+1 details

- Repository helpers added for:
  - `weekly_karma_log`, `mon
…
