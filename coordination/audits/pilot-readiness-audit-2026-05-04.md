---
audit_id: pilot-readiness-2026-05-04
status: clean
base_sha: 3a9e681f2ee02d3d5d462a6cac073e23b11b6cb6
checks_passed: 10/10
findings: []
created_at: 2026-05-04T11:45:00Z
---

# Pilot readiness audit — 2026-05-04

<!-- markdownlint-disable MD013 -->

## 1. Safety rules coverage

- Goal: verify shared safety rules exist and are referenced by all 6 prompts.
- Method: check `prompts/_shared/safety_rules.md` and grep each prompt for
  `prompts/_shared/safety_rules.md`.
- Result: PASS.
- Comment: shared refusal topics cover salary, sick leave, conflicts, personal,
  medical, and political content; morning, evening, friday, monthly, quarterly,
  and fallback all reference the shared file.

## 2. Pilot package coverage

- Goal: verify director pilot samples exist for each required contact type.
- Method: count and parse `content_samples/pilot/*.md`.
- Result: PASS.
- Comment: 5/5 pilot files exist: morning, evening, friday, monthly, quarterly.
  All use `employee_role: руководитель`; fallback is intentionally absent.

## 3. Content sample count and line corridors

- Goal: verify 60 existing samples plus 5 pilot samples stay within corridors.
- Method: parse YAML `type` and count non-empty output lines in
  `content_samples/*/*.md`.
- Result: PASS.
- Comment: total content samples: 65. Corridors pass with tolerance: morning 3-4,
  evening 3-4, friday 5-7, monthly 6-8, quarterly 8-10, fallback 2-4.

## 4. Roles whitelist and real-name grep

- Goal: verify allowed role vocabulary and absence of real-name-like strings.
- Method: YAML role scan plus Cyrillic surname + initials regex over
  `content_samples/`.
- Result: PASS.
- Comment: roles stay within `сотрудник`, `руководитель`, `коллега`;
  real-name hits: 0.

## 5. Expanded stopwords grep

- Goal: verify generated-text disclosure and existing stopwords are absent.
- Method: substring scan using `tests/anonymization/stopwords.txt` over all 65
  content samples and all 10 anonymization pairs.
- Result: PASS.
- Comment: stopword hits: 0, including AI-disclosure variants such as
  `как языковая модель`, `я ИИ`, `как ассистент`, and `я нейросеть`.

## 6. Anonymization pairs regression

- Goal: verify anonymization pair corpus remains available for regression.
- Method: enumerate `tests/anonymization/pairs/batch-1` and `batch-2`.
- Result: PASS.
- Comment: 10/10 pairs exist, each with raw and anonymized variants.

## 7. Markdown lint gate

- Goal: verify touched and linked markdown surfaces are lint-clean.
- Method: `npx --yes markdownlint-cli2@latest` over touched prompts, shared
  safety rules, modified morning samples, pilot samples, anonymization pairs, and
  `tests/anonymization/length_report.txt`.
- Result: PASS.
- Comment: markdownlint summary: 0 errors.

## 8. context_schema.md TBD status

- Goal: verify shared TBD fields were not closed by this phase.
- Method: inspect `prompts/_shared/context_schema.md` for
  `signals[].source_detector`, `actions_deadline_label`, and `confidence_level`.
- Result: PASS.
- Comment: all 3 fields remain TBD. Synchronization with closed TBD fields is
  transferred to a future phase; this is expected by tz-007 and does not block
  the current phase.

## 9. Scope and protected surfaces

- Goal: verify tz-007 stayed inside allowed roots and protected refs are intact.
- Method: diff from report-006 public commit to HEAD and `git ls-remote` for
  protected branches.
- Result: PASS.
- Comment: changed roots are limited to `prompts/`, `content_samples/`,
  `tests/anonymization/`, `coordination/`, and generated `.checkpoints/`.
  `archive/readme-stub` remains at `44a5c8c2838adaa900d42b91c08218ad8c64d742`;
  scaffold branch remains at `eb4f82edeb902402b1011d27598d2ef384409429`.

## 10. Checkpoint/public contour

- Goal: verify corrective checkpoint restored publication after the skipped run.
- Method: inspect workflow runs and public `latest.json`.
- Result: PASS.
- Comment: PR #27 produced skipped run `25315042018`; corrective PR #28 produced
  success run `25316610333`. Public latest points to commit
  `b7bed391a1d6` with message `Republish tz-007 pilot readiness [checkpoint]`.
  Checkpoint-contour grep over `.github/workflows/` and `scripts/` has 0
  messenger-integration hits; secrets list contains only
  `CHECKPOINTS_PUBLIC_DEPLOY_KEY`.

## Summary

Audit status: clean. Findings: none.
