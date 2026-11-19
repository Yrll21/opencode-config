---
name: Autonomous Delivery
description: Use when the Product Owner asks for a feature, bug fix or engineering objective to be delivered end to end through the orchestrator. Composes Superpowers brainstorming, planning, subagent-driven development, review and verification behind two approval gates.
---

# Autonomous Delivery

## Overview

You are the orchestrator. The Product Owner supplies an objective; you return
one approved proposal, execute it without routine checkpoints, and return one
evidence-backed completion report.

**Core principle:** Superpowers owns the mechanics. This skill owns only the
gates, the specialist selection, wave planning, and the consolidation of
evidence. Do not restate or replace a Superpowers skill; load it and follow it.

**Announce at start:** "I'm using the autonomous-delivery skill to run this
delivery."

## Phase 0 — Resolve the repository contract

Read, in this order:

1. The repository `AGENTS.md` (the only instruction file; `CLAUDE.md` is a
   symlink or absent).
2. Its `## Agent workflow profile` block, if present.
3. `package.json` scripts or the equivalent task runner.
4. Any `docs/agent-reference/` or design-system file named by the profile.

The profile block has this shape:

```md
## Agent workflow profile
- verify-fast: <commands>
- verify-full: <commands>
- never-autonomous: <comma-separated>
- plan-home: <path>
- status-home: <path>
- design-system: <path>
- parallel: on | off
- worktree: allowed | ask
```

Rules:

- Missing profile → derive the commands from the project, record the assumption
  as a ruling, and include "add the profile block" in the proposal.
- `parallel: off` → execute strictly sequentially, one task at a time.
- `worktree: ask` → request consent once, in the proposal, before creating it.
- A profile may declare *which* verification commands run and when. It may not
  remove the fast tier or the evidence requirement.

### Verification tiers

| Tier | When |
| --- | --- |
| fast | always, after a meaningful change |
| full | before handoff, or during work when the profile allows |
| never autonomous | only with explicit approval |

## Phase 1 — Propose

Load `brainstorming`, then `writing-plans`. Keep the conversation proportional:
a one-behaviour request gets a short proposal; multi-part work gets the full
design and plan.

Investigate before proposing:

- `explore` for repository mapping when the area is unfamiliar.
- `architect` when the design has real alternatives, cross-cutting contracts,
  migrations, or authorization boundaries.
- Skip both when the change is local and the shape is obvious.

Write the plan to the profile's `plan-home`, defaulting to
`docs/superpowers/plans/YYYY-MM-DD-<slug>.md`. If `plan-home` names a file
rather than a directory, append the delivery plan under a dated heading; never
replace the file. Every task in the plan carries:

```md
### Task N — <name>
**Depends on:** <task ids or none>
**Files touched:** <paths or globs>
**Owner:** general | qa-engineer
**Acceptance:** <observable checks>
**Verification:** <commands>
```

## Gate 1 — Delivery Proposal

Present exactly this, in one message, then ask with the `question` tool:

- **Outcome** — what will be true when this is done, and why.
- **Acceptance criteria** — observable, testable.
- **Tasks** — id, owner, dependencies, files, verification.
- **Waves** — which tasks run together, or `sequential`.
- **Specialists** — who is used and why; name the roles you deliberately skip.
- **Isolation** — worktree name, branch, base.
- **Risks and unknowns** — including profile assumptions.
- **Rulings already made** — reversible decisions you took.

Options: `Approve`, `Revise`, `Reject`. Do not start implementation before
`Approve`. Revise means update the proposal and ask again.

## Phase 2 — Execute

Load `using-git-worktrees`, then `subagent-driven-development`. Create the
worktree at `../.worktrees` named `<repo>/<type>/<slug>`, or the profile's
declared destination. Never implement on `main`/`master`.

Follow SDD for the task loop, ledger, report files, preflight, fix rounds and
the final review. The additions below override only where they differ.

Before the first task, confirm `.superpowers/` is git-ignored; add it to the
repository's `.gitignore` if it is not, and record that as a delivery change.

### Specialist selection

| Work | Dispatch to |
| --- | --- |
| Repository mapping | `explore` |
| Design, contracts, migrations, stuck rounds 4–5 | `architect` |
| Implementation | `general` |
| Independent test/build/E2E evidence | `qa-engineer` |
| Routine task or wave review | `task-reviewer` |
| UI/UX review against the design system | `task-reviewer` with the design-system brief |
| Security, authorization, concurrency, recovery, final whole-branch | `reviewer` |

Do not dispatch every role. A local one-behaviour delivery may be plan →
implement → task review. Add `qa-engineer` when verification is non-trivial or
the worker's evidence is thin. A specialist whose output would not change a
decision is wasted cost.

### Waves

Derive waves from `Depends on:` and `Files touched:`.

- Parallel only when files are disjoint, no shared contract changes, and each
  brief is complete. Cap at three concurrent implementers.
- Overlapping files, a shared contract, or `parallel: off` → sequential.
- Same-shape small edits → batch them (SDD `references/batching.md`).
- Issue all dispatches for a wave in one response so they run concurrently.

**Wave review:** one `task-reviewer` dispatch covering the wave's diff, with a
verdict per task. Each task still gets an explicit spec-compliance and quality
verdict; do not let one task's review hide another's failure.

### Evidence, not reports

A worker's report is a claim. Before accepting a task:

- Read the actual diff for the task's range.
- Confirm the covering test exists and failed before the fix where the repo has
  tests.
- Run the acceptance command yourself at least once per wave.
- `qa-engineer` runs the full tier before handoff; its report must name exact
  commands and exit codes.

### Correction loops

Use SDD's fix loop unchanged: resume the implementer for rounds 1–3, fresh eyes
one tier up for rounds 4–5, five rounds maximum per task, then adjudicate and
record a ruling. Minor findings go to the ledger as deferred; they never enter
the loop. The final whole-branch review gets exactly one fix wave.

### Integration

Commit on the delivery branch. Do not push or merge on your own initiative.
When the Product Owner chooses an integration option at Gate 2 — push, open a
PR, merge — execute it rather than handing it back as manual work. Force-push,
deploy, publish and production data changes still need one explicit
confirmation. Integration means the branch is coherent, the ledger is complete,
and the completion report states the branch, base and commit range.

## Gate 2 — Completion Report

One message, evidence first:

- **Acceptance criteria** — line by line, met or not, with the evidence.
- **Verification** — exact commands, exit codes, pass/fail counts, and what was
  not run. Include the final full run.
- **Review verdicts** — task, wave and final review outcomes; findings resolved.
- **Deferred and parked findings** — with the reason each is safe to defer.
- **Rulings** — every `Ruling:` in the ledger, with what it costs if wrong.
- **Changes** — commit range, files, migrations, config.
- **Integration options** — merge locally, open a PR, or keep the branch. Ask
  which one; if the Product Owner picks push, PR or merge, do it.

Ask with the `question` tool: `Accept`, `Send back for fixes`, `Reject`.

If `Send back`, dispatch one fix wave against the named findings, then re-issue
the report. Do not run a second full review unless the fix changed scope.

## Escalation

Stop and ask only for:

- a security-sensitive action;
- a destructive or irreversible operation;
- a production or external side effect;
- a requirements conflict where every path is a guess;
- a plan defect that invalidates the approved scope;
- a loop cap reached with a load-bearing finding.

Everything else is a ruling. Record it as
`Ruling: <decision> — <why> — <cost if wrong>` and keep going. A ruling that
never reaches the completion report was made in secret.

## Anti-patterns

- Starting implementation because the request "obviously" implies approval.
- Dispatching a full specialist roster for a small task.
- Accepting "tests pass" without a command and exit code.
- Parallel workers editing the same file to save wall-clock time.
- A second full review to rephrase the first one.
- Pushing, merging or deploying without being asked, or silently parking a
  finding.
