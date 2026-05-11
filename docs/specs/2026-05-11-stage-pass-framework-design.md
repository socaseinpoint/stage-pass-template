# Stage-Pass Framework — Design Spec

**Date:** 2026-05-11
**Status:** Draft, awaiting user review

---

## Problem

Large agent-driven tasks (audits, migrations, multi-pass classification, long refactors, research reviews) lose state between sessions. Chat history is ephemeral. Without a persistence protocol, every new session re-derives context from scratch, drifts, or loses prior decisions.

Existing approaches:
- **Pure chat:** state dies when session ends.
- **Single PLAN.md:** plan mutates, no audit trail, sub-stages create link rot.
- **Ad-hoc notes:** no convention, no cold-resume.

## Goal

A shareable, language-agnostic framework that lets any session (your next, another agent, a colleague) cold-resume a large task by reading files. Chat stays live; files carry state across sessions.

Applicable to: software development, codebase audits, multi-source data migration, estimation, multi-pass classification, research reviews — anything where one corpus is processed across many lenses over many sessions.

## Principles

1. **State lives in files, chat is ephemeral.** Files persist across sessions. Chat is the live working layer inside one session. Next session reads files, not chat.
2. **PLAN is immutable per version.** New plan version = new file. Old versions retained for audit. README points to current.
3. **One stage-plan = one session handoff.** Agent A finishes → writes stage-plan for agent B → B reads PLAN + last stage-plan → executes → writes artifacts + new stage-plan.
4. **One pass = one lens.** Don't mix classification + dedup + review in one pass. Each pass ends with a verify block.
5. **Commits are part of the protocol.** Each closed stage-plan = commit. Each artifact batch = commit. `git log` = handoff timeline.
6. **Abstract over domain.** Framework is generic. Project-specific labels, sources, gotchas live in per-project `CLAUDE.md`.

## ID convention

| ID    | Meaning                            | Example                        |
|-------|------------------------------------|--------------------------------|
| `PLAN-vN`   | Plan version (immutable once written) | `plans/PLAN-v2.md`        |
| `S-NN`      | Stage inside a plan (numbered, fixed in plan) | `S-04`             |
| `SP-NNN`    | Stage-plan = one session handoff (global sequential counter) | `SP-003`         |
| `A-NN`      | Artifact inside a stage-plan (counter resets per SP) | `A-01`              |

Cross-refs are explicit by ID. No hyperlinks from PLAN to stage-plans — coupling is only through ID/numbering convention. This avoids link rot when stages split (`S-04` → `SP-003`, `SP-004`, `SP-005`).

## Directory layout

```
README.md                     framework intro, points to plans/PLAN-current
CLAUDE.md                     project rules, conventions, workspace policy, label families
.gitignore                    per workspace policy
.planning/HANDOFF.json        last-session state snapshot

plans/
  PLAN-v1.md
  PLAN-v2.md
  PLAN-current.md             text pointer (or symlink) to active version

stage-plan/
  SP-001-<short-what>.md
  SP-002-<short-what>.md
  _template.md

artifacts/
  SP-001/
    A-01-<what>.md
    A-02-apply-log.md
  SP-002/
    A-01-<what>.md
  _template.md

workspace/
  raw/                        SoT immutable inputs (per policy)
  blockers/
    domain/                   questions for humans (decisions, clarifications)
    agent-inbox/              tasks/notes left for next agent

examples/
  scenario-codebase-audit.md
  scenario-data-migration.md
  scenario-research-review.md

docs/
  PROTOCOL.md                 framework specification
  WHY.md                      rationale, antipatterns, lessons
  specs/                      per-feature design docs (this file)
```

## File schemas

### `plans/PLAN-vN.md`

```markdown
# PLAN-v2

**Version:** 2
**Frozen-at:** 2026-05-11
**Supersedes:** PLAN-v1
**Diff vs prev:** (1-3 lines on what changed)

## S-01 — <stage goal>
**Status:** done | in-progress | not-started
**Goal:** ...
**Exit criteria:** ...

## S-02 — ...
```

### `stage-plan/SP-NNN-<what>.md`

```markdown
# SP-003 — <short title>

**Plan:** PLAN-v2
**Stage:** S-04
**Scope:** alpha
**Prev:** SP-002
**Status:** in-progress | done | superseded
**Closed-at:** 2026-05-11
**Closed-by:** SP-005   # only if superseded

## Context for next agent
What the previous agent learned, decided, ran into.

## Goal of this session
One executable goal. If multiple → split into multiple SPs.

## Steps
1. ...
2. ...

## Verify block
Required at end. Counts + N random samples. See PROTOCOL.md.

## Output
List of artifacts produced under `artifacts/SP-003/`.
```

### `artifacts/SP-NNN/A-NN-<what>.md`

```markdown
# A-01 — <what>

**Parent:** SP-003
**Lens:** dedup | classification | code-triage | review | ...
**Time-box:** 90 min
**Status:** complete | partial

## Method
...

## Result
...

## Verify block
- Total items: N
- By bucket: ...
- Random samples (3): ID-1, ID-2, ID-3
```

## Workspace storage policy

Three modes, chosen per project, declared in `CLAUDE.md`:

| Mode       | Use case                                    | `.gitignore` |
|------------|---------------------------------------------|--------------|
| `tracked`  | Small raw, design docs, screenshots, text   | not ignored  |
| `external` | Heavy raw, corp data, video                 | path ignored, `workspace/README.md` records external location |
| `ignored`  | Secrets, local dumps, sensitive             | path in `.gitignore`, shared out-of-band |

Mix OK: `workspace/raw/` ignored, `workspace/blockers/` tracked.

## Commit discipline

| Event                  | Commit message format                            |
|------------------------|--------------------------------------------------|
| Stage-plan closed      | `[SP-003] close: <title>`                        |
| Artifact batch         | `[SP-003] A-01..A-03: <what>`                    |
| New plan version       | `[PLAN-v2] bump from v1: <1-line diff>`          |
| Workspace tracked add  | `[workspace] add: <what>`                        |
| Blocker raised         | `[blocker] domain or agent-inbox: <title>`       |

Result: `git log --oneline` reads as a clean handoff timeline.

## Per-pass discipline (carried into CLAUDE.md template)

1. One pass = one lens. No mixing.
2. Verify block ends every pass: counts + N random samples quoted.
3. Hard time-box per pass. Overflow → next pass, don't extend.
4. Apply log artifact commits with the pass.

## What's NOT in v1

- Slash commands and hooks (defer until after first real use)
- Domain-specific scripts (Jira, Confluence, etc.)
- Automated SP generators
- Web UI / dashboards

YAGNI until the bare framework is used on at least one non-template project.

## Open questions deferred to implementation phase

- `PLAN-current` as text pointer vs symlink — default text pointer for cross-platform; revisit if friction.
- `examples/` — how detailed each scenario walkthrough should be (decided in planning).
- Whether `_template.md` files are tracked or `.template` extension to avoid being mistaken for real files.

## Hosting

Standalone repo lives inside `projects/03-stage-pass-template/` (own `.git`). Can later be promoted to a public template repo with `gh repo create --template`. Current scope: local repo only.
