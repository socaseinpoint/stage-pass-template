# Scenario: Codebase audit

Abstract walkthrough. Names are illustrative. Apply the structure to your real project.

## Setting

Team needs to audit a 4-service backend (`service-a`, `service-b`, `service-c`, `service-d`) for security, dead code, and outdated dependencies. Estimated 8-12 sessions across 2 weeks. Two reviewers, one agent.

## PLAN-v1

```markdown
# PLAN-v1

**Version:** 1
**Frozen-at:** 2026-05-12

## Project goal

Produce per-service audit reports covering security findings, dead-code candidates,
and dependency-update recommendations for service-a..service-d.

## Constraints

- Deadline: 2026-05-26
- Out of scope: actual fixes, only findings + recommendations

## Stages

### S-01 — Inventory
Goal: per-service file/module inventory + dependency graph.
Exit criteria: one inventory artifact per service.
Estimated sessions: 4.

### S-02 — Security pass
Goal: classify each finding as severity-high/med/low.
Exit criteria: per-service security artifact, frozen `severity-*` family.

### S-03 — Dead-code pass
Goal: candidate list with confidence-high/med/low.
Exit criteria: per-service dead-code artifact.

### S-04 — Dep-update pass
Goal: outdated deps with risk-high/med/low.
Exit criteria: per-service dep artifact.

### S-05 — Consolidation
Goal: cross-service report.
Exit criteria: one summary artifact.
```

## Session 1 — SP-001

Agent reads PLAN-v1 § S-01, picks `service-a`, runs inventory.

```markdown
# SP-001 — inventory service-a

**Plan:** PLAN-v1
**Stage:** S-01
**Scope:** service-a
**Status:** done
**Opened-at:** 2026-05-12
**Closed-at:** 2026-05-12

## Context for the next agent
First session. Inventory tool is `cloc` + manual module listing. Service-a uses 3 frameworks.

## Goal of this session
Produce inventory of service-a: files, modules, external deps, public surface.

## Verify block
Items: 142 files, 23 modules, 47 deps.
Samples (3): `auth/login.ts`, `billing/invoice.ts`, `shared/db.ts`.

## Outputs
- artifacts/SP-001/A-01-inventory.md
- artifacts/SP-001/A-02-apply-log.md

## Handoff to next agent
Next SP: inventory service-b (S-01 continues). cloc command in apply-log.
```

Artifacts:
```
artifacts/SP-001/
  A-01-inventory.md       # the actual inventory table
  A-02-apply-log.md       # what commands were run, with output samples
```

Commit: `[SP-001] close: inventory service-a`

## Session 2 — SP-002

Agent reads `plans/PLAN-current.md`, last SP (SP-001), runs same lens on service-b.

```markdown
# SP-002 — inventory service-b
**Prev:** SP-001
**Scope:** service-b
**Status:** done
... (same shape as SP-001)
```

Commit: `[SP-002] close: inventory service-b`

## Session 5 — SP-005 (S-02 begins)

After SP-001..SP-004 covered all four services' inventories, S-01 exits.

```markdown
# SP-005 — security pass service-a

**Plan:** PLAN-v1
**Stage:** S-02
**Scope:** service-a
**Prev:** SP-004
**Status:** done

## Context for the next agent
S-01 done for all 4 services. Inventories in artifacts/SP-001..SP-004.
Tool for this lens: `semgrep` with ruleset X.

## Verify block
Findings: 23 (12 high, 8 med, 3 low).
Samples (3): F-001 (auth bypass), F-007 (sql-fragment), F-019 (open redirect).
```

`severity-*` family freezes when S-02 closes globally (i.e., all four services audited).

## Things this scenario demonstrates

- Sequential SPs across multiple scopes (service-a..service-d) without scope-encoded IDs.
- S-NN advances when all scopes exit, not per scope.
- Lens stays single throughout a pass (inventory ≠ security ≠ dead-code).
- Freezing severity-* lets S-05 (consolidation) trust S-02's classification.
