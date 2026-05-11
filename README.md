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
