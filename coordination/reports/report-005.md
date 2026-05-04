---
report_for: tz-005
status: success
squash_merge_sha: 53fcf94fadc10df2dc1274af0f40ccd1092dd812
public_latest_before: d044a4d73bfa
public_latest_after: null
gates_passed: 8/8
invariants_ok: true
created_at: 2026-05-04T07:55:00Z
---

# report-005 — anonymization pairs batch 2

<!-- markdownlint-disable MD013 -->

## PR flow

- TZ publication branch: `infra/tz-005-push`.
- TZ publication commit: `7ea665e7697fbb75f39acf17420f64c3861beaa8`.
- TZ publication PR: `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/pull/20`.
- TZ publication squash SHA: `d044a4d73bfa3795a798af14a2fa6762ff46313a`.
- TZ publication run: `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25307391907`.

- Batch branch: `feat/p3-m1-anonymization-pairs-batch-2`.
- Batch base SHA: `31713b5291fd3263b017f6cb4c3859142c15a719`.
- Batch commit SHA: `96917706b801df1a078a98317f31bd576e90ac81`.
- Batch PR: `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/pull/21`.
- Batch squash merge SHA: `53fcf94fadc10df2dc1274af0f40ccd1092dd812`.
- Batch checkpoint run: `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25307519734`.
- Public latest before batch: `d044a4d73bfa`.
- Public latest after batch: `53fcf94fadc1`.

## Gates 8/8

1. Scope: OK — 11 target paths only.
2. Pair integrity: OK — pair-06..pair-10 each have raw and anonymized files;
   raw `markers_count` equals anonymized `markers_replaced`.
3. Stopwords grep: OK — anonymized files have 0 stopword hits.
4. Placeholder whitelist: OK — only `[задача]`, `[значение]`, `[диапазон]`,
   `[проект]`, and allowed roles are used.
5. Source-type coverage: OK — batch-2 adds one pair each for morning, evening,
   friday, monthly, and quarterly; together with batch-1 this gives 2 pairs per
   source_type and 10/10 total pairs.
6. Line-count: OK — all raw/anonymized variants have 6 non-empty output lines.
7. `git diff --check`: OK.
8. `markdownlint-cli2`: OK after one minimal markdownlint header fix in
   `tests/anonymization/length_report.txt`.

## Safety invariants

No workflow, script, prompt, content sample, secret, deploy-key, branch
protection, public repo direct write, or real messenger integration was changed.
Raw samples contain only synthetic markers and no real personal data.

## Next

Ready to start audit-PR второго фронта on command.
