# State-as-Files: A Manifesto for Multi-Session Agent Work

You open a new session with an AI agent. Context from the last session — empty. You re-explain why, what for, what's already been done, what you decided about point three. An hour lost before the first useful task.

Is this normal? No. It's a symptom of one simple antipattern: state lives in chat.

## Chat-as-memory — where it breaks

Chat is convenient. You type, the agent answers, everything in flow. On short tasks it works perfectly.

But chat is an ephemeral thing. And as soon as a task stops fitting in a single session, the losses begin:

- Session closed — state lost.
- Context window full — state partially lost (and worst of all: you don't know which part fell out).
- A different agent — state fully lost.
- A colleague picks up the work — they have none of your state.
- A day passed — your own head forgot the state.

Large tasks (a codebase audit across four services, a migration of 18,000 records, a literature review of 80 papers, an estimation project covering hundreds of stories before a hard deadline) **do not fit in one session**. Never. They run for days, weeks, sometimes months. And every new session is a loss.

The symptoms are familiar:

- "I asked this last week."
- "Wait, what did we decide about X?"
- "The agent doesn't understand what we've already finished."
- "We discussed this — go open the Tuesday chat."
- Interpretive drift — each session quietly rewrites prior decisions, slightly.

The worst part: the failures aren't loud. Nothing crashes with a stack trace. The project just slowly rots under a layer of "I forgot," "we re-decided," "let me start from a different angle." Two weeks in, no one knows where things stand or what's trustworthy.

## The idea: state in files, chat stays live

A parallel from the world of code. Git has been around for 20 years. We long ago stopped arguing about whether code lives in files or in IDE state. The IDE is a working tool. Files are the single source of truth. Close the IDE — you've lost nothing. Open a different one — keep going. Share with a colleague — share files, not your workspace.

The same principle works for agent-driven tasks:

- **Chat** is live working memory **inside one session**. Back-and-forth, iteration, thinking out loud.
- **Files** are memory **across sessions**. The project goal, the stage plan, the stage-plans, the artifacts, the blockers — everything under git history.

Chat isn't killed. It stays live, a working instrument. But when the session ends, nothing important is left there. Everything is in files. The next session — yours, another agent's, a colleague's — opens the files and continues.

That's state-as-files. The rest is mechanics.

## Principles

### 1. The project goal lives in the plan, under version

The plan opens with the project's goal. One sentence. No "we'll discuss later," no "roughly understood," no "we'll figure it out as we go." The goal is explicit, in a file, readable in two seconds.

What this buys you. Every session — yours tomorrow, another agent's, a colleague's — starts by reading the goal. Nobody drifts on "what are we even doing." The agent doesn't slide into an adjacent problem over five sessions. The project's finale is checked against what's written on line one, not against your interpretation of the goal three weeks in.

The goal changes — you write `PLAN-v2.md` with an explicit diff against v1 in the header: "added stage S-05 after the PM review on 2026-05-11." v1 doesn't get edited, isn't deleted, stays archived. A README points at the current version. Changing the goal is a deliberate act, recorded in git history, visible as a two-line diff.

The same rule applies to the rest of the plan: stages, exit criteria, constraints. The plan is a strategic snapshot, frozen per version. A mutating plan rots: sub-stages multiply (`S-04` → `S-04a` → `S-04b`), status markers contradict reality, two weeks in nobody trusts the document. Immutability is what preserves trust in v1 at the moment its version was frozen. It happens rarely (most projects ship on v1). But v1 is always a working snapshot.

### 2. One stage-plan = one session handoff

Each session produces exactly **one** stage-plan for the next agent (or for you tomorrow). Never "I'll update the old SP" — that's how you turn a snapshot into a mutating document.

```
Agent A finishes → writes SP-003 → Agent B
reads PLAN-current + SP-003 → executes →
writes artifacts + a new SP-004 for Agent C
```

ID convention: `SP-001`, `SP-002`, `SP-003` — a global sequential counter across the whole project. Parallel branches of work get encoded in the `Scope:` field in the SP header, not by mangling the numbering.

The SP header carries short context for the next agent: what the previous one learned, decided, ran into. Five to fifteen lines. Enough that a cold session can open and not ask "wait, why are we doing this at all."

### 3. Every pass ends with a verify block

An artifact without a verify block is an unfinished artifact. The end of every pass requires:

- Total items processed: N
- Counts by bucket / class / label: …
- Three random samples quoted by ID

Why. Counts catch fan-out bugs. If a pass processed 480 of 500 items, you'll see it. Without verify it surfaces two passes downstream, when debugging is painful.

Samples catch silent misclassification. A reviewer reads three IDs, decides whether to trust the whole batch. Without samples the first wrong call surfaces only at the end, when redoing the work is expensive.

It's cheap (five lines at the bottom of an artifact) and saves hours over the long run.

### 4. One pass = one lens

Don't mix passes. Dedup is its own pass. Classification is its own pass. Code review is its own pass. Never "while I'm here, I'll clean up the noise too."

Why. A mixed-lens pass is unreviewable. A reviewer can't read 500 items of mixed output and trust anything. The diff becomes unreadable. Worse: errors from one lens hide errors from another. When something goes wrong, you can't tell which logic produced it.

Single-lens passes produce clean, comparable diffs and recoverable failures. This isn't pedantry — it's engineering.

### 5. Hard time-box per pass

You scheduled the pass for 90 minutes — close it at 90 minutes. Didn't finish — write the remainder into the handoff section and open a new SP for the continuation.

Why. Overrunning passes turn into multi-lens passes — the operator silently adds "while I'm here" work. Or into context-window failures — the agent runs out of working memory and starts hallucinating. A hard time-box keeps each pass clean.

### 6. Commits as part of the protocol

Each closed SP is a commit. Each artifact batch is a commit. A plan bump is a commit with the diff in the body.

```
[SP-007] close: dep-update service-d
[SP-007] A-01..A-02: dep audit + apply log
[SP-006] close: dep-update service-c
[PLAN-v2] bump from v1: added dep-update stage
[blocker] domain: license-X compatibility?
[SP-005] close: dep-update service-b
…
```

`git log --oneline` reads as a handoff timeline. A new agent scrolls the log and gets the **entire project narrative in 20 lines**. No status docs, no weekly updates, no storytelling. The history is already there.

### 7. Coupling by ID, not by hyperlink

Don't put links from the plan to stage-plans. No `[See SP-003](../stage-plan/SP-003.md)` in the plan's body. Just prose: `See SP-003.`

Why. Plans evolve, SPs split, links rot. `PLAN-v1` linked to `SP-003`; later `SP-003` got superseded by `SP-007` — link broken. The diff between plan versions becomes noisy with link churn instead of strategy.

ID-only coupling is stable. A grep for `SP-003` finds current state in any tool. An ID outlives a filesystem path.

### 8. Frozen state families

If you have a classification (priority-*, severity-*, kind-*) — once it's assigned in a given stage, the label **freezes**. Reclassification without an explicit apply-log entry is forbidden.

Why. If `severity-high` can flip back two stages later, every downstream pass has to recheck the whole batch. Late-bound state has enormous fan-out cost. Freezing is what lets `S-05` trust `S-03`'s output without re-running.

The escape hatch stays: relabeling is allowed, but only through a deliberate apply-log entry. On purpose. Recorded. Not "by accident."

## Antipatterns

| Antipattern | What breaks | Fix |
|---|---|---|
| "I'll just update PLAN.md in place" | Trust in the plan erodes; the diff is lost | Write PLAN-vN+1, archive vN |
| "This pass will also clean up the noise" | Errors hide in a mixed diff; unreviewable | Cleanup is its own pass |
| "Small pass, I'll skip the verify block" | Fan-out bugs surface three passes downstream | Verify block always |
| "The SP is taking longer, I'll extend it" | Stale handoff; multi-lens drift | Close as done with overflow note; open next SP |
| "PLAN-v1 links to SP-003 § Findings" | Link rot when an SP is superseded | Reference by ID in prose |
| "I'll relabel a few without a log entry" | Downstream can't trust upstream | Apply-log decision entry first |
| "I remember in chat what we decided" | The next session loses the context | Write it into the SP or HANDOFF.json |

## The template

I packaged all of this into a template repo: **[github.com/socaseinpoint/stage-pass-template](https://github.com/socaseinpoint/stage-pass-template)**.

What's inside:

- Directory structure (`plans/`, `stage-plan/`, `artifacts/`, `workspace/`, `.planning/`)
- File templates (`_template.md` for PLAN, SP, artifacts)
- A `CLAUDE.md` starter — fill in for your project (state families, gotchas, workspace policy)
- `docs/PROTOCOL.md` — framework specification
- `docs/WHY.md` — rationale for every rule plus an antipattern table
- Three abstract scenario walkthroughs (codebase audit, data migration, research review) — each shows how the protocol lands on a different class of work

The minimal loop:

```bash
# 1. Clone the template under your project
git clone https://github.com/socaseinpoint/stage-pass-template.git my-project
cd my-project
rm -rf .git && git init      # start your own history

# 2. Fill in conventions
$EDITOR CLAUDE.md            # replace <...> placeholders
                             # state families, workspace policy, gotchas

# 3. Write the first plan version
cp plans/_template.md plans/PLAN-v1.md
$EDITOR plans/PLAN-v1.md
$EDITOR plans/PLAN-current.md   # point at PLAN-v1.md

# 4. Open the first session handoff
cp stage-plan/_template.md stage-plan/SP-001-collect-inputs.md
$EDITOR stage-plan/SP-001-collect-inputs.md

# 5. Work. Drop artifacts as the pass runs.
mkdir -p artifacts/SP-001
$EDITOR artifacts/SP-001/A-01-inventory.md
$EDITOR artifacts/SP-001/A-02-apply-log.md   # verify block lives here

# 6. Close the SP with a commit (sp status → done)
git add -A
git commit -m "[SP-001] close: collect inputs"

# 7. Next session — new SP, same loop
cp stage-plan/_template.md stage-plan/SP-002-classify.md
```

After 3-4 SPs the loop becomes automatic. After 10, `git log --oneline` already reads like a project map.

## When this fits

- Large tasks running across many sessions (audits, migrations, classification, refactors, research).
- Work shared between agents and humans, between people on a team.
- Anywhere you've felt "I forgot what I decided last week" or "the previous agent left me nothing to pick up from."

## When this doesn't fit

- **One-shot tasks** — no handoff needed; one session covers it.
- **Autonomous agent loops** — this framework assumes a human in the loop at every step (verify blocks, artifact review, explicit SP closure). There is no auto-chain of agents executing without supervision. If you want full autonomy, this is the wrong shape.

## Closing

The idea is simple. State in files, chat stays live. Git and its apologists have done this for code for 20 years — nobody argues anymore. For long-running work with AI agents it's just as critical, because agents remember the previous session **even worse** than humans do.

Chat is working memory. Files are memory across sessions. When chat dies, the work shouldn't die with it.

---

*Template: [github.com/socaseinpoint/stage-pass-template](https://github.com/socaseinpoint/stage-pass-template)*
*Published: 2026-05-11*
