# Architecture

`ai-cowork` is a TypeScript CLI orchestrator for AI-assisted development workflows. It coordinates provider CLIs, project state, workflow steps, parallel agents, and a local dashboard around a single run directory under `.ai/runs`.

## High-Level Flow

```text
User task
  -> CLI argument parser or interactive menu
  -> Orchestrator
  -> Planning workflow
  -> Implementation or parallel agent workflow
  -> Review workflow
  -> Optional repair loop
  -> Summary workflow
  -> Optional git commit
```

The orchestrator owns the lifecycle. Individual workflows own one phase and communicate through typed objects such as `Plan`, `ReviewResult`, and agent task results.

## Core Modules

| Area | Files | Responsibility |
| --- | --- | --- |
| CLI | `src/cli.ts` | Parses flags, renders the terminal menu, dispatches command handlers. |
| Orchestration | `src/orchestrator.ts` | Coordinates run creation, planning, implementation, review, repair, summary, and commit. |
| Runners | `src/runners/*` | Provider boundary for Claude and Codex CLI execution. |
| Workflows | `src/workflows/*` | Isolated workflow phases for planning, implementation, agents, review, repair, validation, and summary. |
| State | `src/state.ts` | File-backed run state in `.ai/runs/<runId>`. |
| Agent management | `src/agent-management/*` | Runtime agent state, event stream, confirmations, persistence, and websocket support. |
| Agent config | `src/agents/*` | Project agent definitions, ownership rules, and rule files. |
| Dashboard | `src/dashboard.server.ts`, `src/dashboard.ui.ts` | Local HTTP, SSE, websocket API, and bundled UI. |
| Utilities | `src/utils/*` | Git helpers, logging, prompts, spinners, and shared helpers. |

## Provider Boundary

`createAllRunners()` creates Claude and Codex runners and assigns them to roles:

- Planner and reviewer use `ClaudeRunner`.
- Implementer, repairer, and summarizer use `CodexRunner`.
- Parallel agents use Codex runners, with the `code` agent reusing the implementer runner and other agents creating role-specific runners.

Workflow code should call runners through this boundary instead of spawning provider CLIs directly.

## State Model

Each run is stored below `.ai/runs/<runId>` and may include:

```text
state.json
plan.json
plan.md
review.json
summary.md
agents.json
agent-<role>.md
plan-raw-output.txt
```

`state.json` tracks status, task, timestamps, attempts, changed files, review data, and agent-management counters. The dashboard can read persisted state so it remains useful across processes.

## Workflow Contracts

Shared Zod schemas live in `src/types/index.ts`. Important contracts include:

- `PlanSchema`: summary, tasks, files, steps, risks, metadata.
- `ReviewResultSchema`: review status, score, issues, metadata.
- `RunStateSchema`: run lifecycle state and persisted run details.

Agent-management contracts have their own schemas in `src/agent-management/contracts.ts`.

## Prompt Loading

Prompt files live in `.ai/prompts` by default:

```text
planner.md
coder.md
reviewer.md
summarizer.md
docs-agent.md
testcase-agent.md
```

`loadPrompt()` centralizes prompt file lookup. Workflow classes pass prompt content to runner system prompts or inline prompt text depending on the provider.

## Dashboard Architecture

The dashboard server exposes read APIs, control endpoints, SSE, and websocket streams. It reads from the state manager and agent-management storage rather than relying only on in-memory state. This keeps historical runs visible even when the dashboard starts after a workflow has already finished.

## Design Guidelines

- Keep workflow logic in workflow modules, not the CLI.
- Keep provider-specific behavior inside runners.
- Keep persisted JSON backward-compatible where practical.
- Use Zod validation for data crossing module or persistence boundaries.
- Keep dashboard API responses stable and source them from state managers or storage.
