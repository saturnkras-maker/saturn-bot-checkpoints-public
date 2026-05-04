---
tz_id: tz-006
status: active
created_at: 2026-05-04T08:55:00Z
base_sha: ca84137944f8e60c255bef6d9e116cd9a4ac1093
scope_files:
  - coordination/audits/second-front-audit-2026-05-04.md
depends_on: tz-005
---

# tz-006 — audit второго фронта: приёмка перед выходом в штатный режим

<!-- markdownlint-disable MD013 MD032 MD033 MD003 MD022 MD025 -->

Цель: выполнить read-only перекрёстную проверку артефактов второго фронта и зафиксировать её результат одним audit-отчётом. Никаких правок кода, контента, промптов, пар, workflow. Только чтение и фиксация.

Состав PR (ровно один путь):
- coordination/audits/second-front-audit-2026-05-04.md — новый файл с результатами аудита. Если папки coordination/audits/ ещё нет, создать её одновременно с этим файлом.

Контракт audit-файла (Markdown + YAML-шапка):

---
audit_id: second-front-2026-05-04
status: one of {clean, findings}
base_sha: ca84137944f8e60c255bef6d9e116cd9a4ac1093
checks_passed: integer / integer
findings: [] | list of objects {id, severity, area, description, reference}
created_at: ISO-8601 UTC
---

# Second front audit — 2026-05-04

Содержит 9 разделов, по одному на проверку. В каждом разделе: цель проверки, команда/метод, результат (PASS/FAIL), короткий комментарий.

Проверки (все read-only, выполнять по актуальному private/main):

1. Content-layer scope coverage.
   Ожидание: существуют prompts/morning.md, prompts/evening.md, prompts/friday.md, prompts/monthly.md, prompts/quarterly.md, prompts/fallback.md. В content_samples/ существуют подпапки morning, evening, friday, monthly, quarterly, fallback, каждая с 10 образцами (для fallback — 10 образцов с разбивкой по подтипам).
   Метод: ls prompts/, ls content_samples/*/ и подсчёт файлов.
   PASS: все 6 типов присутствуют, по 10 образцов каждая.

2. Line-count coridors.
   Ожидание: длины output-строк соответствуют коридорам по типам (morning 6–8, evening 4–6, friday 5–7, monthly 6–8, quarterly 8–10, fallback 2–4), допуск ±1.
   Метод: пройти по всем content_samples/<type>/*.md, извлечь output-строки, посчитать; сверить с коридорами, зафиксированными в prompts/<type>.md.
   PASS: все образцы в коридорах с учётом допуска.

3. Distribution по зонам и подтипам.
   Ожидание: для morning/evening/friday/monthly/quarterly — распределение green/white/yellow/red = 3/3/2/2. Для fallback — распределение no_data/partial_signals/detector_error/input_anomaly = 3/3/2/2.
   Метод: парсить YAML-шапки образцов и считать.
   PASS: все 6 типов соответствуют своим распределениям.

4. Roles whitelist.
   Ожидание: во всех образцах 6 типов используются только роли сотрудник, руководитель, коллега. Real-name grep по content_samples/: 0 совпадений с реальными ФИО (проверяется против списка-шаблона, либо по эвристике "два инициала через точку + фамилия не из whitelist").
   Метод: grep + парсинг ролей.
   PASS: whitelist соблюдён, real-name hits = 0.

5. Stopwords grep.
   Ожидание: 0 совпадений стоп-слов в content_samples/ и в anonymized-вариантах tests/anonymization/pairs/batch-1/ и batch-2/. Список стоп-слов — тот же, что использовался в gates предыдущих PR.
   Метод: grep по файлу со стоп-словами.
   PASS: 0/0.

6. Anonymization pairs integrity (cross-batch).
   Ожидание: существует ровно 10 пар (pair-01..pair-10), по 2 пары на каждый source_type; в каждой паре оба файла (raw и anonymized); markers_count == markers_replaced; плейсхолдеры в anonymized только из whitelist [задача], [значение], [диапазон], [проект], сотрудник, руководитель, коллега.
   Метод: парсинг YAML-шапок + grep плейсхолдеров.
   PASS: 10/10 пар целостны, плейсхолдеры whitelist соблюдён.

7. length_report.txt structural integrity.
   Ожидание: tests/anonymization/length_report.txt содержит секции morning, evening, friday, monthly, quarterly, fallback, anonymization_pairs_batch_1, anonymization_pairs_batch_2. Секции не дублируются, порядок не критичен, но каждая секция должна быть ровно одна.
   Метод: парсинг секций.
   PASS: 8 уникальных секций присутствуют.

8. context_schema.md TBD delta.
   Ожидание: prompts/_shared/context_schema.md не менялся относительно состояния на момент начала второго фронта (base SHA всего второго фронта = 8d6404df465e). TBD-поля signals[].source_detector, actions_deadline_label, confidence_level остаются TBD, не закрыты контентом.
   Метод: git diff 8d6404df465e..HEAD -- prompts/_shared/context_schema.md; визуальная проверка, что diff пуст либо не содержит закрытий TBD.
   PASS: diff пуст или только косметика без закрытия TBD.

9. Safety invariants cross-check.
   Ожидание по списку:
   - archive/readme-stub цел на 44a5c8c2838adaa900d42b91c08218ad8c64d742.
   - feat/p3-m1-discovery-and-v1-scaffold на eb4f82edeb902402b1011d27598d2ef384409429 и его worktree не тронут.
   - В .github/workflows/, scripts/, src/ (части контура checkpoints) нет мессенджер-интеграций по refined grep invariant (coordination/README.md, секция "Checkpoint-contour grep invariant").
   - gh secret list: только CHECKPOINTS_PUBLIC_DEPLOY_KEY.
   - public repo содержит только whitelist + coordination/ + .checkpoints/.
   - bitrix-bot-mvp не тронут.
   Метод: git ref-check, gh secret list, grep по уточнённому списку путей, ls public repo.
   PASS: все 6 пунктов OK.

Итог: если все 9 проверок PASS — status: clean, findings: []. Если хотя бы одна FAIL — status: findings, findings заполняется объектами с severity {blocker, major, minor}, area, description и точной ссылкой (файл:строка или путь).

Локальные gates (4 пункта):
1. Scope: diff содержит ровно один путь — coordination/audits/second-front-audit-2026-05-04.md.
2. Read-only: никакие другие файлы репозитория не изменены. git diff --stat за пределами целевого пути = 0 файлов.
3. YAML-шапка audit-файла парсится, обязательные поля заполнены.
4. markdownlint-cli2 по audit-файлу: 0.

Красный gate — минимальная правка audit-файла + повтор, максимум 2 цикла, дальше stop.

PR-флоу:
- Branch: infra/second-front-audit от актуального private/main.
- Commit: "audit(second-front): cross-artifact acceptance check".
- gh pr create --base main, дождаться зелёных автопроверок, squash merge, дождаться success run checkpoint-publisher.yml, сверить public latest.json.commit.

Итоговый отчёт (в coordination/reports/report-006.md + краткое резюме в тред):
- SHA мержей трёх PR связки (tz-006 push, audit PR, report-006 push)
- run URLs
- public latest до/после
- gates 4/4 OK
- public URL audit-файла
- итоговый статус audit (clean или findings) одной строкой
- если findings — краткий список, по одному объекту, без развёрнутых описаний (полные формулировки живут в audit-файле)
- next: "второй фронт закрыт, готов к выходу в штатный режим" (если clean) либо "требуется решение по findings" (если findings)

Критические угрозы (только здесь needs_human: yes):
- force-push в main
- любые правки вне целевого пути audit-PR (audit — read-only по контракту)
- правки .github/workflows/, scripts/, src/, prompts/, content_samples/, tests/anonymization/, coordination/tz/, coordination/reports/ вне публикации tz-006 и report-006 через их собственные mini-PR
- затрагивание feat/p3-m1-discovery-and-v1-scaffold или его worktree
- потеря archive/readme-stub (44a5c8c)
- utечка в public repo вне whitelist+coordination+.checkpoints+audits
- skipped у checkpoint-publisher.yml после штатного squash merge любого из 3 PR связки
- более 2 циклов правок audit-файла
- обнаружение несоответствия с severity blocker (любой blocker-finding — немедленный stop с needs_human: yes и выходом к заказчику до записи в audit-файл; в audit-файл такое не дописывается без human decision)

Запрещено:
- Любые правки кода, контента, промптов, пар, workflow, скриптов в рамках этой связки. Audit — только чтение и фиксация.
- Правки существующих файлов в coordination/ вне публикации tz-006 и report-006.
- force-push.
- Расширение whitelist публикации в scripts/generate_checkpoint.py. Если потребуется опубликовать coordination/audits/ — это делается отдельным следующим ТЗ, не внутри этого.
- Действия в bitrix-bot-mvp, write в public repo напрямую.
- Добавление реальных мессенджер-интеграций.
