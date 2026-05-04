---
tz_id: tz-002
status: active
created_at: 2026-05-04T06:30:00Z
base_sha: fbf3c6fca4830d0345aa9a340e18059eaaa4afe9
scope_files:
  - tests/anonymization/pairs/batch-1/*.md
  - tests/anonymization/pairs/README.md
  - tests/anonymization/length_report.txt
depends_on: tz-001
---

# tz-002 — anonymization pairs batch 1

<!-- markdownlint-disable MD013 MD032 -->

Цель: ввести 5 пар "raw → anonymized" для smoke-проверки пайплайна обезличивания. Это первый из двух батчей (batch-2 = ещё 5 пар, отдельным ТЗ позже).

Состав PR (ровно эти пути, ничего другого):
- tests/anonymization/pairs/README.md — контракт папки pairs/, формат пары, правила раскладки по батчам.
- tests/anonymization/pairs/batch-1/pair-01-raw.md
- tests/anonymization/pairs/batch-1/pair-01-anonymized.md
- tests/anonymization/pairs/batch-1/pair-02-raw.md
- tests/anonymization/pairs/batch-1/pair-02-anonymized.md
- tests/anonymization/pairs/batch-1/pair-03-raw.md
- tests/anonymization/pairs/batch-1/pair-03-anonymized.md
- tests/anonymization/pairs/batch-1/pair-04-raw.md
- tests/anonymization/pairs/batch-1/pair-04-anonymized.md
- tests/anonymization/pairs/batch-1/pair-05-raw.md
- tests/anonymization/pairs/batch-1/pair-05-anonymized.md
- tests/anonymization/length_report.txt — дописать секцию anonymization_pairs_batch_1 с длинами обоих вариантов каждой пары. Существующие секции morning/evening/friday/monthly/quarterly/fallback сохранить.

Требования к raw-варианту каждой пары:
- Типичный утренний/вечерний сигнал из области, которую уже покрывают content_samples/ (ровно по одному примеру на пару из: morning, evening, friday, monthly, quarterly — по одному на подтип).
- Обязательно включать хотя бы 3 маркера, которые должны быть удалены/замаскированы: вымышленная ФИО (типа Иванов И.И., не реальное имя), вымышленный номер задачи/тикета (формат TASK-1234), вымышленная метрика с точным значением (например, 17.4%), при необходимости вымышленное название проекта (формат Project-Alpha).
- Никаких реальных имён, фамилий, email, телефонов, доменов компании, реальных сумм. Всё синтетическое.
- Длина 6–8 строк.

Требования к anonymized-варианту каждой пары:
- Все целевые маркеры заменены на обезличенные плейсхолдеры по фиксированным правилам:
  - ФИО → сотрудник / руководитель / коллега (выбор по роли в исходнике).
  - TASK-1234 → [задача].
  - Числовая метрика с точным значением → диапазон-плейсхолдер [значение] или [диапазон].
  - Project-Alpha → [проект].
- Сохранена структура и тон исходника. Смысл не искажается.
- 0 совпадений со stopwords.
- Длина 6–8 строк.

YAML-шапка каждого файла pair-NN-*.md обязательна:
  pair_id: pair-NN
  variant: raw | anonymized
  source_type: one of {morning, evening, friday, monthly, quarterly}
  markers_count: integer (для raw, количество маркеров, подлежащих замене)
  markers_replaced: integer (для anonymized, должно равняться markers_count парного raw)

Локальные gates (8 пунктов):
1. Scope: diff относительно private/main содержит ровно 12 путей из списка выше.
2. Pair integrity: для каждого pair-NN есть оба файла raw и anonymized; markers_count == markers_replaced.
3. Stopwords grep по anonymized-файлам: 0 совпадений. По raw-файлам — игнорируется (raw намеренно содержит маркеры).
4. Placeholder whitelist в anonymized: допустимы только [задача], [значение], [диапазон], [проект], сотрудник, руководитель, коллега. Любой другой плейсхолдер или реальное имя — красный gate.
5. Source-type coverage: по одному pair на каждый source_type из {morning, evening, friday, monthly, quarterly}. Итого 5.
6. Line-count: 6–8 строк ±1 в обоих вариантах каждой пары.
7. git diff --check: 0.
8. markdownlint-cli2 по новым файлам: 0.

Красный gate — минимальная правка + повтор, максимум 2 цикла, дальше stop.

PR-флоу:
- Branch: feat/p3-m1-anonymization-pairs-batch-1 от актуального private/main.
- Commit: "Add anonymization pairs batch 1 (5 pairs)".
- gh pr create --base main, дождаться зелёных автопроверок, squash merge, дождаться success run checkpoint-publisher.yml, сверить public latest.json.commit с triggering squash-коммитом.
- Fix-коммиты по автопроверкам: максимум 2 подряд.

Итоговый отчёт (дублируется в coordination/reports/report-002.md и кратко в тред заказчика):
- base SHA, branch, commit SHA, PR URL, squash merge SHA, run URL, public latest до/после
- gates 8/8 с пунктами
- подтверждение, что coordination/tz/tz-002-* и coordination/reports/report-002.md доступны по public raw-URL
- инварианты безопасности (единый блок)
- next: ready to start anonymization-pairs-batch-2 on command

Критические угрозы (только здесь needs_human: yes):
- force-push в main, переписывание истории main
- правки вне 12 целевых путей (tests/anonymization/pairs/batch-1/ + README + length_report.txt)
- правки в docs/, src/, .github/workflows/, scripts/, prompts/, content_samples/, .checkpoints/
- утечка в public repo вне whitelist+coordination+.checkpoints
- операции с secrets, deploy keys, branch protection
- затрагивание feat/p3-m1-discovery-and-v1-scaffold или его worktree
- потеря archive/readme-stub (44a5c8c)
- появление мессенджер-интеграций где-либо в контуре checkpoints
- skipped у checkpoint-publisher.yml после штатного squash merge
- более 2 циклов правок gates или 2 fix-коммитов подряд
- реальные ФИО/email/телефоны/домены в raw-файлах (допускаются только синтетические)
- ситуация, не покрытая правилами этого ТЗ

Запрещено:
- Правки вне 12 целевых путей.
- Любые изменения .github/workflows/, scripts/, prompts/, content_samples/.
- force-push в main в любом виде.
- Реальные персональные данные в raw-файлах. Только синтетические маркеры.
- Переиспользование placeholder'ов вне whitelist (любые новые типы плейсхолдеров — красный gate).
- Действия в bitrix-bot-mvp и write в public repo напрямую.
- мессенджер-уведомления в любом виде.
