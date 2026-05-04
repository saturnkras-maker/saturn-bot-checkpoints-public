# coordination/reports

<!-- markdownlint-disable MD013 -->

Папка для отчётов бота по постановкам из `coordination/tz/`.

## Имя файла

`report-NNN.md`, где `NNN` совпадает с номером постановки.

## YAML-шапка

```yaml
report_for: tz-NNN
status: success | stop | partial
squash_merge_sha: <sha> | null
public_latest_before: <sha>
public_latest_after: <sha>
gates_passed: <passed>/<total>
invariants_ok: true | false
created_at: YYYY-MM-DDTHH:MM:SSZ
```

## Тело

Тело отчёта фиксирует base SHA, branch, PR, run URLs, gates, инварианты и следующий шаг.
