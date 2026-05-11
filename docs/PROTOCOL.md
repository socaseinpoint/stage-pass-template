# Stage-Pass Protocol

This document is the framework specification. For rationale and antipatterns, see `WHY.md`.

## Core idea

State lives in files. Chat is the live working layer inside one session. The next session — whether your future self, another agent, or a colleague — cold-resumes by reading files only.

## The four layers

| Layer       | File(s)                              | Mutates       | Audience            |
|-------------|--------------------------------------|---------------|---------------------|
| Plan        | `plans/PLAN-vN.md`                   | Rarely (new version) | Strategic — humans + agents |
| Stage-plan  | `stage-plan/SP-NNN-<what>.md`        | Once (closed once) | The next agent in the chain |
| Artifact    | `artifacts/SP-NNN/A-NN-<what>.md`    | Once (per pass) | Verifiers, downstream agents |
| Workspace   | `workspace/raw/`, `workspace/blockers/` | Per policy | All |

## ID convention

- `PLAN-vN` — plan version. Immutable once written.
- `S-NN` — stage inside a plan. Numbered fixed by the plan version.
- `SP-NNN` — stage-plan. One session handoff. Global sequential counter across the whole project.
- `A-NN` — artifact. Counter resets per parent SP.

Cross-refs are explicit by ID. No hyperlinks from PLAN to stage-plans — coupling only via ID convention. This avoids link rot when stages split.

## Session lifecycle

```
Agent A opens session
  → read README, plans/PLAN-current.md, last SP, .planning/HANDOFF.json, agent-inbox blockers
  → execute the goal of one SP
  → write artifacts under artifacts/SP-NNN/
  → close current SP (Status: done)
  → write next SP with context + goal for agent B
  → update .planning/HANDOFF.json
  → commit
Agent B opens session — same loop.
```

## File schemas

### PLAN-vN.md

Header: version, frozen-at date, supersedes prev version, 1-3 line diff. Body: project goal, constraints, stages (S-NN with goal + exit criteria + status), risks, glossary.

### SP-NNN-<what>.md

Header: plan ref, stage ref, scope, prev SP, status, opened/closed dates, closed-by (if superseded). Body: context for next agent, goal of this session, inputs, steps, verify block, outputs, handoff notes.

### A-NN-<what>.md

Header: parent SP, lens, time-box, status, date. Body: method, result, verify block (counts + 3 sample IDs), notes for next pass.

### .planning/HANDOFF.json

Single-file snapshot of last-session state. Updated by each closing agent.

```json
{
  "schema_version": 1,
  "current_plan": "PLAN-vN",
  "last_session": {"sp_id": "SP-NNN", "closed_at": "YYYY-MM-DD", "scope": "..."},
  "next_session": {"expected_sp_id": "SP-NNN+1", "starting_stage": "S-NN", "scope": "...", "notes": "..."},
  "open_blockers": {"domain": [...], "agent_inbox": [...]}
}
```

## Verify block

Required at the end of every artifact that processes a set:
- Total items
- Counts by bucket / class / label
- 3 random samples quoted by ID (so a reader can spot-check inline)

Verify blocks catch fan-out bugs early — if the counts don't reconcile across passes, something silently went wrong.

## Per-pass discipline

1. **One pass = one lens.** Mixing lenses (e.g. dedup + classification) hides errors and produces unreviewable diffs.
2. **Hard time-box.** Pass overflow → next pass. Never extend a pass to "finish".
3. **Apply-log artifact** every pass: what changed, where, sample evidence.
4. **Verify block** every pass.

## Frozen state families

A family is a set of labels/statuses that, once assigned in a given stage, cannot be relabeled without an explicit apply-log decision entry. CLAUDE.md declares which families freeze after which stage. Freezing is what makes downstream passes trust the upstream classification.

When relabeling is required, record the rationale in the closing SP's apply-log artifact (`A-NN-apply-log.md`) as an apply-log decision entry. This is the framework's decision record — no separate file.

## Workspace storage policies

| Mode       | What's in `workspace/`            | Git treatment                                   |
|------------|-----------------------------------|-------------------------------------------------|
| `tracked`  | Small raw, design docs, screenshots | Committed normally                             |
| `external` | Heavy data, corp dumps, video     | Path ignored; `workspace/README.md` records external location |
| `ignored`  | Secrets, sensitive dumps          | Path in `.gitignore`; shared out-of-band       |

Mix per sub-path is allowed and common.

## Commit discipline

Each closed SP = one commit. Each artifact batch within a session = one commit. Plan bumps = one commit with diff in body. `git log --oneline` should read as a handoff timeline a new agent can scroll.

## What this protocol does NOT do

- Does not replace chat. Chat is live state within a session.
- Does not enforce timestamps inside files beyond schema headers — git history is the timeline.
- Does not specify domain. Labels, sources, gotchas are per-project in `CLAUDE.md`.
- Does not prescribe tooling. Slash commands, hooks, generators are optional add-ons after the bare protocol is used at least once.
