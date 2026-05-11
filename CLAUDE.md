# CLAUDE.md — <project name>

**Goal:** <one sentence describing the whole effort>
**Output:** <where final deliverables live>
**Code:** read-only references to external repos; never modify external code from this project.

---

## Conventions

### IDs

| ID         | Meaning                                | Example         |
|------------|----------------------------------------|-----------------|
| `PLAN-vN`  | Plan version (immutable once written)  | `PLAN-v2`       |
| `S-NN`     | Stage inside a plan                    | `S-04`          |
| `SP-NNN`   | Stage-plan = one session handoff       | `SP-003`        |
| `A-NN`     | Artifact inside a stage-plan           | `A-01`          |

Cross-refs are explicit by ID. PLAN does not link to stage-plans — coupling is via ID only.

### Files

- `plans/PLAN-vN.md` — strategic, immutable per version. New version = new file. `plans/PLAN-current.md` points to active.
- `stage-plan/SP-NNN-<what>.md` — one session handoff. Sequential `NNN` across whole project.
- `artifacts/SP-NNN/A-NN-<what>.md` — outputs of one pass. Scope inherited from parent SP.
- `workspace/raw/` — source-of-truth inputs (immutable).
- `workspace/blockers/domain/` — questions for humans.
- `workspace/blockers/agent-inbox/` — tasks/notes for the next agent.

### Per-pass discipline

1. **One pass = one lens.** Don't mix classification + dedup + review in one pass.
2. **Verify block required** at end of every artifact: counts + 3 random samples quoted by ID.
3. **Hard time-box per pass.** If overflow, defer to next pass — don't extend.
4. **Apply-log artifact** commits with the pass.

### State families (illustrative — replace with your project's)

| Family       | Values                                      | Owner       | Frozen after |
|--------------|---------------------------------------------|-------------|--------------|
| `phase-*`    | `phase-intake`, `phase-triage`, `phase-done`| lead        | S-02         |
| `priority-*` | `priority-p0`, `priority-p1`, `priority-p2` | lead        | S-04         |
| `status-*`   | `status-open`, `status-blocked`, `status-resolved` | agent | —            |

Stages frozen for label families: after the listed stage, values may not be relabeled without a decision recorded in that pass's apply-log artifact.

### Source-system gotchas

<Project-specific quirks of your data sources go here. Examples: "issue type is `История` in Russian Jira", "API rate limit 100/min", "field X is nullable but in practice never null". Empty for new projects.>

### Workspace storage policy

This project uses: `tracked` | `external` | `ignored`

- `tracked` — committed to git as-is. For small raw, design docs, screenshots.
- `external` — pointer file in `workspace/README.md` records the external location; content lives outside the repo.
- `ignored` — `workspace/` (or sub-path) is in `.gitignore`; shared out-of-band.

Mix is OK: e.g. `workspace/raw/` ignored while `workspace/blockers/` tracked. Update `.gitignore` accordingly.

---

## Commit discipline

| Event                    | Commit message format                        |
|--------------------------|----------------------------------------------|
| Stage-plan closed        | `[SP-003] close: <title>`                    |
| Artifact batch           | `[SP-003] A-01..A-03: <what>`                |
| New plan version         | `[PLAN-v2] bump from v1: <1-line diff>`      |
| Workspace tracked add    | `[workspace] add: <what>`                    |
| Blocker raised           | `[blocker] domain or agent-inbox: <title>`   |

`git log --oneline` should read as a handoff timeline.

---

## Memory + context

- Long-running user-level facts → `~/.claude/projects/.../memory/` via `MEMORY.md` index.
- Project-level conventions → this file. Update on correction.
- Session continuity → `.planning/HANDOFF.json`.
- Chat is the live working layer within one session. Files are state across sessions.

---

## Things to avoid

- Don't relabel frozen state families without an apply-log decision entry.
- Don't write inside `workspace/raw/` — sources are immutable.
- Don't push mass-edits to external systems without dry-run counts first (verify block protects against fan-out bugs).
- Don't mix lenses in one pass.
- Don't hyperlink PLAN to stage-plans — IDs only.
