# Changelog

All notable changes to the agystack configuration and agent orchestration rules are documented in this file.

## [2026-10-14] - Subagent Fan-Out Concurrency Cap

### Overview
Configured and enforced a maximum limit of **2 concurrent subagents per skill invocation** across the agystack plugin. This prevents excessive context and quota consumption while maintaining multi-model adversarial validation and isolated candidate exploration.

### Changes Made

#### 1. Model Configuration ([`rules/agystack-models.md`](file:///home/joecyu/sandbox/.agents/plugins/agystack/rules/agystack-models.md))
- Reduced runner and reviewer lists from 3 models (`inherit, flash, pro`) to 2 models (`inherit, flash`):
  - `arena runners`: `inherit, flash`
  - `arena cross-judge pool`: `inherit, flash`
  - `architect runners`: `inherit, flash`
  - `interrogate reviewers`: `inherit, flash`
- **Impact**: Skills whose concurrency is directly governed by model configuration lists ([`arena`](file:///home/joecyu/sandbox/.agents/plugins/agystack/skills/arena/SKILL.md), [`interrogate`](file:///home/joecyu/sandbox/.agents/plugins/agystack/skills/interrogate/SKILL.md), and [`architect`](file:///home/joecyu/sandbox/.agents/plugins/agystack/skills/architect/SKILL.md)) now deterministically launch exactly 2 subagents instead of 3.

#### 2. Orchestration Rule Invariant ([`rules/AGENTS.md`](file:///home/joecyu/sandbox/.agents/plugins/agystack/rules/AGENTS.md))
- Added the `Fan-Out Concurrency Cap (Max 2 Subagents)` invariant under `Mandatory Subagent Fan-Out`:
  - **`how`**: Restricted to at most 2 parallel exploration slices during Step 2a.
  - **`why`**: Restricted to querying at most 2 prioritized evidence categories concurrently.
  - **`reflect`**: Spawn exactly 2 reviewer subagents instead of 3.
  - **`swarm`**: Restricted total concurrent workers to $N \le 2$ per wave.
  - **`maintain-verification-skill`**: Batch feature file audits in waves of at most 2 concurrent workers.
  - **`arena` / `interrogate` / `architect`**: Capped runners and reviewers at 2.
- **Impact**: Overrides procedural defaults in individual `SKILL.md` instructions with a binding agent invariant.

### Verification
- Ran full test suite via `python3 -m unittest discover tests`.
- All 227 unit and integration tests passed cleanly (with 3 standard cloud skips).
