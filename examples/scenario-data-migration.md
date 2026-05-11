# Scenario: Data migration

Abstract walkthrough. Names are illustrative.

## Setting

Migrating 18,000 customer records from System Old to System New. Field semantics differ; some New fields have no Old equivalent and need synthesis. Two passes per record minimum.

## PLAN-v1 (abbreviated)

```markdown
## S-01 — Field-mapping draft
## S-02 — Auto-mappable records (high-confidence) — direct transform
## S-03 — Ambiguous records (med-confidence) — agent triage with human spot-check
## S-04 — No-equivalent records (low-confidence) — manual rules
## S-05 — Reconciliation + load
```

## Session 3 — SP-003 (S-02 starts)

```markdown
# SP-003 — auto-mappable batch 1

**Stage:** S-02
**Scope:** batch-1 (records 1-6000)
**Status:** done

## Goal of this session
Transform batch-1 high-confidence records to New schema. Reject any record where any field flips confidence to <high.

> Note: `## Inputs` and `## Steps` sections omitted for brevity — they are required in real SPs per `stage-plan/_template.md`.

## Verify block
Input: 6000. Transformed: 5876. Rejected for re-triage: 124.
Samples (3, quoted):
- R-00041: `{old_id: "X41", new_id: "U41", status: "active", migrated_at: "..."}` — clean
- R-00302: `{old_id: "X302", new_id: "U302", status: "active", migrated_at: "..."}` — clean
- R-04711: `{old_id: "X4711", new_id: "U4711", status: "active", migrated_at: "..."}` — clean

## Outputs
- artifacts/SP-003/A-01-transformed.jsonl (linked, not inline)
- artifacts/SP-003/A-02-rejected-for-triage.md (the 124 IDs)
- artifacts/SP-003/A-03-apply-log.md
```

## Workspace policy here

This project uses **mixed policy**:
- `workspace/raw/` (the 18k records) — **ignored** in git. Pulled from System Old via script; size + PII reasons.
- `workspace/blockers/` — tracked.
- `artifacts/SP-003/A-01-transformed.jsonl` — also ignored (PII); only metadata + verify block tracked.

`workspace/README.md` records:
```
Source: System Old export, dump-2026-05-10.jsonl
Location: ~/migrations/dumps/system-old/dump-2026-05-10.jsonl
Pull command: pull-from-system-old.sh
Records: 18000
```

## Things this scenario demonstrates

- Scope-as-batch (`batch-1`, `batch-2`) when the corpus is too large for one pass.
- Cross-stage handoff: SP-003 (S-02) hands rejects to SP-007 (S-03) — IDs only, no hyperlink.
- Workspace policy **per sub-path**: raw + transformed ignored, blockers + apply logs tracked.
- Verify block reconciles input/output/rejected counts — fan-out bug catcher.
