# Task 08 — Orchestrator

## Outcome

`packages/fugue/runtime/src/orchestrator.ts` exports `runFugue(taskDescription: string, repo: string): Promise<void>` that executes the full canon skill chain: breakdown → parallel write-task → parallel verify-task → delegate implement (branch + task doc commit, then pause). The function logs token usage per step to stdout and pauses at two human-in-the-loop gates by polling GitHub for an approval comment before continuing.

## Why

The orchestrator is the core product. It is what makes Fugue an agent rather than a collection of functions. Everything built in Tasks 01-07 is plumbing — this is where it gets wired together into a workflow that runs autonomously from a single call.

## Context

### Repository state

Tasks 01-07 are complete. All tools and clients exist:
- `src/tools/breakdown.ts` — `runBreakdown()`
- `src/tools/write-task.ts` — `runWriteTask()`
- `src/tools/verify-task.ts` — `runVerifyTask()`
- `src/github.ts` — `getDefaultBranch`, `createBranch`, `commitFile`, `openPR`, `postComment`

### Human-in-the-loop mechanism

**Polling model:** After posting to GitHub (issue comment or PR), the orchestrator polls every 30 seconds for a specific approval signal, up to a configurable timeout (default: 24 hours).

**Gate 1 — After breakdown:**
1. Run `runBreakdown()` → get task list
2. Post task list as a comment on the triggering issue (formatted as a checklist)
3. Poll the issue for a comment containing exactly `approved` from the repo owner
4. On approval: proceed to write-task fan-out

**Gate 2 — After each implement delegate:**
1. Create branch, commit task doc, open draft PR
2. Post comment on the PR: "Fugue has set up this task. Please implement and mark PR ready for review."
3. Poll the PR for merge (check `merged` status)
4. On merge: mark task complete, proceed to next task

### Dependency ordering

Tasks from `runBreakdown()` include a `dependsOn` array. Order implementation by resolving dependencies: tasks with no dependencies run first (in parallel), then tasks whose dependencies are all merged, and so on.

Simple topological sort — no need for a complex library.

### OrchestrationContext

All state for a single `runFugue()` call is held in one typed object. Stable fields come first so prompt builders can reliably put cacheable content at the head of every message array.

```typescript
interface OrchestrationContext {
  // Stable across the entire run — put these first in every prompt
  skills: Record<SkillName, string>;         // loaded once from skills/*.md
  scaffoldFiles: Record<string, string>;     // README.md, CLAUDE.md, package.json from target repo

  // Derived once at breakdown, read-only after
  intent: string;                            // original task description
  repo: string;                              // "owner/repo"
  issueNumber: number;

  // Mutable — updated as steps complete
  tasks: Task[];
  taskDocs: TaskDoc[];
  verifiedTasks: VerifiedTask[];

  // Rate limiting and retry config
  limits: {
    concurrency: number;          // max parallel calls per fan-out chunk (default: 4)
    maxRetries: number;           // per-item retry ceiling (default: 3)
    interChunkDelayMs: number;    // delay between chunks to stay under RPM (default: 500)
  };

  // Items that exhausted retries — surfaced at Gate 1, not silently dropped
  failures: Array<{
    taskId: string;
    step: string;
    error: string;
    retryCount: number;
  }>;

  // Observability
  langsmithRunId: string;
  stepTimings: Record<string, number>;       // step name → elapsed ms
}
```

`SkillName`, `Task`, `TaskDoc`, and `VerifiedTask` are defined in `src/types.ts` (Task 04).

### Fan-out with retry

`Promise.all()` with no limit fires N calls simultaneously and fails the entire batch on any single error. Use `fanOut()` instead — no extra dependency needed:

```typescript
type FanOutFailure<T> = { item: T; error: Error; retryCount: number };

async function fanOut<T, R>(
  items: T[],
  fn: (item: T) => Promise<R>,
  ctx: Pick<OrchestrationContext, 'limits'>,
): Promise<{ results: R[]; failures: FanOutFailure<T>[] }> {
  const { concurrency, maxRetries, interChunkDelayMs } = ctx.limits;
  const results: R[] = [];
  const failures: FanOutFailure<T>[] = [];

  for (let i = 0; i < items.length; i += concurrency) {
    const chunk = items.slice(i, i + concurrency);

    // Track retry state per item in this chunk
    let pending = chunk.map(item => ({ item, retryCount: 0 }));

    while (pending.length > 0) {
      const outcomes = await Promise.allSettled(pending.map(p => fn(p.item)));
      const nextPending: typeof pending = [];

      for (let j = 0; j < outcomes.length; j++) {
        const outcome = outcomes[j];
        const p = pending[j]!;

        if (outcome.status === 'fulfilled') {
          results.push(outcome.value);
        } else {
          const err = outcome.reason as Error;
          const retryAfterMs = extractRetryAfterMs(err); // read Retry-After header if present
          if (p.retryCount < maxRetries) {
            if (retryAfterMs > 0) await sleep(retryAfterMs);
            nextPending.push({ item: p.item, retryCount: p.retryCount + 1 });
          } else {
            failures.push({ item: p.item, error: err, retryCount: p.retryCount });
          }
        }
      }

      pending = nextPending;
    }

    // Inter-chunk delay to stay under RPM
    if (i + concurrency < items.length) await sleep(interChunkDelayMs);
  }

  return { results, failures };
}
```

`extractRetryAfterMs` reads the `retry-after` header from the error if it is a 429 — use this rather than a fixed backoff. `sleep` is `(ms: number) => new Promise(r => setTimeout(r, ms))`.

Apply `fanOut` to write-task and verify-task. The default concurrency is 4 (`ctx.limits.concurrency`). LangSmith traces will show whether that ceiling needs adjustment.

### Input overflow handling

If the serialized `OrchestrationContext` grows large enough to overflow the model's context window, prompt construction must downgrade to a compact form before sending — not after receiving a context-length error.

The orchestrator supports two serialization modes:

- **`full`** — includes file contents and task doc bodies (used by default)
- **`compact`** — includes only file names and task IDs (used as fallback)

Prompt builders must estimate token count before invoking. A rough estimate is `Math.ceil(text.length / 4)` (characters to tokens). If the estimate exceeds a configurable ceiling (e.g., 150,000 tokens), downgrade to compact mode and log a warning:

```
[fugue] context too large for full mode (est. 182,400 tokens) — using compact
```

The compact form is never used for skill prompts (those are always included in full). Only the `scaffoldFiles` content and task doc bodies are omitted.

### Token logging

After each LLM call (breakdown, write-task × N, verify-task × N), log to stdout and record in `stepTimings`:

```
[fugue] breakdown: 1243 input, 892 output tokens (1840ms)
[fugue] write-task task-01: 2100 input, 1450 output tokens (2310ms)
...
```

LangChain's callback system provides token counts. Use `response.usage_metadata` on the LLM response. Record elapsed ms with `performance.now()` and store in `ctx.stepTimings[stepName]`.

### Orchestrator flow (simplified)

```typescript
export async function runFugue(taskDescription: string, repo: string): Promise<void> {
  // 1. Breakdown
  const codebaseSummary = await summarizeRepo(repo);
  const tasks = await runBreakdown(taskDescription, codebaseSummary);

  // 2. Gate 1: post task list, wait for approval
  // Include any retry-exhausted failures in the comment so the human sees them before approving
  await postIssueComment(repo, issueNumber, formatTaskList(tasks, ctx.failures));
  await waitForApproval(repo, issueNumber);

  // 3. Write-task (parallel, capped at ctx.limits.concurrency)
  const { results: docs, failures: writeFails } = await fanOut(tasks, runWriteTask, ctx);
  ctx.failures.push(...writeFails.map(f => ({ taskId: f.item.id, step: 'write-task', error: f.error.message, retryCount: f.retryCount })));

  // 4. Verify-task (parallel, capped at ctx.limits.concurrency)
  const { results: verifyResults, failures: verifyFails } = await fanOut(docs, runVerifyTask, ctx);
  ctx.failures.push(...verifyFails.map(f => ({ taskId: f.item.id, step: 'verify-task', error: f.error.message, retryCount: f.retryCount })));

  const lowConfidence = verifyResults.filter(r => r.confidence === 'LOW');
  if (lowConfidence.length > 0 || ctx.failures.length > 0) {
    // Post gaps and exhausted-retry failures as issue comment, then halt
    await postIssueComment(repo, issueNumber, formatGaps(lowConfidence, ctx.failures));
    return;
  }

  // 5. Implement: delegate model (branch + commit doc + open PR per task)
  for (const ordered task of topologicalSort(tasks, docs)) {
    const { sha } = await getDefaultBranch(repo);
    const branch = `fugue/${task.filename.replace('.md', '')}`;
    await createBranch(repo, branch, sha);
    await commitFile(repo, branch, `.claude/tasks/${doc.filename}`, doc.content, `task: ${task.title}`);
    const pr = await openPR(repo, branch, task.title, formatPRBody(task, doc));
    await postComment(repo, pr.number, 'Fugue has set up this task. Implement and merge when ready.');

    // Gate 2: wait for this PR to merge before next task
    await waitForMerge(repo, pr.number);
  }
}
```

`summarizeRepo` is a lightweight function that reads key files (README.md, CLAUDE.md, package.json) from the target repo via GitHub API and concatenates them as context.

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/orchestrator.ts` | Create — `runFugue()` and helpers |

## Out of Scope

- Do not implement the `/reprise` call here — that is Task 11.
- Do not write code to the target repo — the delegate model opens a branch and task doc only. Humans (or Cursor) write the implementation.
- Do not implement the CLI entry point — that is Task 09.

**Dependency:** Tasks 01-07 must all be complete.

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `runFugue('Add a README section about contributing', 'codewithbre/orchestra')` runs without error in a test environment with valid credentials.
3. After step 1, a comment appears on the target issue with the formatted task list. If any tasks exhausted retries, those failures are included in the same comment before Gate 1 prompts for approval.
4. After Gate 1 approval, task docs are committed to branches and draft PRs are opened — one per task.
5. The orchestrator halts at Gate 2 and does not open the next PR until the current PR is merged.
6. Token counts and elapsed ms per step are logged to stdout and stored in `ctx.stepTimings`.
7. `OrchestrationContext` is fully typed — no `any` or implicit `any` anywhere in the file.
8. Write-task and verify-task fan-outs use `fanOut()` — not raw `Promise.all()`. Individual item failures are retried up to `ctx.limits.maxRetries` times before being recorded in `ctx.failures`.
9. `fanOut()` honors the `retry-after` header from 429 responses. It does not use a fixed backoff when a `retry-after` value is present.
10. Prompt construction estimates token count before invoking. If the estimate exceeds the ceiling, it downgrades to compact mode and logs a warning — it does not send and wait for a context-length error.
