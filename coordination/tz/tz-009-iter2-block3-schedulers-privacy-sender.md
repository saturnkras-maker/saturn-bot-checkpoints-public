---
tz_id: tz-009
status: active
phase: iter2-block3-schedulers-privacy-sender
created_at: 2026-05-04T20:32:00+07:00
base_sha: e1e2fb8
previous_tz: tz-008
needs_human_policy: stop_on_real_blocker
is_final_block_of_phase: true
---

<!-- markdownlint-disable MD004 MD007 MD013 MD026 MD032 MD033 -->

# TZ-009 — Iter-2 / Block 3: Schedulers + Privacy + Sender + Integration tests + Phase DoD

## Контекст

Block 1 (foundation, scopes) и Block 2 (workday tracker + детекторы) закрыты. Финальный блок фазы Iter-2. Замыкает цепочку в end-to-end: от детектора до доставки текста в Bitrix IM двум получателям.

Это ЗАКРЫТЫЙ пилот на двух админов. Рассылки по отделам, manager-local матрицам, permission-mirroring на большую группу — НЕ нужны. Простой whitelist из двух user_id.

## Получатели (recipients) — вшиты жёстко

config/pilot_recipients.yaml:

recipients:
  - bitrix_user_id: 1
    name_full: "Владимир"
    role: admin_primary
    timezone: Asia/Barnaul
    salutation: "Владимир"
  - bitrix_user_id: 143
    name_full: "Сергей Иванович Костусев"
    role: admin_secondary
    timezone: Asia/Barnaul
    salutation: "Сергей Иванович"

portal: portalspl.ru
delivery_channel: bitrix_im
telegram_allowed: false
audience_policy: closed_pilot_admins_only
expansion_allowed_from: iter3_explicit_go_signal

## Инварианты (строгие)

- archive/readme-stub = 44a5c8c2838a — не трогать.
- feat/p3-m1-discovery-and-v1-scaffold + worktree — не трогать.
- Checkpoints-инварианты: никаких мессенджер-интеграций вне ADR-007.
- bitrix-bot-mvp не трогать.
- Telegram на инфра-контуре запрещён. Только Bitrix IM для sender'а.
- Секреты/env — не коммитить.
- Расширение списка получателей за пределы [1, 143] ЗАПРЕЩЕНО без отдельного go-signal Владимира.

## Scope блока

### 1. Schedulers

Два cron-like задания:
- morning_window: 08:00–09:00 Asia/Barnaul, по одному тику на каждого recipient. Собирает сигналы за прошедший рабочий день + overnight backlog. Дедуп (recipient_id, signal_fingerprint, date).
- evening_window: 18:00–19:00 Asia/Barnaul. Собирает сигналы за текущий рабочий день.

config/schedulers.yaml: cron expressions, window bounds, grace period.

Артефакт: artifacts/iter2-block3/schedulers_design.md — таблица окон, обработка пропусков (бот был down), идемпотентность.

### 2. Privacy filter

- Жёсткий whitelist по bitrix_user_id из pilot_recipients.yaml.
- Попытка sender'а отправить вне whitelist → RuntimeError + alert + отказ.
- Per-recipient scope: user_id=1 видит всё, user_id=143 видит всё (админ-наблюдатель пилота).
- В Iter-3 правила будут переопределены под реальных менеджеров с permission-mirroring.

Артефакт: artifacts/iter2-block3/privacy_pilot.md — whitelist enforcement + тест-кейсы попыток отправки не-whitelist ID (падает).

### 3. Last-hop sender — Bitrix IM через ADR-007

- Bitrix REST im.message.add (или эквивалент из ADR-007) через webhook 62.
- На каждого recipient — отдельное сообщение (1:1 с ботом Мудрый Дед bot_id=2350).
- Текст — результат p3_prompt → p3_renderer, без пост-обработки.
- Ретраи: 3 попытки exponential backoff (1s/3s/9s); failover — alert-worker.
- Идемпотентность: outbox pilot_deliveries (recipient_id, window, date, signal_fingerprint, status, attempts, sent_at, bitrix_message_id).
- Alembic миграция на base предыдущего head.
- Telegram/альт. каналы — RuntimeError если попытка.

Артефакт: artifacts/iter2-block3/sender_bitrix_im.md — flow, ретраи, outbox, ошибки API.

### 4. End-to-end integration tests

Dry-run на синтетических audit-данных: tracker → детекторы → signals → frame → prompt → text → sender (логирует payload, не отправляет).

Минимум 3 сценария: STAGNATION на одной задаче, пустой день (0 сигналов → sender пропускает), SPIKE+SILENCE на одном recipient.

+ Реальный prod smoke на portalspl.ru: "[PILOT-SMOKE iter2-block3]" двум user_id 1 и 143. Только после явного go-signal Владимира в треде.

Артефакты: artifacts/iter2-block3/e2e_tests.md + artifacts/iter2-block3/prod_smoke_result.md (после smoke).

## DoD Блока 3 и всей фазы Iter-2

- Schedulers реализованы, два окна идемпотентны, тесты зелёные.
- Privacy whitelist enforces [1, 143], попытка вне списка → RuntimeError (тест зелёный).
- Sender доставляет в Bitrix IM через ADR-007, outbox + миграция, ретраи (mocked API).
- E2E интеграционный тест на 3+ сценариях зелёный.
- Prod smoke выполнен после go-signal, оба сообщения дошли, подтверждение Владимира.
- Full suite ≥ 880 passed, ноль регрессов (вывод явно в отчёте).
- Все артефакты с [checkpoint].
- report-009.md через отдельный infra-PR + секция "Iter-2 Phase DoD summary": что вошло, что в Iter-3, открытые вопросы.

## Правила исполнения

- Автономно. Реальный блокер → needs_human: yes с фактами. НЕ обходить требования наощупь, НЕ заменять Bitrix IM на Telegram, НЕ расширять whitelist.
- Если im.message.add требует scope, которого нет на webhook 62 — эскалируй как с timeman в tz-007.
- Full suite прогонять обязательно, вывод в report.
- Prod smoke — ТОЛЬКО после явного go-signal Владимира. До него — dry-run.
- Секреты не коммитить. Коммиты с [checkpoint].
- Промежуточные in_progress отчёты — на твоё усмотрение.
- В тред — только финальное резюме + запрос go-signal на prod smoke.

## После закрытия Блока 3

Iter-2 закрыта. Фаза переходит в monitoring mode. План Iter-3 — отдельная фаза, не начинать до явного go-signal Владимира.
