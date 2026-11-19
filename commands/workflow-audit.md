---
description: Report workflow drift in the current repository without changing it
agent: plan
model: opencode-go/deepseek-v4.1-flash
---

Audit this repository against the global autonomous-delivery contract. Read
only. Do not edit, create, delete, commit, or run the build or test suite.

Check, in this order:

1. **Instruction files.** Is there an `AGENTS.md`? Is `CLAUDE.md` a symlink to
   it, absent, or a separate file that OpenCode silently ignores because
   `AGENTS.md` exists?
2. **Workflow profile.** Is there a `## Agent workflow profile` block? List the
   keys present and missing: `verify-fast`, `verify-full`,
   `never-autonomous`, `plan-home`, `status-home`, `design-system`,
   `parallel`, `worktree`.
3. **Command validity.** For each command in the profile, confirm the script or
   binary exists (`package.json`, `Makefile`, `pyproject.toml`, `justfile`).
   Do not execute it.
4. **Durable state.** Is `.superpowers/` ignored by git? Does `plan-home` and
   `status-home` exist if declared?
5. **Dead configuration.** Untracked `.opencode/package.json` pinning
   `@opencode-ai/plugin` 1.x, `.opencode/node_modules`, empty `.opencode/agents`,
   or a project `opencode.jsonc` that overrides the global model, agents, or
   reviewer permissions.
6. **Worktrees.** List `../.worktrees/<repo>/*` entries for this repository and
   flag ones whose branch is merged or gone.

Report a table with `status` (`ok`, `drift`, `missing`), `path:line`, the
concrete inconsistency, and the one-line fix. Then list the three highest-value
fixes in order. If everything matches, say so and name what you could not check.
Do not propose unrelated refactors.
