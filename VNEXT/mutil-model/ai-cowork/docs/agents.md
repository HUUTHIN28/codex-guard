# Agents

`ai-cowork` supports configurable project agents for parallel work. Agent definitions live under `.ai/agents` and are loaded by `ProjectAgentConfigStore`.

## Default Agents

| Agent ID | Label | Ownership |
| --- | --- | --- |
| `code` | Code Agent | `src/**`, `lib/**`, `app/**`, `package.json`, `tsconfig.json` |
| `docs` | Docs Agent | `README*`, `CHANGELOG*`, `docs/**`, `**/*.md` |
| `tests` | Testcase Agent | `tests/**`, `__tests__/**`, `**/*.test.*`, `**/*.spec.*` |

Each agent has:

- `id`: stable role identifier.
- `label`: display name.
- `description`: short purpose.
- `enabled`: whether CLI selection can use it.
- `model`, `temperature`, `timeoutMs`: optional runner tuning.
- `ownershipPatterns`: file paths the agent may write.
- `ruleFile`: markdown rule file stored in `.ai/agents/rules`.
- `rules`: loaded prompt rules.

## Running Agents

```bash
# Run all enabled agents after planning
thin-cli --agents "Add user profile settings"

# Run only docs agent after planning
thin-cli --agent docs "Document user profile settings"
```

Important: agent runs still start with planning. A docs-agent command does not skip the planner. If the planner provider is not authenticated, the docs agent never starts.

## Ownership Enforcement

Before applying a file change, `ParallelAgentWorkflow` checks the target path against the selected agent's ownership patterns. If the path is outside the agent's ownership, the change is rejected and the agent result records an error.

This prevents common parallel-agent conflicts such as:

- Docs agent editing source files.
- Test agent editing implementation files.
- Multiple agents writing the same relative path in one parallel run.

## Output Format

Agents must return explicit file blocks so the workflow can parse and apply changes.

Create or replace a file:

````markdown
FILE: docs/example.md
```markdown
# Example

Content goes here.
```
````

Modify a file:

````markdown
MODIFY: README.md
```markdown
# Updated content
```
````

Delete a file:

```text
DELETE: docs/old-file.md
```

If the agent cannot safely perform a change, it should explain the reason in notes instead of writing outside ownership.

## Runtime Tracking

Agent runtime state is tracked by `src/agent-management`:

- `AgentRuntimeManager` starts agents, appends logs, updates status, and emits events.
- `AgentManagementStorage` persists events and snapshots.
- `AgentEventBus` broadcasts runtime events.
- Dashboard endpoints and websocket streams expose the state to the UI.

Persisted files:

```text
.ai/agent-management/events.jsonl
.ai/agent-management/snapshot.json
```

## Adding or Updating Agents

Use `ProjectAgentConfigStore` to upsert definitions when adding agents from code. Manual config changes should preserve the schema in `src/agents/agent-config.ts`.

When adding a new agent:

- Give it a unique `id`.
- Keep ownership patterns narrow.
- Add clear rules in `.ai/agents/rules/<id>.agent.md`.
- Decide whether it should run by default when `--agents` is used.
- Verify that file output uses supported `FILE:`, `MODIFY:`, and `DELETE:` blocks.
