# SP-<NNN> — <short title>

**Plan:** PLAN-v<N>
**Stage:** S-<NN>
**Scope:** <e.g. alpha, backend, service-a>
**Prev:** SP-<NNN-1>   <!-- omit if first SP -->
**Status:** in-progress | done | superseded
**Opened-at:** YYYY-MM-DD
**Closed-at:** YYYY-MM-DD   <!-- fill on close -->
**Closed-by:** SP-<NNN>   <!-- only if Status=superseded -->

---

## Context for the next agent

What the previous agent learned, decided, ran into. What's surprising. What's *not* a problem so the reader doesn't re-investigate. 5-15 lines.

## Goal of this session

One executable goal. One lens. If multiple goals are needed → split into multiple SPs.

## Inputs

- `plans/PLAN-v<N>.md` § S-<NN>
- `artifacts/SP-<NNN-1>/...` (if any)
- `workspace/raw/...`
- `workspace/blockers/agent-inbox/...` (if any)

## Steps

1. ...
2. ...

## Verify block (required)

End of session must produce:
- Total items processed: N
- Counts by bucket / class / label: ...
- 3 random samples quoted by ID
- Apply-log artifact `A-NN-apply-log.md`

## Outputs

- `artifacts/SP-<NNN>/A-01-<what>.md`
- `artifacts/SP-<NNN>/A-02-apply-log.md`

## Handoff to next agent

- Suggested next SP: ...
- Open blockers raised: ...
- Pitfalls to warn about: ...
