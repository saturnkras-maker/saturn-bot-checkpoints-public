---
tz_id: tz-008
status: active
phase: coordination-hardening-and-pilot-expansion
track: 1-of-2
created_at: 2026-05-04T12:15:00Z
base_sha: 58c8e124fa45  # уточни актуальный HEAD private/main на момент исполнения
scope_roots:
  - coordination/
depends_on: tz-007
---

# tz-008 — coordination hardening: формализация правил PR и неявных инвариантов

<!-- markdownlint-disable MD013 MD032 MD033 -->

## Контекст и доверие

Это первый из двух ёмких треков новой фазы. Фазовый формат ТЗ сохранён: цель, DoD, границы — твои, декомпозиция на PR и формулировки локальных gates — тоже твои. Обязанность останавливаться по needs_human: yes при любом сомнении — тоже сохранена.

Основание для этого трека: за tz-001..tz-007 мы несколько раз наступали на неявные правила, которые обнаруживались только постфактум через skipped run, false-positive grep, нарушенную формулировку инварианта. Каждый такой случай успешно разбирался, но если правило не закреплено в документации, оно повторится. Цель трека — закрыть этот долг одним PR.

## Цель трека

Вынести в coordination/README.md новый раздел "Правила PR в этом репозитории", в котором явно зафиксированы все накопленные неявные инварианты. Для каждого инварианта — формулировка правила, пример нарушения (по возможности — реальный из истории tz-001..tz-007), и явное указание последствия (skipped run / needs_human: yes / stop / red gate / rollback).

## Definition of Done

1. В coordination/README.md существует новый раздел "## Правила PR в этом репозитории" (или эквивалентный заголовок того же уровня). Раздел структурирован по пунктам, каждый пункт — одно правило.

2. Раздел содержит не менее 8 правил, покрывающих как минимум:
   - **Обязательный маркер [checkpoint] в squash title.** Без него checkpoint-publisher.yml даёт skipped run. Пример нарушения: PR #27 в tz-007, починен корректирующим PR #28. Последствие: skipped — stop по needs_human: yes, требуется корректирующий PR.
   - **Grep-инвариант v2 (по природе контента).** Проверяется только код/конфигурация/secrets. Документы coordination/ и их зеркала в .checkpoints/ — не проверяются. Формулировка и обоснование уже есть в разделе "Checkpoint-contour grep invariant" того же README — сошлись на него явной перекрёстной ссылкой, не дублируй текст целиком.
   - **Loop guard у checkpoint-publisher.yml.** Workflow не должен триггериться на собственные публикационные коммиты. Если триггерится — red flag, needs_human: yes. Формулировка как правило координации, не как правка workflow (workflow read-only в рамках любой фазы фронт2).
   - **Whitelist scope_roots фронт2.** Разрешённые корни правок для любой фазы фронт2: prompts/, content_samples/, tests/anonymization/, coordination/. Всё остальное — read-only. Любой выход за scope_roots — needs_human: yes до явного расширения через новое ТЗ.
   - **Правила force-push.** На feature-ветках — только --force-with-lease, никогда --force без lease. На main — force-push запрещён в любой форме. Нарушение — критическая угроза.
   - **Roles whitelist для контента.** В образцах content_samples/ допустимы только роли: сотрудник, руководитель, коллега. Любое реальное имя или роль вне whitelist — red gate.
   - **Распределение по zones 3/3/2/2.** Для типов morning/evening/friday/monthly/quarterly — распределение green/white/yellow/red = 3/3/2/2 в content_samples/<type>/. Для fallback — по подтипам no_data/partial_signals/detector_error/input_anomaly = 3/3/2/2.
   - **Line-count коридоры по типам.** morning 3–4, evening 3–4, friday 5–7, monthly 6–8, quarterly 8–10, fallback 2–4. Допуск ±1. Коридоры установлены и не расширяются автономно.

3. Если ты видишь дополнительные неявные инварианты, которые проявлялись в tz-001..tz-007 и не попали в список выше, — добавь их девятым+ пунктом с тем же форматом (правило + пример нарушения + последствие). Это приветствуется, но не обязательно.

4. В каждом пункте:
   - Явная формулировка правила одной-двумя короткими фразами.
   - Пример нарушения: либо реальный из истории (со ссылкой на tz-NNN или PR #NN), либо гипотетический с пометкой "гипотетический".
   - Последствие: одно из {skipped run, needs_human: yes, red gate, stop, критическая угроза с rollback}.

5. Структура раздела не должна конфликтовать с существующими разделами coordination/README.md. Если ты видишь дублирование с существующим разделом "### Checkpoint-contour grep invariant" — не дублируй, сошлись.

6. Audit внутри этого трека не требуется. Финальный audit фазы выполняется в треке 2 (отдельное ТЗ). Здесь достаточно локальных gates PR + финального отчёта report-008.

## Локальные gates (рекомендуемый минимум, формулировку уточняешь сам)

- Scope: diff ровно один файл — coordination/README.md.
- Существующие разделы README не изменены (diff содержит только добавление нового раздела в конец или в структурно правильное место, без переписывания старых).
- Раздел содержит ≥8 правил из обязательного списка + опциональные.
- Каждое правило содержит три компонента (формулировка, пример, последствие).
- markdownlint-cli2: 0.
- git diff --check: 0.

Fix-циклы: не более 2 подряд.

## Границы

- scope_roots: coordination/. Остальное — read-only.
- Существующие файлы coordination/ вне coordination/README.md — не трогать в этом треке.
- Никаких правок в prompts/, content_samples/, tests/anonymization/.
- Никаких правок кода, workflow, scripts, secrets.

## Инварианты безопасности (полный список для фазы, применим ко всем трекам)

- archive/readme-stub на 44a5c8c2838adaa900d42b91c08218ad8c64d742 — не тронут.
- feat/p3-m1-discovery-and-v1-scaffold на eb4f82edeb902402b1011d27598d2ef384409429 и его worktree — не тронуты.
- Checkpoint-contour grep invariant v2 соблюдён.
- secrets/deploy keys/branch protection — не меняются.
- public repo — только whitelist.
- bitrix-bot-mvp — не трогается.
- prompts/_shared/context_schema.md — TBD-поля не закрываются в этой фазе ни в одном треке.

## Критические угрозы (needs_human: yes немедленно)

- force-push в main в любом виде.
- Правки вне scope_roots.
- Правки .github/workflows/, scripts/, src/, secrets.
- Затрагивание scaffold branch или его worktree.
- Потеря archive/readme-stub.
- Утечка в public вне разрешённых путей.
- Skipped run checkpoint-publisher.yml после штатного squash merge (проверь [checkpoint] в title перед мержем).
- Самостоятельное закрытие TBD в context_schema.md.
- Переписывание существующих разделов coordination/README.md вместо добавления нового.
- Более 2 fix-циклов подряд на одном gate.
- Любая ситуация вне явных правил ТЗ, где цена ошибки неочевидна.

## Запреты

- Добавление мессенджер-интеграций в любое место.
- Расширение whitelist публикации в scripts/generate_checkpoint.py.
- Правка существующих tz-NNN, report-NNN, audit-файлов coordination/.
- Правки в bitrix-bot-mvp.
- Force-push на main.
- Создание новых подпапок в coordination/ в рамках этого трека (audits/ уже существует, reports/ и tz/ тоже; если кажется, что нужна новая — needs_human).

## Отчётность

- Финальный отчёт: coordination/reports/report-008.md по контракту, status: success либо partial.
- В отчёт обязательно включить список правил, которые ты зафиксировал в README, с кратким пояснением по каждому (одна строка на правило). Это нужно, чтобы координатор мог сверить трек без чтения README целиком.
- Промежуточные отчёты со status: in_progress — по твоему усмотрению.
- PR-флоу: минимум 2 PR (tz-008 push + report-008 push), основной content-PR по правилам — отдельно, squash merge с [checkpoint] в title. Без force-push. --delete-branch=false.

## Завершение трека

После мержа report-008 с status: success — трек 1 закрыт, жди следующего ТЗ (tz-009 — pilot expansion + phase closure). Не начинай трек 2 сам.
