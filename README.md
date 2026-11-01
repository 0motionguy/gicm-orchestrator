# @gicm/orchestrator

Multi-agent orchestration primitives for TypeScript: an agent pool, task decomposition, a three-stage council and dependency-ordered workflows.

The library decides who gets a task, how a large task is split, how several answers are compared and in what order workflow steps run. It never calls a model itself: you provide an `AgentExecutor` that runs a task on an agent, so it fits any LLM stack. It is for developers building multi-agent systems who want that coordination logic as small, tested code.

Part of [gICM](https://github.com/0motionguy/gICM), a family of OpenClaw agent packages.

## What it does

- **Agent pool:** `AgentPool` registers agents (role, capabilities, trigger words, keywords, concurrency cap) and hands one out by `round-robin`, `least-loaded` or `best-match` scoring, tracking status and success and duration stats.
- **Task decomposition:** `TaskDecomposer` has eight built-in patterns (multi-file, step-by-step, feature-impl, bug-fix, refactor, api-integration, security, review) and a generic fallback. It scores complexity from 1 to 10 and builds a dependency graph with parallel groups.
- **Council:** `Council.deliberate` runs three stages among at least two members: independent answers with a confidence value, anonymised peer review and ranking, then synthesis by the top-ranked member.
- **Consensus and synthesis:** ranking helpers aggregate scores, detect conflicts and set a consensus level from unanimous to split; the level selects one of four synthesis strategies (best-of-n, merge-top, consensus-filter, debate-resolution).
- **Workflows:** `Orchestrator` registers workflows, orders steps with a topological sort, runs them as parallel, sequential, conditional or vote steps, retries failed tasks with a delay, supports cancel, and offers `executeDynamic` (decompose a task, then run the pieces).
- **Config and events:** zod-validated settings (concurrency, retries, consensus threshold, timeout) and typed events such as `task:completed` and `consensus:reached`.

## Example

```ts
import { AgentPool, TaskDecomposer, Orchestrator } from "@gicm/orchestrator";

const pool = new AgentPool({ loadBalancing: "best-match" });
pool.registerAgent({
  id: "a1", name: "Reviewer", role: "reviewer",
  capabilities: ["review"], triggers: ["review"], keywords: ["pull request"], maxConcurrent: 2,
});

// Your code: run the task on a model and return a TaskResult.
const executor = { execute: (agent, task) => runOnYourModel(agent, task) };

const orchestrator = new Orchestrator(pool, new TaskDecomposer(), executor);
const results = await orchestrator.executeDynamic({
  id: "t1", description: "Fix the login bug", priority: "high",
});
```

## How it's built

TypeScript (ESM) bundled with tsup and tested with vitest; zod validates configuration. Every class is an `EventEmitter`, and the only integration point is the injectable `AgentExecutor` interface.

## Repository layout

- `src/`: `agent-pool`, `task-decomposer`, `council`, `ranking`, `synthesis`, `orchestrator`, `types`
- `src/__tests__/`: vitest suite (32 test cases)
- `skill/SKILL.md`: the OpenClaw skill definition

## Run it locally

```bash
pnpm install
pnpm build   # tsup -> dist/
pnpm test    # vitest (add --run for a single pass)
```

## Status

Beta. Version 1.0.0, a single extracted release (February 2026) with a test suite. No model integration, queue or persistence is included; results live in memory.
