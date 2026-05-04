---
report_for: tz-004
status: success
squash_merge_sha: null
public_latest_before: c5d526c19eafa9ee7346d410957d2ea6b82b4972
public_latest_after: null
gates_passed: 4/4
invariants_ok: true
created_at: 2026-05-04T07:36:00Z
---

# report-004 — invariant refine v2 + report backfill

<!-- markdownlint-disable MD013 -->

## Scope

This report closes `tz-004`: the checkpoint-contour grep invariant was refined
by content nature, `report-003.md` was backfilled, and this final report was
published through the checkpoint workflow.

## PR flow

- A / TZ publication: PR #16, squash `5fcdcbdbe041c049ba7ea8bb2fcec3dc45e0d292`, run `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25306499581`.
- B / invariant refine v2: PR #17, squash `dd80bcc7dd51501c335be5488e8929956f598b34`, run `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25306537068`.
- C / report-003 backfill: PR #18, squash `9c0d824e8eb9a71fd3bbd37cec6f7a0d4feb07f2`, run `https://github.com/saturnkras-maker/saturn-bot-mudriy-ded/actions/runs/25306598746`.
- D / report-004 publication: this file; final squash SHA and public latest are
  resolved after merge by `latest.json` and the final customer-thread summary.

## Gates

- Scope gates: OK for A, B, C, and D; each PR touched only its target coordination file.
- Refined grep: OK by the new content-nature invariant. Code/config checkpoint
  contour and repository secrets remain clean; coordination documents and their
  generated mirrors are not executable integration surface.
- markdownlint-cli2: OK for all changed or created files.
- YAML: `report-003.md` and `report-004.md` parse successfully.

## Public URLs

- report-003: `https://raw.githubusercontent.com/saturnkras-maker/saturn-bot-checkpoints-public/main/coordination/reports/report-003.md`
- report-004: `https://raw.githubusercontent.com/saturnkras-maker/saturn-bot-checkpoints-public/main/coordination/reports/report-004.md`

## Result

Buffer closed. The new invariant applies from the `tz-004` base SHA onward:
`c5d526c19eafa9ee7346d410957d2ea6b82b4972`.
