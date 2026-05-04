---
tz_id: tz-003
status: active
created_at: 2026-05-04T07:20:00Z
base_sha: ebb749fa6092f40bfd2125de6a3b1b3b0df98c5b
scope_files:
  - coordination/tz/tz-002-anonymization-pairs-batch-1.md
  - coordination/reports/report-002.md
  - coordination/README.md
depends_on: tz-002
---

# tz-003 — cleanup tz-002 wording + refine checkpoint-contour invariant

<!-- markdownlint-disable MD013 MD032 MD007 -->

Цель: снять буквальное нарушение grep-инварианта в coordination/tz/tz-002-anonymization-pairs-batch-1.md без изменения смысла запрета; формально уточнить, что grep на запрещённые технологии не применяется к текстам ТЗ и отчётов; зафиксировать это в coordination/README.md; внести ремарку в report-002.md.

Состав (ровно эти пути, ничего другого):
- coordination/tz/tz-002-anonymization-pairs-batch-1.md — изменить только две строки, содержащие слово "Telegram":
  - "появление Telegram где-либо в контуре checkpoints" → "появление мессенджер-интеграций где-либо в контуре checkpoints"
  - "Telegram в любом виде." → "мессенджер-уведомления в любом виде."
  Никаких других правок в файле. Порядок строк, YAML-шапка, остальной текст — без изменений.
- coordination/reports/report-002.md — добавить в конец файла, ПОСЛЕ существующего содержимого, секцию:
  "## cleanup note (tz-003)
   В tz-002 заменены две строки с прямым упоминанием Telegram на обобщённую формулировку про мессенджер-интеграции. Batch-1 и PR #12 не затронуты. См. tz-003 и report-003."
  Никаких других правок в файле.
- coordination/README.md — добавить в конец раздела про инварианты (или создать такой раздел, если его нет) подсекцию:
  "### Checkpoint-contour grep invariant
   Grep на названия запрещённых технологий (mesenger-интеграции, Telegram и т.п.) применяется к:
   - .github/workflows/**
   - scripts/**
   - .checkpoints/**
   - список secrets репозитория
   Grep НЕ применяется к:
   - coordination/tz/**
   - coordination/reports/**
   Причина: тексты ТЗ и отчётов по природе дела содержат упоминания запрещённых технологий в формулировках запретов. Это не код и не конфигурация, на поведение системы не влияет. Нарушением считается только фактическое появление соответствующей интеграции в коде, конфигурации или secrets."

Локальные gates (6 пунктов):
1. Scope: diff относительно private/main содержит ровно 3 пути из списка выше.
2. tz-002 diff: ровно 2 изменённых строки, обе соответствуют правилу замены выше; слово "Telegram" в файле tz-002 больше не встречается.
3. report-002 diff: только добавление новой секции в конец; существующее содержимое не изменено.
4. coordination/README.md diff: только добавление новой подсекции; существующее содержимое не изменено.
5. Refined grep по контуру checkpoints в соответствии с новым инвариантом:
   - в .github/workflows/**, scripts/**, .checkpoints/**: 0 совпадений по telegram/TELEGRAM_BOT_TOKEN/TELEGRAM_CHAT_ID/api.telegram.org
   - gh secret list: только CHECKPOINTS_PUBLIC_DEPLOY_KEY, никаких Telegram-related
   - coordination/tz/**, coordination/reports/** — НЕ проверяются по новому инварианту
6. markdownlint-cli2 по трём изменённым файлам: 0.

Красный gate — минимальная правка + повтор, максимум 2 цикла, дальше stop.

PR-флоу:
- Branch: infra/cleanup-tz-002-and-invariant от актуального private/main.
- Commit: "infra(coordination): refine checkpoint-contour grep invariant and cleanup tz-002 wording".
- gh pr create --base main, дождаться зелёных автопроверок, squash merge, дождаться success run checkpoint-publisher.yml, сверить public latest.json.commit.

Итоговый отчёт в coordination/reports/report-003.md по контракту + краткое резюме в тред заказчика.

Критические угрозы (только здесь needs_human: yes):
- force-push в main
- правки вне 3 целевых путей
- любые правки кода или конфигурации в .github/workflows/, scripts/, secrets
- правка содержимого tz-002 кроме двух указанных строк
- правка существующего содержимого report-002 (допускается только добавление новой секции в конец)
- затрагивание feat/p3-m1-discovery-and-v1-scaffold или его worktree
- потеря archive/readme-stub (44a5c8c)
- skipped у checkpoint-publisher.yml после штатного squash merge
- более 2 циклов правок gates

Запрещено:
- Добавление любой реальной мессенджер-интеграции (Telegram, Slack, Discord, WhatsApp и любых других) в код, конфигурацию или secrets.
- Правки .github/workflows/, scripts/.
- force-push.
- Удаление слов "Telegram"/"мессенджер" из текстов вне двух целевых строк tz-002.
- Действия в bitrix-bot-mvp, write в public repo напрямую.
