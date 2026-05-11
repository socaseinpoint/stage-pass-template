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

If `severity-high` can flip back to `severity-low` after stage 4, every downstream pass has to recheck it. The cost of late-bound state is enormous in fan-out. Freezing is what lets `S-05` trust `S-03`'s output without re-verifying every item.

The escape hatch is the apply-log decision entry — relabeling is allowed, but it's deliberate and recorded.

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
| "The SP is taking longer than estimated, I'll extend it"     | Stale handoff; multi-lens drift        | Close as `done` with overflow noted in handoff section; open next SP for remainder |
| "I'll relabel a few without an apply-log decision entry"     | Downstream passes can't trust upstream | Apply-log decision entry first       |
| "PLAN-v1 links to SP-003 § 'Findings'"                       | Link rot on SP supersede               | Reference by ID in prose             |
| "Stage-plan describes 3 goals to do this session"            | Becomes multi-lens; hard to verify     | Split into 3 SPs                     |
| "Workspace policy not declared, files dumped wherever"       | Secrets leak; repo bloats              | Declare in CLAUDE.md before SP-001   |
| "I'll just remember in chat what I decided"                  | Next session loses context             | Write it in SP or HANDOFF.json       |
