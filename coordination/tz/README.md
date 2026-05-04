# coordination/tz

Папка для постановок задач от координатора.

## Имя файла

`tz-NNN-<slug>.md`, где `NNN` — трёхзначный счётчик от `001`.

## YAML-шапка

```yaml
tz_id: tz-NNN
status: draft | active | superseded | done
created_at: YYYY-MM-DDTHH:MM:SSZ
base_sha: <private-main-sha>
scope_files:
  - path/to/file.md
depends_on: tz-NNN | null
```

## Тело

После YAML-шапки идёт постановка в свободной форме. Рекомендуется держать структуру:
цель, scope, последовательность, gates, stop-критерии, итоговый отчёт.
