# Autonomous Delivery Workflow — Implementation Plan

> **For agentic workers:** executed inline by the controller. Steps use
> checkbox syntax for tracking. No product code is involved; each task's
> verification is a config or discovery check.

**Goal:** Add a portable, gate-driven orchestrator workflow on top of
Superpowers, then normalise the five active repositories to one instruction
contract.

**Spec:** `docs/superpowers/specs/2026-11-20-autonomous-delivery-design.md`

**Global constraints:** no push, merge, deploy, publish. Do not disturb
unrelated working-tree changes. Config changes apply without a restart.

---

## Task 1 — Orchestrator and specialists

**Files:** `agents/orchestrator.md`, `agents/qa-engineer.md`

- [x] Add `orchestrator` (primary) with a narrow subagent allowlist and no
      product-code ownership.
- [x] Add `qa-engineer` (subagent, read-only edits, shell allowed) that runs
      checks and reports evidence.
- [x] Verify: `opencode debug agents` lists both with the right mode/model.

## Task 2 — Composition skill and commands

**Files:** `skills/autonomous-delivery/SKILL.md`, `commands/deliver.md`,
`commands/workflow-audit.md`

- [x] Skill composes brainstorming → writing-plans → gates → SDD → final
      review → finishing; points at Superpowers, does not copy it.
- [x] `/deliver` pins the orchestrator and its model, submits the request.
- [x] `/workflow-audit` reports drift read-only.
- [x] Verify: skill registered (announced in the session) and command files
      discovered.

## Task 3 — Global configuration and policy

**Files:** `opencode.jsonc`, `managed-routing.jsonc`, `AGENTS.md`, `README.md`

- [x] Add `worktree.directory` and the global safety denies.
- [x] Mirror new agent routing in the managed source.
- [x] Amend worktree, subagent, verification and two-layer rules.
- [x] Pin Superpowers to 7.0.1.
- [x] Verify: `opencode debug config` shows the worktree setting; global denies
      appear on `build`, `orchestrator` and `qa-engineer`.

## Task 4 — Repository normalization

**Files:** five repository `AGENTS.md` files, `.gitignore` files, symlinks

- [x] `gcbc-website`: merge orphaned `CLAUDE.md` into `AGENTS.md`, symlink,
      replace the no-verify rule with tiers, ignore `.superpowers/`.
- [x] `ceremony-nextjs`: profile block, `parallel: off`, ignore `.superpowers/`.
- [x] `loving-god-network`: replace stale inline-only rule, profile block,
      remove dead V1 plugin package files.
- [x] `tentmakers-oms`: profile block, ignore `.superpowers/`.
- [x] Verify: each repo loads one instruction file and its profile parses.

## Task 5 — Retire the old harness and validate

- [x] Remove orphaned `~/.agent-harness` state and backups.
- [x] Run `opencode debug agents`, config parse, and confirm intended diffs.
- [ ] Live gate dry-run in a scratch repository (not executed; needs a model
      call and a real delivery).
- [ ] Reinstall the system routing override with sudo (requires the Product
      Owner; the source file is updated).

## Validation log

- `opencode debug agents`: `orchestrator` primary / DeepSeek Flash,
  `qa-engineer` subagent / DeepSeek Flash; subagent allowlist and global
  denies merged as intended.
- `opencode debug config`: `worktree.directory = ../.worktrees`.
- `managed-routing.jsonc`: valid JSON after comment stripping.
- Repository scripts checked against each profile: gcbc `qa:checks`/`qa:browser`
  exist; ceremony `lint` exists; lgn `lint`/`typecheck`/`build` exist;
  tentmakers `typecheck`/`check:ui-boundaries`/`test:*` exist.
- Not verified: nested `worktree_create` names (`<repo>/<type>/<slug>`), because
  creating a real worktree is a side effect.
- No commits were made; every repository already had unrelated uncommitted
  work.
