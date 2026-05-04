---
audit_id: second-front-2026-05-04
status: clean
base_sha: ca84137944f8e60c255bef6d9e116cd9a4ac1093
checks_passed: 9/9
findings: []
created_at: 2026-05-04T10:04:00Z
---

# Second front audit — 2026-05-04

<!-- markdownlint-disable MD013 -->

## 1. Content-layer scope coverage

- Goal: verify all second-front prompt and sample surfaces exist.
- Method: `ls prompts/`; count `content_samples/<type>/*.md`.
- Result: PASS.
- Comment: six prompt files exist; each sample directory has 10 files:
  morning, evening, friday, monthly, quarterly, fallback.

## 2. Line-count corridors

- Goal: verify sample body/output line counts against corridors fixed in prompts.
- Method: parse non-empty body lines in `content_samples/<type>/*.md` and compare
  with each `prompts/<type>.md` volume rule.
- Result: PASS.
- Comment: prompt corridors are morning 3-4, evening 3-4, friday 5-7,
  monthly 6-8, quarterly 8-10, fallback 2-4; all samples match with tolerance.

## 3. Distribution by zones and fallback subtypes

- Goal: verify expected distribution across samples.
- Method: parse sample filenames by zone/subtype prefix.
- Result: PASS.
- Comment: morning/evening/friday/monthly/quarterly are 3 green, 3 white,
  2 yellow, 2 red. Fallback is 3 no_data, 3 partial_signals,
  2 detector_error, 2 input_anomaly.

## 4. Roles whitelist and real-name grep

- Goal: verify only allowed roles are used and no real-name-like strings appear.
- Method: YAML role scan plus Cyrillic surname + two initials regex over
  `content_samples/`.
- Result: PASS.
- Comment: roles stay within сотрудник, руководитель, коллега; real-name hits: 0.

## 5. Stopwords grep

- Goal: verify stopwords are absent from content samples and anonymized variants.
- Method: substring scan using `tests/anonymization/stopwords.txt` over
  `content_samples/`, `tests/anonymization/pairs/batch-1/*-anonymized.md`, and
  `tests/anonymization/pairs/batch-2/*-anonymized.md`.
- Result: PASS.
- Comment: stopword hits: 0.

## 6. Anonymization pairs integrity

- Goal: verify cross-batch anonymization pair consistency.
- Method: parse YAML headers and placeholders in pair-01..pair-10.
- Result: PASS.
- Comment: 10/10 pairs exist; each pair has raw and anonymized files;
  `markers_count == markers_replaced`; placeholder whitelist is respected;
  coverage is 2 pairs per source_type.

## 7. length_report.txt structural integrity

- Goal: verify length report contains all required sections once.
- Method: count expected section headings in `tests/anonymization/length_report.txt`.
- Result: PASS.
- Comment: 8 unique sections exist exactly once: morning, evening, friday,
  monthly, quarterly, fallback, anonymization_pairs_batch_1,
  anonymization_pairs_batch_2.

## 8. context_schema.md TBD delta

- Goal: verify second-front work did not close shared TBD fields prematurely.
- Method: `git diff 8d6404df465e..HEAD -- prompts/_shared/context_schema.md`.
- Result: PASS.
- Comment: diff is empty; `signals[].source_detector`,
  `actions_deadline_label`, and `confidence_level` remain TBD.

## 9. Safety invariants cross-check

- Goal: verify protected branches, checkpoint contour, secrets, and public surface.
- Method: `git ls-remote` for protected refs; refined grep over
  `.github/workflows/` and `scripts/`; `gh secret list`; public repo root listing.
- Result: PASS.
- Comment: `archive/readme-stub` is at
  `44a5c8c2838adaa900d42b91c08218ad8c64d742`; scaffold branch is at
  `eb4f82edeb902402b1011d27598d2ef384409429`; workflow/scripts grep has 0 hits;
  secrets list contains only `CHECKPOINTS_PUBLIC_DEPLOY_KEY`; public repo has the
  expected published surfaces and checkpoint index/history files; no direct
  public write or messenger integration was introduced.

## Summary

Audit status: clean. Findings: none.
