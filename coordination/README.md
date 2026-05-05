# coordination

<!-- markdownlint-disable MD013 -->

`coordination/` — канал обмена постановками и отчётами между координатором и ботом.

## Структура

- `coordination/tz/` — постановки задач от координатора.
- `coordination/reports/` — отчёты бота по выполненным постановкам.

## Имена файлов

- Постановка: `tz-NNN-<slug>.md`, где `NNN` — трёхзначный счётчик от `001`.
- Отчёт: `report-NNN.md`, где `NNN` совпадает с номером постановки.

## Жизненный цикл

1. Координатор добавляет постановку в `coordination/tz/` со статусом `draft` или `active`.
2. Бот выполняет работу по актуальной постановке.
3. Бот добавляет отчёт в `coordination/reports/`.
4. После закрытия постановки статус становится `done`; заменённые постановки получают `superseded`.

## Правила

- YAML-шапка обязательна для каждой постановки и каждого отчёта.
- Все пути в `scope_files` должны быть явно перечислены.
- В отчётах фиксируются SHA, PR, gates, инварианты и следующий шаг.
- Служебные секреты, ключи и приватные идентификаторы в `coordination/` не добавляются.


## Канонический URL для Perplexity agent fetch

Public checkpoint mirror доступен не только через GitHub raw/api, но и через GitHub Pages.

Canonical agent-readable endpoint для Perplexity Spaces / Agent Mode:

- `https://saturnkras-maker.github.io/saturn-bot-checkpoints-public/latest.md`
- `https://saturnkras-maker.github.io/saturn-bot-checkpoints-public/latest.json`
- `https://saturnkras-maker.github.io/saturn-bot-checkpoints-public/index.json`

Raw GitHub endpoints (`raw.githubusercontent.com`, `github.com/.../raw`, `api.github.com`) считаются fallback, потому что anonymous `fetch_url` в Perplexity может блокироваться на GitHub/raw/api/CDN даже при публичном репозитории.

Handshake line для координатора после проверки:

```text
AGENT-FETCH-READY: url=https://saturnkras-maker.github.io/saturn-bot-checkpoints-public/latest.md verified_at=<ISO> latest.json.commit_sha=<sha>
```

## Invariants

### Checkpoint-contour grep invariant

Grep на названия запрещённых технологий (мессенджер-интеграции, в т.ч.
Telegram, Slack, Discord, WhatsApp и т.п.) проверяет **только код и
конфигурацию**, где появление слова-маркера означало бы реальное
использование технологии:

- `.github/workflows/**`
- `scripts/**`
- `src/**` (части, относящиеся к контуру публикации checkpoints)
- имена secrets репозитория (`gh secret list`)

Grep **не применяется** к документам и их зеркалам:

- `coordination/tz/**`
- `coordination/reports/**`
- `.checkpoints/coordination/**` (автоматические реплики `coordination/`)
- `.checkpoints/latest.md`, `.checkpoints/index.json`, `.checkpoints/history/**`
  в части, цитирующей тексты ТЗ и отчётов
- любые будущие поддиректории `coordination/`, добавленные через расширение whitelist

Причина: в текстах документов упоминание запрещённой технологии может быть
частью формулировки запрета, исторической справки или обсуждения. Это не код,
на поведение системы не влияет. Нарушением считается только фактическое
подключение соответствующей интеграции в коде, конфигурации или secrets.

Практическая процедура проверки: при каждом итоговом отчёте выполнять grep по
списку "проверяем", ожидать 0 совпадений. Grep по списку "не проверяем" не
выполняется.

## Правила PR в этом репозитории

### 1. Маркер checkpoint в squash title

Правило: каждый PR, который должен опубликовать checkpoint, мержится squash
commit title с маркером `[checkpoint]`. Перед merge нужно проверить title.

Пример нарушения: PR #27 в tz-007 был смёржен без `[checkpoint]`, поэтому
`checkpoint-publisher.yml` завершился как skipped; это было исправлено PR #28.

Последствие: skipped run; stop и `needs_human: yes` до корректирующего PR.

### 2. Grep-инвариант v2 по природе контента

Правило: grep на запрещённые технологии применяется только к коду,
конфигурации и secrets. Документы `coordination/` и их `.checkpoints/`-зеркала
не проверяются; см. раздел `Checkpoint-contour grep invariant` выше.

Пример нарушения: гипотетический — считать упоминание Telegram в тексте ТЗ
нарушением кода и блокировать PR документации.

Последствие: red gate до повторной проверки по корректному scope.

### 3. Loop guard у checkpoint-publisher.yml

Правило: workflow публикации checkpoint не должен запускаться на собственные
коммиты `chore(checkpoint): ...`. Эти коммиты являются результатом публикации,
а не новым пользовательским PR.

Пример нарушения: гипотетический — workflow повторно запускается на
`chore(checkpoint): update latest snapshot` и начинает цикл публикаций.

Последствие: needs_human: yes; red gate для дальнейших merge до диагностики.

### 4. Scope roots фронт2

Правило: фазы фронт2 по умолчанию работают только в `prompts/`,
`content_samples/`, `tests/anonymization/` и `coordination/`. Любой другой root
считается read-only, если новое ТЗ явно не расширило scope.

Пример нарушения: гипотетический — в фазе контент-слоя одновременно правится
`scripts/` или `.github/workflows/` без отдельного разрешения.

Последствие: needs_human: yes; stop до явного расширения ТЗ.

### 5. Force-push

Правило: на feature-ветках допустим только `--force-with-lease`, если он вообще
нужен. На `main` force-push запрещён в любой форме.

Пример нарушения: гипотетический — переписать `main` через `git push --force`
после неудачного checkpoint.

Последствие: критическая угроза с rollback/восстановлением под контролем
человека.

### 6. Roles whitelist для контента

Правило: в `content_samples/` допустимы только роли `сотрудник`,
`руководитель`, `коллега`. Реальные имена и роли вне whitelist не допускаются.

Пример нарушения: tz-007 нормализовал старые morning samples, где встречались
роли вне whitelist вроде `менеджер проекта` и `специалист продаж`.

Последствие: red gate; PR не закрывается до нормализации ролей.

### 7. Распределение зон и fallback-подтипов

Правило: для `morning`, `evening`, `friday`, `monthly`, `quarterly` распределение
зон в `content_samples/<type>/` — `green/white/yellow/red = 3/3/2/2`. Для
`fallback` распределение подтипов —
`no_data/partial_signals/detector_error/input_anomaly = 3/3/2/2`.

Пример нарушения: гипотетический — добавить одиннадцатый `yellow` sample без
компенсации распределения.

Последствие: red gate до восстановления распределения.

### 8. Line-count corridors

Правило: коридоры строк не расширяются автономно: morning 3-4, evening 3-4,
friday 5-7, monthly 6-8, quarterly 8-10, fallback 2-4. Допуск: ±1 строка.

Пример нарушения: гипотетический — добавить quarterly sample на 13 строк и
объявить новый коридор без отдельного ТЗ.

Последствие: red gate; изменение коридора требует нового явного решения.

### 9. TBD-поля context_schema.md

Правило: `signals[].source_detector`, `actions_deadline_label` и
`confidence_level` закрываются только основным фронтом или отдельным ТЗ. Фронт2
не закрывает эти поля самостоятельно.

Пример нарушения: гипотетический — в контентной фазе заменить `TBD` описанием
поля без сигнала от основного фронта.

Последствие: stop и `needs_human: yes` до решения координатора.

### 10. Публикация public repo

Правило: public repo обновляется только через whitelist checkpoint workflow и
опубликованные `coordination/`/`.checkpoints/` поверхности. Direct write в
public repo запрещён.

Пример нарушения: гипотетический — вручную запушить `coordination/reports/` в
public repo в обход `checkpoint-publisher.yml`.

Последствие: критическая угроза; stop и разбор публикационного контура.
