# Autonomous Delivery Workflow — Design

Date: 2026-11-20
Status: approved in chat, 2026-11-20

## Intent

The user is the Product Owner. A feature request, bug report, or engineering
objective goes in; one approved design + plan comes back; execution then runs
without routine checkpoints; one consolidated completion report comes back for
acceptance.

The workflow must behave identically across repositories while still honouring
each repository's stack, commands and conventions.

## Goals

- One global lifecycle, reused by every repository.
- Reuse Superpowers brainstorming, writing-plans, subagent-driven development,
  worktrees, TDD, review and verification instead of reimplementing them.
- Two human gates only: Delivery Proposal and Completion Report.
- Evidence-based completion, never a worker's self-report.
- Bounded review/correction loops with recorded rulings.
- Consistent behaviour across repositories.

## Non-goals

- A plugin or state-machine that enforces phases in code.
- Per-repo copies of the workflow.
- Automatic push, merge, deploy, publish, or production data changes.
- Reinstalling the `agent-harness` installer or adapter system.

## Constraints found

- OpenCode 2.3.2, Superpowers 7.0.1.
- `experimental.subagent_depth: 1`: only the top-level orchestrator dispatches.
- Subagents have no per-child working directory; parallel work is safe only on
  disjoint files in one worktree.
- OpenCode combines instruction files and does not resolve conflicts.
- `CLAUDE.md` is ignored in any directory that has an `AGENTS.md`.
- Existing worktrees live at `~/dev/.worktrees/<repo>/<type>/<slug>`.
- Repositories disagree on whether verification commands may run at all.

## Approaches considered

**A. Thin composition layer (chosen).** A global orchestrator agent, one
composition skill, a `/deliver` command, and specialist agents that reuse
existing pinned reviewers. Superpowers does the work; the new layer only adds
the two gates, specialist selection, wave planning and the completion report.

**B. Prompt-only.** Cheaper, but the gate and specialist selection drift across
compaction and long sessions.

**C. Plugin-enforced state machine.** Strongest enforcement, duplicates working
Superpowers behaviour and adds plugin API maintenance. Rejected.

## Decisions

1. Global layer owns workflow, roles, gates, safety and model routing. Project
   layer owns stack, commands, design system and durable-note locations.
2. Project instructions may declare verification *commands and tiers*, never
   permission to skip verification.
3. One canonical `AGENTS.md` per repository; `CLAUDE.md` and `GEMINI.md` are
   symlinks. gcbc's orphaned `CLAUDE.md` is merged into `AGENTS.md`.
4. One worktree per delivery at `../.worktrees`, named
   `<repo>/<type>/<slug>`. Parallelism comes from disjoint file ownership.
5. `ceremony-nextjs` runs `parallel: off`; other repositories allow waves.
6. Specialist roster: `architect`, `explore`, `general`, `qa-engineer` (new),
   `task-reviewer`, `reviewer`. Frontend/backend roles are task briefs, not
   agents. UI/UX review uses `task-reviewer` with the design-system brief.
7. `agent-harness` stays as source material; its installed state is not
   revived. Role prose may be reused; its installer and adapters are not.
8. Reviews are bounded by SDD: 5 fix rounds per task, one final fix wave.
9. Rulings are recorded in the SDD ledger and surfaced in the completion report.
10. Superpowers is pinned to 7.0.1.
11. Global denies: push, force-push, PR merge, release, publish, deploy.

## Verification tiers

| Tier | When | Examples |
| --- | --- | --- |
| fast | always, after a meaningful change | focused tests, changed-file lint, typecheck, `git diff --check` |
| full | before handoff, or when the profile allows it during work | whole suite, production build, browser E2E |
| never autonomous | requires explicit approval | deploy, prod migration, production data, emulator reset, index publication |

Repositories that forbid expensive checks map them to `full`; they do not
remove the fast tier or the evidence requirement.

## Open items

- System routing override must be reinstalled manually with sudo after
  `managed-routing.jsonc` changes.
- Repositories without a workflow profile fall back to discovery and record the
  assumed commands as a ruling.
