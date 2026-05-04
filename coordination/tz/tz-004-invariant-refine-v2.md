---
tz_id: tz-004
status: active
created_at: 2026-05-04T07:25:00Z
base_sha: c5d526c19eafa9ee7346d410957d2ea6b82b4972
scope_files:
  - coordination/README.md
  - coordination/reports/report-003.md
  - coordination/reports/report-004.md
depends_on: tz-003
---

# tz-004 — invariant refine v2 + backfill report-003 + close loop

<!-- markdownlint-disable MD013 MD032 MD007 MD001 MD022 MD023 MD029 -->

Цель: заменить формулировку grep-инварианта в coordination/README.md на однозначную, основанную на природе контента (код/конфигурация vs тексты документов), а не на списке путей; создать пропущенный report-003 по факту выполнения tz-003; создать report-004 по факту этой связки.

Состав (ровно эти пути, ничего другого):

1. coordination/README.md — заменить существующую подсекцию "### Checkpoint-contour grep invariant" на новую формулировку ниже. Если подсекция существует — заменить её целиком. Другие разделы файла не трогать.

Новая формулировка подсекции:

   ### Checkpoint-contour grep invariant
   Grep на названия запрещённых технологий (мессенджер-интеграции, в т.ч. Telegram, Slack, Discord, WhatsApp и т.п.) проверяет **только код и конфигурацию**, где появление слова-маркера означало бы реальное использование технологии:
   - .github/workflows/**
   - scripts/**
   - src/** (части, относящиеся к контуру публикации checkpoints)
   - имена secrets репозитория (gh secret list)

   Grep **не применяется** к документам и их зеркалам:
   - coordination/tz/**
   - coordination/reports/**
   - .checkpoints/coordination/** (автоматические реплики coordination/)
   - .checkpoints/latest.md, .checkpoints/index.json, .checkpoints/history/** в части, цитирующей тексты ТЗ и отчётов
   - любые будущие поддиректории coordination/, добавленные через расширение whitelist

   Причина: в текстах документов упоминание запрещённой технологии может быть частью формулировки запрета, исторической справки или обсуждения. Это не код, на поведение системы не влияет. Нарушением считается только фактическое подключение соответствующей интеграции в коде, конфигурации или secrets.

   Практическая процедура проверки: при каждом итоговом отчёте выполнять grep по списку "проверяем", ожидать 0 совпадений. Grep по списку "не проверяем" не выполняется.

2. coordination/reports/report-003.md — создать новый файл по контракту coordination/reports/README.md. Содержимое отразить фактические работы tz-003:
   - report_for: tz-003
   - status: partial (связка довела до cleanup PR, но сам report-003 остался несозданным, что и исправляется в tz-004)
   - squash_merge_sha: c5d526c19eafa9ee7346d410957d2ea6b82b4972 (cleanup PR #15)
   - public_latest_before: ebb749fa6092f40bfd2125de6a3b1b3b0df98c5b
   - public_latest_after: SHA, который будет присвоен после публикации этого файла (допустимо указать null и дополнить в теле пояснением "см. report-004 и latest.json")
   - gates_passed: 5 / 6 (gate №5 был остановлен stop-условием — ложное срабатывание инварианта на .checkpoints/**, исправлено в tz-004)
   - invariants_ok: true (по существу; буквальный gate был ложным срабатыванием)
   - В теле: список выполненных работ (tz-003 PR #14, cleanup PR #15), ссылка на ложное срабатывание, ссылка вперёд на tz-004/report-004 как закрытие.

3. coordination/reports/report-004.md — создать новый файл по контракту. Отразить работы этой связки:
   - report_for: tz-004
   - status: success
   - squash_merge_sha: SHA мержа основного PR этой связки (см. PR-флоу ниже)
   - public_latest_before: c5d526c19eafa9ee7346d410957d2ea6b82b4972
   - public_latest_after: SHA после публикации
   - gates_passed: 4 / 4
   - invariants_ok: true
   - В теле: краткое описание правки инварианта, подтверждение, что report-003 backfilled, ссылки на public URL.

Локальные gates (4 пункта):

1. Scope: diff по каждому из трёх PR этой связки содержит ровно целевые файлы, ничего другого. PR tz-004 push — один файл coordination/tz/tz-004-*.md. PR report-003 push — один файл coordination/reports/report-003.md. PR report-004 push — один файл coordination/reports/report-004.md. Основной PR invariant-refine-v2 — ровно coordination/README.md.
2. Refined grep по новому инварианту (только "проверяем", не "не проверяем"):
   - .github/workflows/**: 0 совпадений по telegram/slack/discord/whatsapp/TELEGRAM_BOT_TOKEN/TELEGRAM_CHAT_ID/api\.telegram\.org
   - scripts/**: то же
   - src/**: то же
   - gh secret list: только CHECKPOINTS_PUBLIC_DEPLOY_KEY
   Любое совпадение — красный gate.
3. markdownlint-cli2 по всем изменённым/созданным файлам: 0.
4. YAML-шапки report-003 и report-004 парсятся без ошибок и соответствуют контракту coordination/reports/README.md.

Красный gate — минимальная правка + повтор, максимум 2 цикла, дальше stop.

PR-флоу этой связки (4 PR, все squash merge, все без force-push):

A. infra/tz-004-push: один файл coordination/tz/tz-004-*.md. Commit "coord(tz): publish tz-004 invariant refine v2".
B. infra/invariant-refine-v2: один файл coordination/README.md. Commit "infra(coordination): refine grep invariant by content nature".
C. infra/report-003-push: один файл coordination/reports/report-003.md. Commit "coord(report): backfill report-003 after tz-003".
D. infra/report-004-push: один файл coordination/reports/report-004.md. Commit "coord(report): publish report-004".

Порядок: A → B → C → D. Между PR паузы не делать, если нет критических угроз. После каждого squash merge дожидаться success run checkpoint-publisher.yml. Skipped — stop.

Итоговый отчёт (только report-004.md + краткое резюме в тред заказчика):
- 4 squash SHA
- 4 run URL
- public latest после D
- gates 4/4 OK
- public URLs обоих отчётов (report-003 и report-004)
- краткая констатация: "buffer closed, новый инвариант действует начиная с base_sha этого ТЗ"

Критические угрозы (только здесь needs_human: yes):
- force-push в main
- правки вне целевых путей каждого из 4 PR
- любые правки .github/workflows/, scripts/, src/, secrets
- правка существующих записей в coordination/tz/tz-002*, tz-003*, report-002, или удаление файлов coordination/
- затрагивание feat/p3-m1-discovery-and-v1-scaffold или его worktree
- потеря archive/readme-stub (44a5c8c)
- skipped у checkpoint-publisher.yml после штатного squash merge любого из 4 PR
- более 2 циклов правок gates или 2 fix-коммитов подряд
- любое фактическое подключение мессенджер-интеграций в код/конфигурацию/secrets

Запрещено:
- Правки кода, конфигурации, workflow, scripts, secrets в этой связке. Только документы coordination/.
- Правки содержимого tz-002, tz-003, report-002. В эту связку они не входят.
- Любое расширение whitelist публикации в scripts/generate_checkpoint.py.
- force-push, merge коммиты вместо squash, удаление веток с флагом --delete-branch=true (оставляем --delete-branch=false как в предыдущих связках).
- Действия в bitrix-bot-mvp, write в public repo напрямую.
- Добавление любой реальной мессенджер-интеграции.
