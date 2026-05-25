# CLI Usage

`thin-cli` is the main command exposed by this package. The package also exposes `ai-dev-orchestrator` as an alias in `package.json`.

Run commands from the `ai-cowork` directory during development:

```bash
npm install
npm run build
node dist/cli.js --help
```

If linked globally with `npm run link`, use `thin-cli` directly.

## Common Commands

```bash
# Open the interactive terminal menu
thin-cli

# Run a full workflow
thin-cli "Add user profile settings"

# Generate a plan only
thin-cli --plan "Add user profile settings"

# Run all enabled parallel agents
thin-cli --agents "Add user profile settings"

# Run a single agent
thin-cli --agent code "Add user profile settings"
thin-cli --agent docs "Document user profile settings"
thin-cli --agent test "Add tests for user profile settings"

# Ask a read-only project question
thin-cli --chat
thin-cli --chat "Which file handles the dashboard API?"

# Review current git changes
thin-cli --review

# Replay a previous run
thin-cli --replay run-2026-05-25T09-58-08-uyx86e

# List historical runs
thin-cli --list

# Start dashboard
thin-cli --dashboard --dashboard-port 4317
```

## Modes

The CLI supports execution modes through `--mode <mode>`. Mode labels are rendered in the CLI UI and influence workflow behavior where handlers support them.

Common user-facing modes include:

- Full auto: run the workflow with minimal prompts.
- Planning or review-focused commands: use explicit flags such as `--plan` and `--review`.
- Agent commands: use `--agents` or `--agent <role>`.

## Agent Role Aliases

`--agent` accepts enabled agent IDs and a few aliases:

| Input | Resolved role |
| --- | --- |
| `code`, `coder`, `dev` | `code` |
| `docs`, `doc`, `documentation` | `docs` |
| `test`, `tests`, `testcase`, `testcase-agent` | `tests` |
| `all` | all enabled agents |

Unknown roles fail early and print the available configured agent IDs.

## Configuration

Configuration is loaded through `src/config.ts` and environment variables. Useful variables:

| Variable | Default | Purpose |
| --- | --- | --- |
| `CLAUDE_CLI` | `claude` | Command used by Claude runner. |
| `CLAUDE_MODEL` | `opus` | Claude model label passed through runner config. |
| `CODEX_CLI` | `codex` | Command used by Codex runner. |
| `CODEX_MODEL` | `gpt-4o` | Codex model label passed through runner config. |
| `MAX_RETRY` | `3` | Max review and repair attempts. |
| `TIMEOUT_MS` | `120000` | Runner command timeout in milliseconds. |
| `AUTO_COMMIT` | `true` | Whether successful full workflows commit automatically. |

## Troubleshooting

### Planner Fails With Missing `tasks`

If a run fails with:

```text
Failed to parse plan: response missing 'tasks' field
```

open the saved `plan-raw-output.txt`. If it contains:

```text
Not logged in · Please run /login
```

the planner did not return a plan. Log in to Claude Code with `/login` or configure `CLAUDE_CLI` to a working command, then rerun the workflow.

### Docs Agent Does Not Start

Full workflows always run planning first. If planning fails, no implementer or parallel agent has started yet. Use `.ai/runs/<runId>/state.json` and `plan-raw-output.txt` to confirm the failed phase.

### Sandbox or Network Failures

Provider CLIs and package installers may need network access or access outside the project directory. Re-run the command in an environment where the provider CLI is authenticated and allowed to reach its service.
