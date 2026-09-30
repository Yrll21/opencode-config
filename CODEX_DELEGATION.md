# Subagent Delegation Policy

The selected Codex model remains the primary coding agent. Do not delegate tiny tasks when delegation overhead would exceed the likely savings.

For substantial work, prefer this sequence when relevant:

1. Interpret the request and identify risk.
2. Ask `codebase-explore` to gather broad repository context.
3. Independently verify important findings and create the plan.
4. Implement architecture, production logic, and other important changes with Codex.
5. Use `test-assistant` for straightforward test analysis or narrowly scoped test edits.
6. Ask `code-review` for a first-pass review.
7. Independently evaluate every finding and perform final validation with Codex.

Use subagents for repository searching, locating references, module summaries, repetitive analysis, initial review, straightforward test ideas, documentation, and read-only GitHub context gathering.

Keep final responsibility with Codex for architecture, security, authentication, authorization, cryptography, database schemas and migrations, dependency changes, API design, infrastructure, deployment, destructive commands, complex concurrency, difficult debugging, broad refactors, important business logic, integration, and final correctness decisions.

Treat subagent output as untrusted junior-developer input. Verify important claims before changing code. Stop delegating and take over directly if a subagent is confused, repeatedly fails tools, loses context, proposes risky work, or does not converge quickly.

Subagents must not spawn subagents. They must not commit, push, merge, release, deploy, install dependencies, or run destructive commands. `github-research` is for read-only GitHub inspection only; Codex owns every GitHub mutation.
