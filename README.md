# OpenCode Config

Reusable global OpenCode configuration for a Codex-led software engineering workflow with narrowly scoped subagents.

## Included

- Global LSP and formatter support
- Context7 documentation MCP integration
- One-level subagent delegation
- Read-only repository exploration and code review agents
- Narrow test and documentation assistants
- Read-only GitHub research agent
- A delegation policy that keeps architecture, security, production code, deployment, and final verification with the primary Codex agent

## Requirements

- OpenCode 1.18.32 or a compatible newer release
- An OpenAI provider connection with access to `openai/gpt-5.6-luna` and `openai/gpt-5.6-sol-fast`
- GitHub CLI authentication for `github-research` shell commands

If those model IDs are unavailable, replace the `model` values in `agents/*.md` with models supported by your account.

## Install On A New Device

Back up any existing global configuration, then clone this repository:

```bash
mv ~/.config/opencode ~/.config/opencode.backup
git clone git@github.com:Yrll21/opencode-config.git ~/.config/opencode
```

Start OpenCode once so it can recreate local package metadata if needed, then verify agent discovery:

```bash
opencode agent list
opencode debug config
```

OpenCode loads configuration at startup. Restart it after changing `opencode.jsonc`, agent definitions, or instruction files.

## Project Rules

Keep product- and repository-specific guidance in each project's `AGENTS.md`. This repository intentionally contains only reusable global defaults.

## Local Files

Generated package metadata, dependencies, machine backups, and secrets are ignored. Never commit API keys, provider credentials, or machine-specific tokens.
