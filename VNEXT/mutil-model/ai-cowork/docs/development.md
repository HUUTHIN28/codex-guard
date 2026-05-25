# Development

This guide covers local development practices for `ai-cowork`.

## Setup

```bash
cd ai-cowork
npm install
npm run build
node dist/cli.js --help
```

The project uses TypeScript and emits compiled JavaScript to `dist`.

## Useful Scripts

| Command | Purpose |
| --- | --- |
| `npm run build` | Compile TypeScript. |
| `npm run dev` | Compile and run the CLI. |
| `npm run cli` | Compile and run the CLI. |
| `npm run dashboard` | Compile and start dashboard mode. |
| `npm test` | Run compiled Node test files. |
| `npm run watch` | Run TypeScript in watch mode. |
| `npm run link` | Link CLI binaries globally. |

## Project Layout

```text
src/
  agent-management/  runtime state, events, storage, confirmations
  agents/            configurable project agent definitions
  chat/              read-only project chat helpers and token usage
  runners/           Claude and Codex runner boundary
  workflows/         planning, implementation, review, repair, summary, agents
  utils/             git, prompts, logging, spinners, helper functions
  cli.ts             terminal entry point
  orchestrator.ts    full workflow coordinator
  dashboard.*        dashboard server and UI
  state.ts           file-backed run state
  types/             shared Zod schemas and TypeScript types
```

## Coding Guidelines

- Keep changes scoped to the module that owns the behavior.
- Put provider-specific execution details in `src/runners`.
- Put phase-specific behavior in `src/workflows`.
- Validate persisted or cross-module data with Zod schemas.
- Preserve backward compatibility for saved run and agent-management files when practical.
- Prefer small helper functions over large mixed-responsibility modules.
- Do not edit generated `dist` files manually.

## Documentation-Only Changes

For docs-only tasks:

- Write under `docs/**` or existing markdown files.
- Do not modify `src/**`, tests, generated files, or run artifacts.
- Verification can usually be limited to `git diff --check` and a path-focused status review.
- Build is optional unless the docs mention generated API or CLI output that must be checked against compiled behavior.

## Provider CLI Requirements

Full workflows require provider CLIs:

- Claude CLI for planning and review.
- Codex CLI for implementation, repair, summary, and parallel agents.

If Claude is not logged in, planning may produce raw output similar to:

```text
Not logged in · Please run /login
```

In that case, authenticate Claude Code and rerun the workflow. A docs-agent workflow still needs planning first unless the code is changed to support planless agent execution.

## Adding Workflow Features

When adding a workflow feature:

1. Define or update a typed contract in `src/types/index.ts` or the relevant module.
2. Add workflow logic under `src/workflows`.
3. Keep runner calls behind `ClaudeRunner` or `CodexRunner`.
4. Persist run artifacts through `StateManager` or workflow save helpers.
5. Surface user-facing behavior through `src/cli.ts` or `src/orchestrator.ts`.
6. Update docs and tests for the new behavior.

## Adding Dashboard Features

When adding dashboard behavior:

- Add or adjust server endpoints in `src/dashboard.server.ts`.
- Read data through state or agent-management managers.
- Keep response shapes stable and JSON-friendly.
- Update `src/dashboard.ui.ts` for the browser UI.
- Test with a local dashboard port and quick API fetch checks.

## Verification Checklist

For source changes:

```bash
npm run build
node dist/cli.js --help
```

For dashboard changes:

```bash
node dist/cli.js --dashboard --dashboard-port 4328
node -e "fetch('http://127.0.0.1:4328/api/overview').then(r => r.json()).then(console.log)"
```

For docs-only changes:

```bash
git diff --check -- docs
git status --short -- docs
```

## Commit Scope

Because this repository can live inside a larger git worktree, commit only the files relevant to the task:

```bash
git add docs/architecture.md docs/cli-usage.md docs/workflows.md docs/agents.md docs/dashboard.md docs/development.md
git commit -m "docs: add project documentation"
```
