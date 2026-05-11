# Stage-Pass Template Scaffold Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use `superpowers:subagent-driven-development` (recommended) or `superpowers:executing-plans` to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Scaffold the stage-pass framework template — a working, shareable git repo containing all directories, template files, narrative docs, and abstract example scenarios described in the design spec.

**Architecture:** Pure docs/markdown repo. No code, no build, no tests in the runtime sense. "Verification" per task = file exists, required sections present, references resolve, ID conventions consistent. Files created in dependency order: skeleton → file templates → narrative docs → example scenarios → polish.

**Tech Stack:** Markdown, git, bash. Optional `jq` for `HANDOFF.json` validation. No package manager, no dependencies.

**Spec:** `docs/specs/2026-05-11-stage-pass-framework-design.md`

---

## File Structure

```
README.md                              new — framework intro, current plan pointer, quickstart
CLAUDE.md                              new — per-project rules template
.gitignore                             new — workspace policy aware
.planning/HANDOFF.json                 new — session continuity schema (empty)
plans/_template.md                     new — PLAN-vN template
plans/PLAN-current.md                  new — text pointer (placeholder)
stage-plan/_template.md                new — SP-NNN template
artifacts/_template.md                 new — A-NN template
workspace/raw/.gitkeep                 new
workspace/blockers/domain/.gitkeep     new
workspace/blockers/agent-inbox/.gitkeep new
docs/PROTOCOL.md                       new — framework spec (rendered from design)
docs/WHY.md                            new — rationale + antipatterns
examples/scenario-codebase-audit.md    new — abstract walkthrough
examples/scenario-data-migration.md    new — abstract walkthrough
examples/scenario-research-review.md   new — abstract walkthrough
```

Each file has one clear responsibility. Templates carry no domain content. Examples carry no real client data.

---

## Task 1: Directory skeleton + .gitkeep

**Files:**
- Create: `workspace/raw/.gitkeep`
- Create: `workspace/blockers/domain/.gitkeep`
- Create: `workspace/blockers/agent-inbox/.gitkeep`
- Create: `.planning/.gitkeep` (placeholder until HANDOFF.json added in Task 7)
- Create: `plans/.gitkeep`
- Create: `stage-plan/.gitkeep`
- Create: `artifacts/.gitkeep`
- Create: `examples/.gitkeep`

- [ ] **Step 1: Create dirs and .gitkeep files**

Run from repo root:
```bash
mkdir -p workspace/raw workspace/blockers/domain workspace/blockers/agent-inbox \
         .planning plans stage-plan artifacts examples
touch workspace/raw/.gitkeep \
      workspace/blockers/domain/.gitkeep \
      workspace/blockers/agent-inbox/.gitkeep \
      .planning/.gitkeep \
      plans/.gitkeep \
      stage-plan/.gitkeep \
      artifacts/.gitkeep \
      examples/.gitkeep
```

- [ ] **Step 2: Verify structure**

Run:
```bash
find . -type d -not -path './.git*' | sort
```

Expected output (order may vary):
```
.
./.planning
./artifacts
./docs
./docs/specs
./examples
./plans
./stage-plan
./workspace
./workspace/blockers
./workspace/blockers/agent-inbox
./workspace/blockers/domain
./workspace/raw
```

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "scaffold: directory skeleton with .gitkeep placeholders"
```

---

## Task 2: .gitignore

**Files:**
- Create: `.gitignore`

- [ ] **Step 1: Write .gitignore**

Contents of `.gitignore`:
```gitignore
# OS
.DS_Store
Thumbs.db

# Editors
.vscode/
.idea/
*.swp

# Claude / agent caches
.claude/
.cache/

# Workspace storage policy
# Default: workspace/ is tracked. Per-project override:
#   - For `external` policy: uncomment paths under workspace/raw/
#   - For `ignored` policy: uncomment workspace/raw/ entirely
# Examples (uncomment as needed):
# workspace/raw/
# workspace/raw/large-corpus/
# workspace/private/

# Local-only files
*.local
*.local.*
```

- [ ] **Step 2: Verify**

Run:
```bash
git check-ignore -v .DS_Store .claude/foo workspace/raw/.gitkeep
```

Expected: `.DS_Store` and `.claude/foo` matched; `workspace/raw/.gitkeep` NOT matched (workspace tracked by default).

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "scaffold: .gitignore with workspace policy hints"
```

---

## Task 3: HANDOFF.json schema

**Files:**
- Create: `.planning/HANDOFF.json`
- Delete: `.planning/.gitkeep` (replaced)

- [ ] **Step 1: Write HANDOFF.json**

Contents of `.planning/HANDOFF.json`:
```json
{
  "schema_version": 1,
  "current_plan": "PLAN-v1",
  "last_session": {
    "sp_id": null,
    "closed_at": null,
    "scope": null
  },
  "next_session": {
    "expected_sp_id": "SP-001",
    "starting_stage": "S-01",
    "scope": null,
    "notes": "First session of the project. Read README.md, plans/PLAN-current.md, and any agent-inbox entries before starting."
  },
  "open_blockers": {
    "domain": [],
    "agent_inbox": []
  }
}
```

- [ ] **Step 2: Validate JSON**

Run:
```bash
python3 -m json.tool .planning/HANDOFF.json > /dev/null && echo "valid"
```

Expected: `valid`

- [ ] **Step 3: Remove placeholder**

```bash
rm .planning/.gitkeep
```

- [ ] **Step 4: Commit**

```bash
git add .planning/
git commit -m "scaffold: HANDOFF.json session-continuity schema"
```

---

## Task 4: plans/_template.md and plans/PLAN-current.md

**Files:**
- Create: `plans/_template.md`
- Create: `plans/PLAN-current.md`
- Delete: `plans/.gitkeep`

- [ ] **Step 1: Write plans/_template.md**

Contents of `plans/_template.md`:
```markdown
# PLAN-v<N>

**Version:** <N>
**Frozen-at:** YYYY-MM-DD
**Supersedes:** PLAN-v<N-1>   <!-- omit for v1 -->
**Diff vs prev:** <1-3 lines summarizing what changed from previous version>

## Project goal

One paragraph. What this whole effort delivers and to whom.

## Constraints

- Deadline: ...
- Scope boundary: ...
- Out of scope: ...

## Stages

### S-01 — <stage goal>
**Status:** not-started | in-progress | done
**Goal:** ...
**Exit criteria:** ...
**Estimated sessions:** N

### S-02 — <stage goal>
...

## Risks

- ...

## Glossary

- **Term:** definition (only domain-specific terms)
```

- [ ] **Step 2: Write plans/PLAN-current.md**

Contents of `plans/PLAN-current.md`:
```markdown
# Current plan pointer

**Active plan:** none yet. Copy `_template.md` to `PLAN-v1.md`, fill it in, and update this file.

When v1 is written, replace the body of this file with a single line:

> Current plan: [PLAN-v1.md](PLAN-v1.md)

When v2 supersedes v1, replace with:

> Current plan: [PLAN-v2.md](PLAN-v2.md)
> Previous: [PLAN-v1.md](PLAN-v1.md)

Old versions are retained, never deleted.
```

- [ ] **Step 3: Remove placeholder**

```bash
rm plans/.gitkeep
```

- [ ] **Step 4: Commit**

```bash
git add plans/
git commit -m "scaffold: plans/ — PLAN-vN template + current pointer"
```

---

## Task 5: stage-plan/_template.md

**Files:**
- Create: `stage-plan/_template.md`
- Delete: `stage-plan/.gitkeep`

- [ ] **Step 1: Write stage-plan/_template.md**

Contents of `stage-plan/_template.md`:
```markdown
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
```

- [ ] **Step 2: Remove placeholder**

```bash
rm stage-plan/.gitkeep
```

- [ ] **Step 3: Commit**

```bash
git add stage-plan/
git commit -m "scaffold: stage-plan/_template.md — session handoff template"
```

---

## Task 6: artifacts/_template.md

**Files:**
- Create: `artifacts/_template.md`
- Delete: `artifacts/.gitkeep`

- [ ] **Step 1: Write artifacts/_template.md**

Contents of `artifacts/_template.md`:
```markdown
# A-<NN> — <what>

**Parent:** SP-<NNN>
**Lens:** <one lens — e.g. dedup, classification, code-triage, review, inventory, apply-log>
**Time-box:** <e.g. 90 min>
**Status:** complete | partial
**Created-at:** YYYY-MM-DD

---

## Method

How this pass was run. Tooling, queries, prompt, manual steps. Reproducibility matters.

## Result

The actual output. Tables, lists, mappings, decisions. The substance.

## Verify block

- Total items: N
- By bucket: ...
- Random samples (3): ID-1, ID-2, ID-3 — quoted inline so a reader can spot-check without leaving this file

## Notes for next pass

- What was deferred
- What surprised
- What would need re-running if upstream changes
```

- [ ] **Step 2: Remove placeholder**

```bash
rm artifacts/.gitkeep
```

- [ ] **Step 3: Commit**

```bash
git add artifacts/
git commit -m "scaffold: artifacts/_template.md — per-pass output template"
```

---

## Task 7: CLAUDE.md template

**Files:**
- Create: `CLAUDE.md`

- [ ] **Step 1: Write CLAUDE.md**

Contents of `CLAUDE.md`:
```markdown
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

Stages frozen for label families: after the listed stage, values may not be relabeled without an explicit decision log entry.

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

- Don't relabel frozen state families without a decision log entry.
- Don't write inside `workspace/raw/` — sources are immutable.
- Don't push mass-edits to external systems without dry-run counts first (verify block protects against fan-out bugs).
- Don't mix lenses in one pass.
- Don't hyperlink PLAN to stage-plans — IDs only.
```

- [ ] **Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "scaffold: CLAUDE.md — per-project rules template"
```

---

## Task 8: docs/PROTOCOL.md

**Files:**
- Create: `docs/PROTOCOL.md`

- [ ] **Step 1: Write docs/PROTOCOL.md**

Contents of `docs/PROTOCOL.md`:
```markdown
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

A family is a set of labels/statuses that, once assigned in a given stage, cannot be relabeled without an explicit decision-log entry. CLAUDE.md declares which families freeze after which stage. Freezing is what makes downstream passes trust the upstream classification.

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
```

- [ ] **Step 2: Commit**

```bash
git add docs/PROTOCOL.md
git commit -m "docs: PROTOCOL.md — framework specification"
```

---

## Task 9: docs/WHY.md

**Files:**
- Create: `docs/WHY.md`

- [ ] **Step 1: Write docs/WHY.md**

Contents of `docs/WHY.md`:
```markdown
# WHY — rationale and antipatterns

Why each rule exists. Read this when you're tempted to skip a rule.

## Why state lives in files

A chat session ends. The next session starts cold. If state was only in chat, every session re-derives context, drifts in interpretation, and quietly forgets decisions. Files survive session boundaries. The cost is small (one paragraph in an SP) and the benefit is large (any agent picks up where the last one stopped without an oral handoff).

Chat stays live within a session — the protocol doesn't replace it. It just makes sure that when chat dies, the work doesn't.

## Why PLAN is immutable per version

A mutating plan rots. Sub-stages multiply (S-04 splits into S-04, S-04a, S-04b), hyperlinks break, status markers contradict reality, and no one trusts the document.

Immutable PLAN-vN files force you to take new strategy as a deliberate act — write v2, summarize the diff, retain v1 for audit. The cost is rare (most projects ship on v1). The benefit is that PLAN is always trustworthy at version-cut.

## Why one stage-plan per session

A stage-plan is a handoff letter to the next agent. If one SP spans many sessions, the handoff stops being a snapshot and becomes a living document — and a living document mutates, drifts, contradicts.

One session = one SP = one closed handoff. The next session opens a new SP. Sequential SP-NNN numbering preserves the chain even when scope branches.

## Why one pass = one lens

When you mix lenses (e.g. dedup + classification in the same pass), you can't tell whether a wrong result came from the dedup logic or the classifier. Errors hide each other.

Worse: the diff becomes unreviewable. A reviewer can't scan a 500-item mixed-lens output and trust anything. Single-lens passes produce clean, comparable diffs and recoverable failures.

## Why verify blocks

Counts catch fan-out bugs. If a pass processed 480 of 500 items, the count mismatch is visible. Three random samples catch silent misclassification — a reviewer reads three IDs and decides whether to trust the whole batch.

Without verify blocks, errors surface only downstream, often after multiple passes have built on the bad output.

## Why hard time-box per pass

Overrun passes turn into multi-lens passes (the operator silently adds "while I'm here" work) or context-window failures (the agent runs out of working memory). Stopping at the box and deferring the remainder to a new SP keeps each pass clean.

## Why frozen state families

If `kind-req` can flip back to `kind-noise` after stage 4, every downstream pass has to recheck it. The cost of late-bound state is enormous in fan-out. Freezing is what lets `S-05` trust `S-03`'s output without re-verifying every item.

The escape hatch is the decision log — relabeling is allowed, but it's deliberate and recorded.

## Why no hyperlinks from PLAN to stage-plans

Plans evolve. Stage-plans split. If PLAN linked to SP-003 and SP-003 got superseded by SP-007, the link rots. Worse, PLAN-v1 hyperlinks to SP-003, PLAN-v2 hyperlinks to SP-007, and the diff between versions is dominated by link churn instead of strategy.

ID-only coupling (`See SP-003` in prose) is stable. Searching by ID in any tool resolves to the current state.

## Why commits per SP closure

Commit discipline makes `git log --oneline` readable as a handoff timeline. A new agent scrolling `git log` sees: `[SP-001] close`, `[SP-002] A-01..A-03`, `[SP-002] close`, `[blocker] domain`, `[SP-003] close`. That's the entire project narrative in 20 lines.

The cost is one commit per closure (free). The benefit is auditable history without writing a status doc.

## Antipatterns

| Antipattern                                                  | Why it breaks                          | Fix                                  |
|--------------------------------------------------------------|----------------------------------------|--------------------------------------|
| "I'll just update PLAN.md in place"                          | Trust in PLAN erodes; diff lost        | Write PLAN-vN+1; archive vN          |
| "This pass also did some cleanup while we're here"           | Errors hide; diff unreviewable         | Defer cleanup to its own pass        |
| "Skipping verify block because it's a small pass"            | Fan-out bugs surface downstream        | Verify block always                  |
| "The SP is taking longer than estimated, I'll extend it"     | Stale handoff; multi-lens drift        | Close as `partial`; open next SP     |
| "I'll relabel a few without a decision log entry"            | Downstream passes can't trust upstream | Decision log entry first             |
| "PLAN-v1 links to SP-003 § 'Findings'"                       | Link rot on SP supersede               | Reference by ID in prose             |
| "Stage-plan describes 3 goals to do this session"            | Becomes multi-lens; hard to verify     | Split into 3 SPs                     |
| "Workspace policy not declared, files dumped wherever"       | Secrets leak; repo bloats              | Declare in CLAUDE.md before SP-001   |
| "I'll just remember in chat what I decided"                  | Next session loses context             | Write it in SP or HANDOFF.json       |
```

- [ ] **Step 2: Commit**

```bash
git add docs/WHY.md
git commit -m "docs: WHY.md — rationale + antipatterns table"
```

---

## Task 10: examples/scenario-codebase-audit.md

**Files:**
- Create: `examples/scenario-codebase-audit.md`
- Delete: `examples/.gitkeep`

- [ ] **Step 1: Write examples/scenario-codebase-audit.md**

Contents of `examples/scenario-codebase-audit.md`:
```markdown
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

## Context for next agent
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

## Context for next agent
S-01 done for all 4 services. Inventories in artifacts/SP-001..SP-004.
Tool for this lens: `semgrep` with ruleset X.

## Verify block
Findings: 23 (12 high, 8 med, 3 low).
Samples (3): F-001 (auth bypass), F-007 (sql-fragment), F-019 (open redirect).
```

`severity-*` family freezes after S-02 closes for a service.

## Things this scenario demonstrates

- Sequential SPs across multiple scopes (service-a..service-d) without scope-encoded IDs.
- S-NN advances when all scopes exit, not per scope.
- Lens stays single throughout a pass (inventory ≠ security ≠ dead-code).
- Freezing severity-* lets S-05 (consolidation) trust S-02's classification.
```

- [ ] **Step 2: Remove placeholder**

```bash
rm examples/.gitkeep
```

- [ ] **Step 3: Commit**

```bash
git add examples/
git commit -m "examples: scenario-codebase-audit walkthrough"
```

---

## Task 11: examples/scenario-data-migration.md

**Files:**
- Create: `examples/scenario-data-migration.md`

- [ ] **Step 1: Write examples/scenario-data-migration.md**

Contents of `examples/scenario-data-migration.md`:
```markdown
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

## Goal
Transform batch-1 high-confidence records to New schema. Reject any record where any field flips confidence to <high.

## Verify block
Input: 6000. Transformed: 5876. Rejected for re-triage: 124.
Samples (3): R-00041 (clean), R-00302 (clean), R-04711 (clean).

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
```

- [ ] **Step 2: Commit**

```bash
git add examples/scenario-data-migration.md
git commit -m "examples: scenario-data-migration walkthrough"
```

---

## Task 12: examples/scenario-research-review.md

**Files:**
- Create: `examples/scenario-research-review.md`

- [ ] **Step 1: Write examples/scenario-research-review.md**

Contents of `examples/scenario-research-review.md`:
```markdown
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

## Context for next agent
SP-002..SP-004 covered P-001..P-015. Extraction template stable since SP-002.
Note: P-017 is a survey paper — extract themes, skip methods.

## Goal
Produce one extraction artifact per paper P-016..P-020.

## Verify block
Papers extracted: 5/5.
Samples (3): P-016 § findings, P-018 § methods, P-020 § limitations.

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
```

- [ ] **Step 2: Commit**

```bash
git add examples/scenario-research-review.md
git commit -m "examples: scenario-research-review walkthrough"
```

---

## Task 13: README.md + final verify

**Files:**
- Create: `README.md`

- [ ] **Step 1: Write README.md**

Contents of `README.md`:
```markdown
# Stage-Pass Template

A protocol for multi-session agent-driven work. State lives in files so any next session — yours, another agent's, a colleague's — cold-resumes from disk. Chat stays live; files carry state.

**Status:** v0 scaffold. Validated against one real-world project (private). Iterating.

## When to use

- Large tasks spanning many sessions (audits, migrations, multi-pass classification, long refactors, research reviews).
- Work shared across humans + agents.
- Anywhere you've felt "I forgot what I decided last session" or "the previous agent left me nothing to pick up from".

## When NOT to use

- One-shot tasks (no handoff needed).
- Real-time collaborative editing (this is async-only).
- Anywhere a single PR/commit message suffices.

## Quickstart

1. Clone or copy this repo as your new project skeleton.
2. Read `docs/PROTOCOL.md` (10 min).
3. Open `CLAUDE.md`, fill in the `<...>` blanks: project name, goal, output destinations, state families, workspace policy.
4. Copy `plans/_template.md` → `plans/PLAN-v1.md`. Fill in stages.
5. Update `plans/PLAN-current.md` to point at `PLAN-v1.md`.
6. Open first session: copy `stage-plan/_template.md` → `stage-plan/SP-001-<what>.md`. Fill in.
7. Work. Produce artifacts under `artifacts/SP-001/`. Close SP. Commit. Open SP-002.

## Layout

```
README.md                 you are here
CLAUDE.md                 per-project rules (edit before first session)
plans/                    strategic — immutable per version
  PLAN-current.md         pointer
  PLAN-v1.md              (you create)
  _template.md
stage-plan/               one file per session handoff
  SP-001-<what>.md        (you create)
  _template.md
artifacts/                grouped by parent SP
  SP-001/
    A-01-<what>.md        (you create)
  _template.md
workspace/
  raw/                    source-of-truth inputs (policy-dependent)
  blockers/
    domain/               questions for humans
    agent-inbox/          tasks for the next agent
.planning/HANDOFF.json    last-session state snapshot
docs/
  PROTOCOL.md             framework spec
  WHY.md                  rationale + antipatterns
examples/
  scenario-codebase-audit.md
  scenario-data-migration.md
  scenario-research-review.md
```

## Key principles

- **State lives in files. Chat is live but ephemeral.**
- **PLAN is immutable per version.** New strategy = new file.
- **One stage-plan = one session handoff.** Sequential `SP-NNN`.
- **One pass = one lens.** Don't mix.
- **Verify block at end of every pass.** Counts + 3 samples.
- **Commits are part of the protocol.** `git log` = handoff timeline.
- **No hyperlinks from PLAN to stage-plans.** Couple by ID only.

Full rationale in `docs/WHY.md`.

## License

TBD (pick one before public release).
```

- [ ] **Step 2: Final scaffold verify**

Run from repo root:
```bash
find . -type f -not -path './.git*' | sort
```

Expected output (exact set):
```
./.gitignore
./.planning/HANDOFF.json
./CLAUDE.md
./README.md
./artifacts/_template.md
./docs/PROTOCOL.md
./docs/WHY.md
./docs/specs/2026-05-11-stage-pass-framework-design.md
./docs/specs/2026-05-11-stage-pass-framework-plan.md
./examples/scenario-codebase-audit.md
./examples/scenario-data-migration.md
./examples/scenario-research-review.md
./plans/PLAN-current.md
./plans/_template.md
./stage-plan/_template.md
./workspace/blockers/agent-inbox/.gitkeep
./workspace/blockers/domain/.gitkeep
./workspace/raw/.gitkeep
```

If any file missing → re-run the corresponding task.

- [ ] **Step 3: ID-convention consistency check**

Run:
```bash
grep -rE 'SP-[0-9]{3}|PLAN-v[0-9]|S-[0-9]{2}|A-[0-9]{2}' --include='*.md' . | head -20
```

Expected: all matches use the exact patterns `SP-NNN`, `PLAN-vN`, `S-NN`, `A-NN`. No `SP-1`, `Plan-v1`, `Stage-04` variations.

- [ ] **Step 4: Commit README**

```bash
git add README.md
git commit -m "scaffold: README.md — quickstart + layout map"
```

- [ ] **Step 5: Push**

```bash
git push origin main
```

Expected: branch up to date with remote.

---

## Self-review checklist (run after plan execution)

- [ ] Every file in spec's "Directory layout" exists in the repo.
- [ ] Every file schema in spec has a matching template in the repo.
- [ ] Every principle in spec is reflected in `CLAUDE.md` or `docs/PROTOCOL.md`.
- [ ] All three example scenarios are abstract — no real client names (Jurix, Dreambis, etc).
- [ ] `git log --oneline` reads as a clean scaffold timeline (one commit per task).
- [ ] `.gitignore` blocks `.claude/` and `.cache/`.
- [ ] No TODO/TBD/placeholder text outside template `<...>` markers and the README license line.
