---
tz_id: tz-005
status: active
created_at: 2026-05-04T07:45:00Z
base_sha: 454a011e34256b98840a7e71a37ce7a3ac63162f
scope_files:
  - tests/anonymization/pairs/batch-2/*.md
  - tests/anonymization/length_report.txt
depends_on: tz-002
---

# tz-005 — anonymization pairs batch 2

<!-- markdownlint-disable MD013 MD032 -->

Цель: ввести 5 дополнительных пар "raw → anonymized" по тому же контракту, что и batch-1. Это закрывает целевое покрытие 10 пар.

Состав PR (ровно эти пути):

- tests/anonymization/pairs/batch-2/pair-06-raw.md
- tests/anonymization/pairs/batch-2/pair-06-anonymized.md
- tests/anonymization/pairs/batch-2/pair-07-raw.md
- tests/anonymization/pairs/batch-2/pair-07-anonymized.md
- tests/anonymization/pairs/batch-2/pair-08-raw.md
- tests/anonymization/pairs/batch-2/pair-08-anonymized.md
- tests/anonymization/pairs/batch-2/pair-09-raw.md
- tests/anonymization/pairs/batch-2/pair-09-anonymized.md
- tests/anonymization/pairs/batch-2/pair-10-raw.md
- tests/anonymization/pairs/batch-2/pair-10-anonymized.md
- tests/anonymization/length_report.txt — дописать секцию anonymization_pairs_batch_2 по аналогии с batch-1. Существующие секции сохранить.

Требования к содержимому: идентичны batch-1 (tz-002), включая:

- YAML-шапка с полями pair_id, variant, source_type, markers_count, markers_replaced.
- Синтетические маркеры: ФИО (Иванов И.И. и подобное), задача (TASK-1234), числовая метрика с точным значением, проект (Project-Alpha).
- Плейсхолдеры whitelist в anonymized: [задача], [значение], [диапазон], [проект], сотрудник, руководитель, коллега.
- Длина 6–8 строк в обоих вариантах, допуск ±1.
- Покрытие source_type в batch-2: ещё по одному pair на каждый из {morning, evening, friday, monthly, quarterly}. Итого 5, и в сумме с batch-1 — по 2 pair на каждый source_type.

Локальные gates (8 пунктов) — те же, что в tz-002:

1. Scope: 11 целевых путей (10 pair-файлов + length_report.txt).
2. Pair integrity: для каждого pair-NN есть оба файла; markers_count == markers_replaced.
3. Stopwords grep по anonymized: 0.
4. Placeholder whitelist в anonymized: только разрешённые плейсхолдеры и роли.
5. Source-type coverage: по одному pair на каждый source_type. В сумме с batch-1: по 2 на тип.
6. Line-count: 6–8 ±1 в обоих вариантах.
7. git diff --check: 0.
8. markdownlint-cli2: 0.

Красный gate — минимальная правка + повтор, максимум 2 цикла, дальше stop.

PR-флоу:

- Branch: feat/p3-m1-anonymization-pairs-batch-2 от актуального private/main.
- Commit: "Add anonymization pairs batch 2 (5 pairs)".
- gh pr create --base main, дождаться зелёных автопроверок, squash merge, дождаться success run checkpoint-publisher.yml, сверить public latest.json.commit.
- Fix-коммиты по автопроверкам: максимум 2 подряд.

Итоговый отчёт (в coordination/reports/report-005.md + краткое резюме в тред заказчика):

- base SHA, branch, commit SHA, PR URL, squash merge SHA, run URL, public latest до/после
- gates 8/8 с пунктами
- подтверждение покрытия 10/10 пар (по 2 на каждый source_type)
- инварианты безопасности (единый блок)
- next: ready to start audit-PR второго фронта on command

Критические угрозы (только здесь needs_human: yes):

- force-push в main
- правки вне 11 целевых путей
- правки в .github/workflows/, scripts/, prompts/, content_samples/, coordination/ (кроме публикации tz-005 и report-005 через их собственные mini-PR)
- затрагивание feat/p3-m1-discovery-and-v1-scaffold или его worktree
- потеря archive/readme-stub (44a5c8c)
- утечка в public repo вне whitelist + coordination + .checkpoints
- операции с secrets, deploy keys, branch protection
- реальные персональные данные в raw-файлах
- skipped у checkpoint-publisher.yml после штатного squash merge
- любое фактическое подключение мессенджер-интеграций в код/конфигурацию/secrets
- более 2 циклов правок gates

Запрещено:

- Правки вне 11 целевых путей.
- Правки .github/workflows/, scripts/, prompts/, content_samples/.
- force-push.
- Реальные персональные данные. Только синтетические маркеры.
- Плейсхолдеры вне whitelist.
- Действия в bitrix-bot-mvp, write в public repo напрямую.
- Добавление реальных мессенджер-интеграций.
