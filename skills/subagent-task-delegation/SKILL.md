---
name: subagent-task-delegation
description: Spawns background worker subagents for parallel task execution.
---

# Subagent Task Delegation

## Overview
Deploys isolated subagent instances to execute long-running builds, broad codebase research, or test permutations concurrently without polluting the primary context window.

## Key Capabilities
- Background execution with non-blocking lifecycle management.
- Epistemic context isolation between parent and worker agents.
- Structured synthesis of subagent deliverables into primary workflows.
