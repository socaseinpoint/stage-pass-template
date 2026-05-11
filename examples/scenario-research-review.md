# Scenario: Research review

Abstract walkthrough. Names are illustrative.

## Setting

Solo researcher reviewing ~80 papers on Topic X. Goal: a synthesis report. Sessions are short (1-2h), spread across 6 weeks. Memory between sessions is fragile.

## PLAN-v1 (abbreviated)

```markdown
## S-01 — Source collection
## S-02 — Per-paper extraction (key findings, methods, limitations)
## S-03 — Cross-paper synthesis (themes, contradictions)
## S-04 — Synthesis report draft
```

## Workspace policy

`workspace/raw/papers/` — **external** policy. PDFs live in `~/research/topic-x/papers/`; `workspace/README.md` records the path. Citations in artifacts reference by paper ID (`P-013`) not by file path.

## Session 5 — SP-005 (mid-S-02)

```markdown
# SP-005 — extraction batch P-016..P-020

**Plan:** PLAN-v1
**Stage:** S-02
**Scope:** batch-4
**Prev:** SP-004
**Status:** done

## Context for the next agent
SP-002..SP-004 covered P-001..P-015. Extraction template stable since SP-002.
Note: P-017 is a survey paper — extract themes, skip methods.

## Goal of this session
Produce one extraction artifact per paper P-016..P-020.

> Note: `## Inputs` and `## Steps` sections omitted for brevity — they are required in real SPs per `stage-plan/_template.md`.

## Verify block
Papers extracted: 5/5.
Samples (3, quoted):
- P-016 § findings: "RAG outperforms baseline by 14% on Topic-X recall, 95% CI [12,16]"
- P-018 § methods: "Authors used a 5-shot prompt with retrieval over a 12k-document corpus"
- P-020 § limitations: "Generalization to languages other than English untested"

## Outputs
- artifacts/SP-005/A-01-P016.md
- artifacts/SP-005/A-02-P017.md
- artifacts/SP-005/A-03-P018.md
- artifacts/SP-005/A-04-P019.md
- artifacts/SP-005/A-05-P020.md
- artifacts/SP-005/A-06-apply-log.md

## Handoff to next agent
Next: SP-006 for P-021..P-025. Same template.
Open question for synthesis (S-03): P-014 and P-019 disagree on definition of Term Y — flagged in workspace/blockers/domain/.
```

## Things this scenario demonstrates

- **Long-running, fragile-memory** workflow — each SP rebuilds context in 5-15 lines.
- One artifact per paper (per item), batched under one SP per session.
- `workspace/raw/` external policy — heavy binaries (PDFs) live outside the repo, referenced by ID.
- Domain blocker raised mid-stage (`P-014 vs P-019 on Term Y`) gets parked in `workspace/blockers/domain/` for resolution during S-03.
- No agent ≠ no framework. The protocol works for a solo human too — agent-inbox becomes self-notes.
