# OpenCode Config

Global OpenCode V2 configuration using both OpenCode Go and a ChatGPT subscription.

## Model routing

| Role | Agent | Model |
| --- | --- | --- |
| Interactive coding/planning | `build`, `plan` | `opencode-go/deepseek-v4.1-flash` |
| Delivery orchestration, independent QA | `orchestrator`, `qa-engineer` | `opencode-go/deepseek-v4.1-flash` |
| Exploration | `explore` | `opencode-go/gpt-6-luna#low` |
| Implementation | `general` | `opencode-go/deepseek-v4.1-flash` |
| Routine task review | `task-reviewer` | `openai/gpt-6-luna#medium` |
| Security/concurrency/recovery and final review | `reviewer` | `openai/gpt-6.1-sol#high` |
| Architecture/escalation | `architect` | `opencode-go/kimi-k3.1-pro#high` |

The root default is DeepSeek Flash. Pick Sol high or Kimi high by hand when a task needs escalation. Existing sessions retain their selected model. Use `/models` to select a model in an existing session if needed.

Reviews are read-only and report actionable findings, not successful-check essays. Re-reviews focus on fixes and affected dependencies; final branch review covers the full range. `/review-branch` explicitly selects Sol high.

## Delivery workflow

One entry point: `/deliver <objective>`. It selects `orchestrator` on DeepSeek Flash and loads the global `autonomous-delivery` skill.

1. **Delivery Proposal** — outcome, acceptance criteria, task table with dependencies and file ownership, specialists, worktree, risks. Nothing is built before approval.
2. **Execution** — one worktree at `../.worktrees/<repo>/<type>/<slug>`, per-task implementers (`general`), independent verification (`qa-engineer`), task/wave review (`task-reviewer`), final review (`reviewer`). Superpowers SDD owns the ledger and the bounded fix loop.
3. **Completion Report** — evidence, review verdicts, deferred findings, rulings, and the integration choice. The agent never pushes, merges a protected branch, deploys or publishes.

`/workflow-audit` reports repository drift read-only. Repositories declare their commands in the `## Agent workflow profile` block in `AGENTS.md`.

## Connections

Use `/connect` to connect **OpenCode Go** with its subscription API key. Connect **OpenAI** using **Sign in with ChatGPT** to use your ChatGPT plan, not OpenAI API billing. Usage allowances are separate; routing does not pool them or guarantee speed. No automatic paid API fallback is configured here.

Use `/models` to confirm availability. Configuration, agents and commands reload automatically; `/reload` forces a reload. No service restart is needed for these edits.

## Global versus project rules

`AGENTS.md` reserves model routing to this global configuration while repositories retain product, security, testing and architecture rules. Dispatch pinned agents without explicit model overrides.

This is an operating policy, not a technical lock: OpenCode project configuration and commands can override user-global defaults. Hard enforcement requires system-managed configuration. Do not silently rewrite project overrides; report conflicts first.

### System routing enforcement (macOS)

`managed-routing.jsonc` contains the approved system-level model defaults, compaction settings and reviewer permissions. The destination now exists on this machine, so back it up before reinstalling:

```sh
sudo cp -p "/Library/Application Support/opencode/opencode.jsonc" "/Library/Application Support/opencode/opencode.jsonc.bak-pre-superpowers-off"
sudo install -d -m 755 "/Library/Application Support/opencode"
sudo install -m 644 ~/.config/opencode/managed-routing.jsonc "/Library/Application Support/opencode/opencode.jsonc"
opencode reload
opencode debug agents
```

System configuration outranks project configuration for these settings. Reinstall after changing the routing table. This does not lock a session's explicit model selection or command/tool model overrides, prevent new agents, or make natural-language instructions immutable. It does not enforce OAuth versus API-key authentication; keep OpenAI connected through ChatGPT sign-in. Review any existing destination file before replacing it on another device.

## Other settings

- Superpowers plugin (`superpowers@git+https://github.com/obra/superpowers.git#v7.0.1`, V2 `plugins` key) providing its skills, pinned so upstream cannot silently change the workflow. `AGENTS.md` maps the abstract roles those skills name to the pinned agents above, so dispatch never relies on model inheritance.
- One-level subagent delegation; compaction starts with a 25% buffer and keeps 12,000 recent tokens.
- LSP and formatters enabled.
- Context7 documentation MCP integration.
- Native worktrees resolve to `../.worktrees`, matching the existing human-owned layout. The orchestrator may create one only inside an approved delivery.
- Global deny rules for push, PR merge, release, publish and deploy commands. The Product Owner can still run them by hand.

## Local files

Never commit API keys, provider credentials, machine-specific tokens or generated dependencies. Preserve unrelated local changes when updating this configuration.
