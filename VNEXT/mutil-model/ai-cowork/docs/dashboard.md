# Dashboard

The dashboard provides a local web view for runs, agent state, events, settings, and confirmations. It is served by `src/dashboard.server.ts`, with bundled UI from `src/dashboard.ui.ts`.

## Start The Dashboard

```bash
npm run build
node dist/cli.js --dashboard --dashboard-port 4317
```

Open:

```text
http://127.0.0.1:4317
```

If the port is busy, pass another value:

```bash
node dist/cli.js --dashboard --dashboard-port 4328
```

## Main Capabilities

- View run overview and historical runs.
- Inspect individual run state.
- See raw planner output when parsing fails.
- Watch agent-management events.
- View pending confirmations.
- Stop active runs through the API.
- Stream runtime updates through SSE or websocket connections.

## HTTP Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/` | Serve dashboard UI. |
| `GET` | `/api/overview` | Return run and system overview data. |
| `GET` | `/api/events` | Stream general events with SSE. |
| `GET` | `/api/settings` | Return dashboard and project settings. |
| `GET` | `/api/runs/:runId` | Return one run, including saved artifacts where available. |
| `POST` | `/api/runs/:runId/stop` | Request stop for a run. |
| `GET` | `/api/agent-management` | Return current agent-management snapshot. |
| `GET` | `/api/agent-management/events` | Return or stream agent-management events. |
| `GET` | `/api/agent-management/confirmations` | Return pending confirmations. |
| `POST` | `/api/agent-management/confirmations/:id` | Resolve a pending confirmation. |
| `WS` | `/ws/agent-events` | Websocket stream for agent events. |

## Data Sources

The dashboard should use persisted state as the source of truth:

- Runs come from `StateManager` and `.ai/runs`.
- Agent events and snapshots come from `AgentRuntimeManager` and `AgentManagementStorage`.
- Confirmation state comes from agent-management helpers.

This makes the dashboard useful even if it starts after a workflow has already created run artifacts.

## Debugging Runs

For a failed planning run, inspect:

```text
.ai/runs/<runId>/state.json
.ai/runs/<runId>/plan-raw-output.txt
```

For a parallel-agent run, inspect:

```text
.ai/runs/<runId>/agents.json
.ai/runs/<runId>/agent-code.md
.ai/runs/<runId>/agent-docs.md
.ai/runs/<runId>/agent-tests.md
```

The dashboard run detail endpoint also exposes raw plan output when present.

## Quick API Checks

After starting the dashboard:

```bash
node -e "fetch('http://127.0.0.1:4317/api/overview').then(r => r.json()).then(console.log)"
node -e "fetch('http://127.0.0.1:4317/api/agent-management').then(r => r.json()).then(console.log)"
```

If localhost access is blocked in the current environment, validate with build/typecheck and inspect the server logs instead.

## Implementation Notes

- Keep API response shapes stable.
- Avoid using mock data as the source of truth.
- Do not make dashboard behavior depend only on in-memory state.
- Add new endpoints in `dashboard.server.ts`.
- Keep UI changes in `dashboard.ui.ts` unless extracting a new local asset or helper is intentional.
