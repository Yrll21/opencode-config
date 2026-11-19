# Global agent operating rules

## Git worktrees (every repository)

- Work happens in the current checkout unless the Product Owner asked for a
  worktree or approved a delivery whose proposal names one.
- Inside an approved `/deliver` run, the `orchestrator` may create one worktree
  at `../.worktrees/<repo>/<type>/<slug>` and move the session into it. That is
  the only standing authority to change the worktree layout.
- Respect existing worktrees and sessions. Removing, renaming, moving or pruning
  existing worktrees requires explicit user approval; never discard unrelated
  changes.
- Follow repository rules for branching, checks, commits and merges.

## How I work (every repository)

Project instruction files win on stack, commands, design system and repository
conventions. They do not override the delivery gates, the evidence requirement
or the safety floor below.

- Size the task first. Small and clear (a copy fix, a rename, a
  one-liner): do it, then verify it. One behaviour or component: a short
  plan in chat, then build once I approve. Multi-part work, or anything
  touching a shared contract: use the Plan agent (`plan`) and a written
  plan, then build.
- No brainstorming round or written plan for work that fits a chat plan.
  Make the small calls yourself and list them.
- A bug fix or behaviour change starts with a failing test where the repo
  has tests. Confirm it fails for the reason you expect, then fix.
- Debug to the root cause before changing code. One hypothesis and one
  change at a time.
- Do not say done, fixed or passing unless you ran the command that proves
  it in the same turn. Say what was not checked.
- Before proposing a PR, run one fresh-context review of the diff with the
  `reviewer` subagent or `/review-branch` and fix what matters.
- One task per session. When I start an unrelated task in a long session,
  say that a fresh session would be cheaper.

### Verification tiers

- **Fast, always:** focused tests for the changed surface, changed-file lint,
  typecheck, `git diff --check` where they exist.
- **Full, before handoff:** the whole suite, production build, browser E2E, when
  the repository's workflow profile allows them.
- **Never autonomous:** deploy, production migration, production data, emulator
  reset, index publication. Explicit approval required.
- A repository may move a check between fast and full. It may not remove the
  fast tier or the rule that completion needs fresh evidence.

## Subagents (every repository)

Subagent fan-out is where most extra usage goes.

- Work inline by default, including executing an approved plan that did not come
  through `/deliver`.
- Inside an approved `/deliver` run, the `orchestrator` delegates per the
  `autonomous-delivery` skill: one implementer per task, a review per task or
  wave, and a final whole-branch review. That workflow replaces the inline
  default and the no-pairing rule for that run only.
- Outside that run, dispatch only for: read-only mapping or search across a large
  area (prefer the built-in `explore` agent), two or more pieces of work that are
  truly independent, and the one pre-PR review.
- 2-4 agents at most for ad-hoc fan-out. Do not use a model below mid-tier for
  review.
- A subagent's report is a claim. Check the diff.

## Autonomous delivery (every repository)

Entry point: `/deliver`, which selects the `orchestrator` on the mid tier.

1. `/deliver` loads the `autonomous-delivery` skill and produces a Delivery
   Proposal. No worktree, no edits, no implementer before approval.
2. Approved proposal → worktree → per-task implementation and review → final
   review → Completion Report.
3. Two human gates only: proposal and completion. Routine decisions are rulings,
   recorded in the ledger and surfaced in the report.
4. Escalate only for a security-sensitive action, a destructive or irreversible
   operation, a production or external side effect, a requirements conflict where
   every path is a guess, a plan defect that invalidates scope, or a loop cap with
   a load-bearing finding.
5. Never push, merge a protected branch, deploy, publish, or mutate production
   data, even after approval. Present the integration options at the second gate.
6. Bounded loops: five fix rounds per task, exactly one final fix wave.
   Unresolved findings are reported, never silently parked.

## Project workflow profile (every repository)

Each repository has exactly one instruction file, `AGENTS.md`. `CLAUDE.md` and
`GEMINI.md` symlink to it; a separate `CLAUDE.md` is invisible to OpenCode when
`AGENTS.md` exists.

Repositories declare their contract in a fixed block:

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

The orchestrator reads this block before planning. Missing keys fall back to
discovery and are recorded as a ruling. `/workflow-audit` reports drift.

## Model routing ownership (every repository)

- The user owns model/provider routing globally in `~/.config/opencode/opencode.jsonc`, the managed source `~/.config/opencode/managed-routing.jsonc`, and the installed system override. Repository `AGENTS.md`, plans, skills and prompts must defer to this routing policy; they may define product rules and review scope, not substitute providers, models or reasoning variants.
- Dispatch the named agent without a `model` override so its explicitly configured model is used. This is not inheritance: each worker/reviewer has a pinned global model. Only a direct user request authorizes a model override or routing change. If an agent/model is unavailable, report it; never silently switch providers or use a paid API fallback.
- Do not create or modify project OpenCode configuration, agent definitions or commands to override global routing. If an existing project definition conflicts, report it and ask before editing it. OpenCode technically allows project settings to override user-global settings; this instruction is an operating policy, not a managed configuration lock.
- ChatGPT (`openai/`): routine task reviews use Luna medium; high-risk and final whole-branch reviews use Sol high. OpenCode Go (`opencode-go/`): default coding and planning use DeepSeek Flash, delivery orchestration and independent QA use DeepSeek Flash, exploration uses Luna low, and architecture/escalation uses Kimi high. Use ChatGPT subscription sign-in, not an OpenAI API-key fallback.

### Review scope and output

- Keep required reviews; choose `reviewer` directly for high-risk work rather than paying for a routine review first.
- Report actionable findings with severity, `path:line`, concrete impact and a short fix direction. No praise, successful-check essays, plan repetition or speculative style suggestions. If clean, say `No actionable findings.` and name any material verification limitation briefly.
- Pass the exact diff range, task acceptance criteria, relevant paths and previous finding IDs. Re-review fixes and affected dependencies only unless scope changed or a concrete new risk requires expansion. Final whole-branch review still covers the full requested range.
- Concise output does not reduce security coverage, required verification or read-only restrictions. Do not launch duplicate reviews just to rephrase the same conclusion.

## Superpowers skill dispatch (every repository)

Superpowers skills name abstract roles ("cheap model", "most capable available model", `agent: "general"`). Map them to these OpenCode agents, which carry the tier in their own config:

| Skill says | Dispatch to |
| --- | --- |
| delivery lifecycle | `orchestrator` (only through `/deliver`) |
| explore, search, "understand this codebase" | `explore` |
| implementer / `Subagent (general-purpose)` | `general` |
| independent test, type, lint, build, E2E evidence | `qa-engineer` |
| task-scoped review (one task's diff) | `task-reviewer` |
| security, authorization, concurrency, transactions, idempotency, recovery or migrations review | `reviewer` |
| final whole-branch review before merge | `reviewer` |
| architecture, design, fix rounds 4-5 | `architect` |

- This table overrides the dispatch line inside a skill's template; the skill's model guidance still decides *which* tier a task earns.
- Prefer `/review-branch` for the final whole-branch review. It pins the `reviewer` agent and its escalation model, so the one review that must never run cheap cannot drift.
- Never leave a dispatch to model inheritance when the skill asks for a tier. Use the pinned agents above; if no configured agent fits, ask rather than inventing a model override.
- Keep the mid tier (`general`) as the floor for reviewers and for implementers working from prose. The cheapest tier is for mechanical work with the code already written in the plan.
- Outside an approved `/deliver` run, the "Subagents" limits still bind: work
  inline by default, and the skill does not authorize an implementer-plus-
  reviewer pair per plan step.
- Subagents cannot launch subagents (`experimental.subagent_depth: 1`), so this mapping applies to the top-level controller.
