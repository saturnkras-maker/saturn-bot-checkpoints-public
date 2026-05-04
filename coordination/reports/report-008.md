---
report_for: tz-008
status: success
squash_merge_sha: 1f597b0ddf2e1eb0331af44f57a132658bb18b3f
public_latest_before: 1f597b0ddf2e
public_latest_after: null
gates_passed: 6/6
invariants_ok: true
created_at: 2026-05-04T12:25:00Z
---

# Report 008 — coordination hardening

<!-- markdownlint-disable MD013 -->

## Summary

TZ-008 track 1 is completed successfully. `coordination/README.md` now has a
new section `Правила PR в этом репозитории` with 10 explicit PR and invariant
rules. Track 2 is not started and must wait for tz-009.

## PR flow

- PR #31 — `Publish tz-008 coordination hardening [checkpoint]`.
  - Squash SHA: `b2586d0f7f154bd73fbd382956da1c324f3ca31f`.
  - Run: <https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25318448775> — success.
- PR #32 — `Document coordination PR invariants [checkpoint]`.
  - Squash SHA: `1f597b0ddf2e1eb0331af44f57a132658bb18b3f`.
  - Run: <https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25318522590> — success.
- Report-008 publication is this PR.

## Rules fixed in README

1. Checkpoint marker in squash title — `[checkpoint]` is mandatory for PRs that
   must publish checkpoints; PR #27 is recorded as the real skipped-run example.
2. Grep invariant v2 — forbidden technology grep is limited to code,
   configuration, and secrets, with a cross-reference to the existing invariant.
3. Checkpoint workflow loop guard — publication commits must not retrigger the
   workflow loop.
4. Front2 scope roots — default writable roots are `prompts/`,
   `content_samples/`, `tests/anonymization/`, and `coordination/`.
5. Force-push rule — `main` force-push is forbidden; feature branch rewrites, if
   needed, must use `--force-with-lease`.
6. Roles whitelist — `content_samples/` allows only `сотрудник`, `руководитель`,
   `коллега`.
7. Zone and fallback subtype distribution — `3/3/2/2` is fixed for main content
   zones and fallback subtypes.
8. Line-count corridors — type corridors are fixed and are not autonomously
   expanded.
9. TBD fields in context schema — `signals[].source_detector`,
   `actions_deadline_label`, and `confidence_level` are not closed by front2.
10. Public repo publication — public repo is updated only through checkpoint
    workflow and allowed published surfaces; direct public writes are forbidden.

## Gates 6/6

1. Scope: PASS — content PR changed exactly `coordination/README.md`.
2. Existing README sections preserved: PASS — new section appended without
   rewriting existing sections.
3. Rule count: PASS — 10 rules, above the required minimum of 8.
4. Rule structure: PASS — each rule has formulation, violation example, and
   consequence.
5. `git diff --check`: PASS.
6. `markdownlint-cli2 coordination/README.md`: PASS, 0 errors.

## Invariants

- Scope roots respected: only `coordination/` was changed in this track.
- `prompts/`, `content_samples/`, and `tests/anonymization/` were not changed.
- Workflows, scripts, src, secrets, deploy keys, and branch protection were not
  changed.
- Existing `coordination/tz`, `coordination/reports`, and audit history files
  were not edited; only new tz/report files were added.
- `bitrix-bot-mvp` was not touched.
- Track 2 was not started.

## Next step

Wait for tz-009 — pilot expansion + phase closure. Do not start it without a new
TZ from the coordinator.
