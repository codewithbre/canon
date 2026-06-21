# Task 05 — Write-Task Tool

## Outcome

`packages/fugue/runtime/src/tools/write-task.ts` exports `runWriteTask(task: Task, codebaseSummary: string): Promise<TaskDoc>` where `TaskDoc` is `{ filename: string; content: string }`. It calls Claude with the write-task skill as system prompt and returns the full markdown document content ready to be committed to the target repo.

## Why

The write-task tool is invoked in parallel for all tasks after the breakdown. It produces the self-contained task documents that isolated implementation agents will read. The structured `TaskDoc` output (filename + content) is what the orchestrator passes to the GitHub client to commit to the repo.

## Context

### Repository state

Tasks 01-04 are complete:
- `src/skills.ts` exports `loadSkill()`
- `src/llm.ts` exports `createLLM()`
- `src/types.ts` exports `Task` type

### TaskDoc type

Add to `packages/fugue/runtime/src/types.ts`:

```typescript
export const TaskDocSchema = z.object({
  filename: z.string(),  // e.g. "task-01-add-dark-mode.md"
  content: z.string(),   // full markdown document
});

export type TaskDoc = z.infer<typeof TaskDocSchema>;
```

### Filename derivation

The filename is derived from the task title: lowercase, spaces to hyphens, prefixed with a zero-padded index.
Example: task index 0, title "Add dark mode toggle" → `task-01-add-dark-mode-toggle.md`

The LLM should not be asked to generate the filename — derive it deterministically in TypeScript from the task title and index.

### Implementation pattern

```typescript
export async function runWriteTask(
  task: Task,
  index: number,
  codebaseSummary: string,
): Promise<TaskDoc> {
  const llm = createLLM();
  const systemPrompt = loadSkill('write-task');

  const response = await llm.invoke([
    new SystemMessage(systemPrompt),
    new HumanMessage(
      `Write a task document for the following task.\n\nTask:\n${JSON.stringify(task, null, 2)}\n\nCodebase context:\n${codebaseSummary}`,
    ),
  ]);

  const content = response.content as string;
  const filename = `task-${String(index + 1).padStart(2, '0')}-${task.title.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/(^-|-$)/g, '')}.md`;

  return { filename, content };
}
```

Note: `write-task` produces free-form markdown, not structured JSON. Do not use `withStructuredOutput` here — use the raw string response.

## Scope

| File | Action |
|---|---|
| `packages/fugue/runtime/src/types.ts` | Update — add `TaskDoc`, `TaskDocSchema` |
| `packages/fugue/runtime/src/tools/write-task.ts` | Create — `runWriteTask()` |

## Out of Scope

- Do not commit the task doc to GitHub — that is the orchestrator's responsibility.
- Do not validate the document's section structure — that is the verify-task tool's job.

**Dependency:** Tasks 01, 02, 03, and 04 must be complete (`Task` type must exist in `types.ts`).

## Acceptance Criteria

1. `pnpm typecheck --filter fugue-runtime` passes.
2. `runWriteTask(task, 0, summary)` returns a `TaskDoc` with a non-empty `content` string containing the required sections (Outcome, Why, Context, Scope, Out of Scope, Acceptance Criteria).
3. The `filename` field is in the format `task-01-kebab-case-title.md`.
4. The function can be called in parallel for multiple tasks without error.
5. The LangSmith trace shows the write-task skill as the system prompt.
