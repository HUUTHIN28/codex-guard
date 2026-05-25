# Workflows

Workflows are small orchestration units under `src/workflows`. The main orchestrator composes them into a full run, while CLI handlers can call some workflows directly for focused commands.

## Full Workflow

```text
createRun()
  -> PlanningWorkflow.execute()
  -> save plan
  -> ImplementationWorkflow or ParallelAgentWorkflow
  -> ReviewWorkflow
  -> optional RepairWorkflow loop
  -> SummaryWorkflow
  -> optional git commit
```

The full workflow is implemented in `src/orchestrator.ts`.

## Planning Workflow

File: `src/workflows/planning.workflow.ts`

Responsibilities:

- Load `.ai/prompts/planner.md`.
- Execute the Claude runner.
- Save raw output to `plan-raw-output.txt` for debugging.
- Parse JSON from runner output.
- Validate the result with `PlanSchema`.
- Save `plan.json` and human-readable `plan.md`.

Expected plan shape:

```json
{
  "summary": "Brief summary",
  "tasks": [],
  "files": [],
  "steps": [],
  "risks": []
}
```

Planning is the first step for full workflows and agent workflows. If it fails, later workflow phases do not run.

## Implementation Workflow

File: `src/workflows/implementation.workflow.ts`

Responsibilities:

- Convert a `Plan` into Codex instructions.
- Parse `FILE:`, `MODIFY:`, and `DELETE:` blocks.
- Apply generated changes to disk.
- Return changed files and errors.

This path is used when a run is not using parallel agents.

## Parallel Agent Workflow

File: `src/workflows/parallel-agent.workflow.ts`

Responsibilities:

- Load enabled agents from `.ai/agents/config.json`.
- Filter agents by requested roles.
- Run selected agents concurrently.
- Enforce ownership patterns before writing files.
- Write raw agent output to `agent-<role>.md`.
- Save `agents.json` manifest.

Agent output must use file blocks:

````markdown
FILE: docs/example.md
```markdown
content
```
````

or:

````markdown
MODIFY: README.md
```markdown
content
```
````

Deletes use:

```text
DELETE: path/to/file
```

## Review Workflow

File: `src/workflows/review.workflow.ts`

Responsibilities:

- Read the current git diff.
- Load reviewer prompt.
- Execute Claude review.
- Parse and validate review JSON.
- Return `ReviewResult`.

The orchestrator treats `PASS` as successful. Non-passing reviews can enter the repair loop until max retry is reached.

## Repair Workflow

File: `src/workflows/repair.workflow.ts`

Responsibilities:

- Group review issues by file.
- Ask Codex to generate targeted fixes.
- Apply returned changes.
- Report how many issues were fixed.

The orchestrator re-runs review after a repair attempt.

## Summary Workflow

File: `src/workflows/summary.workflow.ts`

Responsibilities:

- Inspect git status and diff.
- Ask Codex for a run summary.
- Ask Codex for a conventional commit message.
- Save `summary.md`.

When `AUTO_COMMIT` is enabled, the orchestrator uses the summary output to commit changes.

## Validation and Utility Workflows

Additional focused workflows include:

- `analyze.workflow.ts`: scans project characteristics and updates memory.
- `validate.workflow.ts`: runs available validation checks.
- `autofix.workflow.ts`: combines review results with automated fixes.
- `pr-summary.workflow.ts`: generates PR summaries from current changes.

## Failure Debugging

Use the run directory first:

```text
.ai/runs/<runId>/state.json
.ai/runs/<runId>/plan-raw-output.txt
.ai/runs/<runId>/agent-docs.md
.ai/runs/<runId>/agents.json
```

If `state.json` shows `planning` or `failed` and no `agents.json` exists, the agent phase did not start.
