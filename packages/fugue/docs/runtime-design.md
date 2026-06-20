# Runtime Design

## Two execution modes

Canon operates in two modes that share the same skill definitions but differ in who executes them.

### Mode 1 — Claude Code (interactive)

The current mode. You invoke skills manually (`/breakdown`, `/write-task`, etc.) inside a Claude Code session. The live Claude instance is the runtime. Requires a human at the keyboard. Best for: planning sessions, architectural decisions, reviewing task docs before committing them.

### Mode 2 — Programmatic API (automated)

The canon runtime makes direct calls to the Claude API. No human required mid-run. Triggered via CLI or GitHub Actions webhook. Best for: fully automated runs, remote triggering, parallel fan-out, LangSmith observability.

**Both modes read the same `skills/*.md` files.** In Claude Code, they load as skill prompts. In API mode, the runtime loads them as system prompts for API calls. The skill definitions are the shared contract.

---

## Runtime stack

**Language:** TypeScript — consistent with the PlayByKey ecosystem, same `@langchain/anthropic` and `langsmith` packages already in use elsewhere.

**Dependencies:**
- `@langchain/anthropic` — Claude via LangChain (automatic LangSmith tracing)
- `langsmith` — token tracking, latency, cost per step
- `@anthropic-ai/sdk` — direct API access where needed
- `zod` — structured output validation for skill responses
- `@octokit/rest` — GitHub API for branch creation, file commits, PR opening

---

## Directory structure

```
canon/
└── runtime/
    ├── package.json
    ├── tsconfig.json
    ├── src/
    │   ├── index.ts           ← CLI entry point
    │   ├── orchestrator.ts    ← chains skills end-to-end
    │   ├── skills.ts          ← loads skills/*.md as system prompts
    │   ├── github.ts          ← branch creation, file commits, PR opening
    │   └── tools/
    │       ├── breakdown.ts   ← LangChain tool wrapping /breakdown skill
    │       ├── write-task.ts  ← LangChain tool wrapping /write-task skill
    │       ├── verify-task.ts ← LangChain tool wrapping /verify-task skill
    │       └── implement.ts   ← LangChain tool wrapping /implement skill
    └── .env.example
```

---

## Orchestration flow

```typescript
// orchestrator.ts (simplified)

async function run(taskDescription: string, targetRepo: string) {
  // Step 1: Breakdown
  const tasks = await breakdownTool.invoke({ task: taskDescription, repo: targetRepo });

  // Step 2: Write task docs (parallel)
  const taskDocs = await Promise.all(
    tasks.map(task => writeTaskTool.invoke({ task, repo: targetRepo }))
  );

  // Step 3: Verify (parallel)
  const gaps = await Promise.all(
    taskDocs.map(doc => verifyTaskTool.invoke({ doc }))
  );

  // Step 4: Generate tracker
  const tracker = await createTracker(taskDocs);

  // Step 5: Implement (sequential, respecting dependencies)
  for (const task of orderByDependency(tasks)) {
    await implementTool.invoke({ task, repo: targetRepo });
    // opens PR, waits for merge signal before next task
  }
}
```

LangSmith traces every `.invoke()` call automatically when `LANGCHAIN_TRACING_V2=true`.

---

## Token tracking

The runtime logs token usage per skill step via LangSmith callbacks:

```typescript
// Per workflow run, LangSmith captures:
{
  step: 'breakdown',
  input_tokens: 1243,
  output_tokens: 892,
  cost_usd: 0.0021,
  latency_ms: 1840
}
```

This gives visibility into exactly which step in the workflow is most expensive — the gap that the manual mode cannot measure.

---

## GitHub Actions trigger

```yaml
# .github/workflows/canon.yml
on:
  issues:
    types: [labeled]

jobs:
  run:
    if: github.event.label.name == 'canon'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm install
        working-directory: runtime
      - run: node src/index.ts --task "${{ github.event.issue.title }}" --repo "${{ github.repository }}"
        working-directory: runtime
        env:
          ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
          LANGCHAIN_API_KEY: ${{ secrets.LANGCHAIN_API_KEY }}
          LANGCHAIN_TRACING_V2: 'true'
          LANGCHAIN_PROJECT: 'canon'
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Opening a GitHub issue with the `canon` label and a task description in the title triggers the full workflow.

---

## Human-in-the-loop gates

The runtime pauses at two points for human review:

1. **After /breakdown** — the task list is posted as an issue comment. Human approves before write-task workers are spawned.
2. **After each /implement** — a PR is opened. Human reviews and merges before the next task begins.

These gates preserve the canon philosophy: AI executes, humans decide. Removing them would make canon fully autonomous — that is a deliberate non-goal.
