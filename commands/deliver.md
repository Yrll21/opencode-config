---
description: Run one delivery autonomously from an approved plan to a reviewed result
agent: orchestrator
model: opencode-go/deepseek-v4.1-flash
---

Load the `autonomous-delivery` skill before anything else, then treat the text
below as the Product Owner's request.

Produce the Delivery Proposal and stop for approval. Do not create a worktree,
edit files, or dispatch an implementer until the Product Owner approves.

$ARGUMENTS
